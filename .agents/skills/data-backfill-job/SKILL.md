---
name: data-backfill-job
description: Write a one-off data backfill or migration job that runs over a tenant-isolated table at scale. Covers planning, dry-run, batching, idempotency, RLS, throttling, observability, and shutdown semantics. Use when an app-level migration cannot be expressed as a single Drizzle migration.
---

# data-backfill-job — write a safe one-off backfill

> Plan ref: [Sec. 19 (DB)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#19-database--postgres-drizzle),
> [Sec. 22 (RLS)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#22-multi-tenancy--rls),
> [Sec. 26 (Idempotency)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#26-idempotency-retry-and-dlq),
> [Sec. 42 (Backup/DR)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#42-backup--disaster-recovery).

> **Hard rule:** never run an ad-hoc `UPDATE ... WHERE ...` against
> prod from a console. Always go through a versioned, reviewed job.

## When to use a backfill (vs. a Drizzle migration)

| Use a Drizzle migration | Use a backfill job |
| --- | --- |
| Schema change | Per-row recompute / enrichment |
| Default value applied via `SET DEFAULT` | Default needs business logic |
| Small constant data fix (`UPDATE ... WHERE id IN (...)`) | Spans millions of rows |
| Single transaction safe | Long-running, must batch |
| Reversible via down migration | Best-effort, idempotent re-runs |

If you can do it in `<10 seconds` with a Drizzle SQL migration, do
that. Otherwise, this skill.

## File layout

```
packages/jobs/backfills/
  20260512-recompute-project-stats/
    job.ts            # the worker
    job.test.ts
    README.md         # plan, ETA, rollback
    state.sql         # optional: state-tracking table DDL
```

Naming: `<YYYYMMDD>-<short-slug>`. Once shipped, never rename.

## Steps

1. **Write the plan.** `README.md` first. Don't open code.

   ```markdown
   # Backfill: recompute project stats

   ## Why
   Projects had `lastActiveAt` and `nodeCount` denormalized; logic was
   wrong before #PR-432. Recompute from source-of-truth tables.

   ## Scope
   - Table: `projects`
   - Rows affected: ~3.2M
   - ETA at 500 rows/s: ~110 minutes
   - Tenants affected: all
   - Read load: light (indexed)
   - Write load: 500 UPDATEs/s peak; PgBouncer pool size +2

   ## Idempotency
   Each row is keyed by (orgId, projectId). Re-running computes the
   same value if source rows haven't changed.

   ## State tracking
   `backfill_state` table keeps cursor + checkpoint per shard.

   ## Failure mode
   Stop on first error. Operator can resume from checkpoint.

   ## Verification
   - Sample 1,000 rows pre and post; assert nodeCount diff bounded.
   - Sentinel orgs unchanged (`org_demo`).
   - p95 read latency for projects unchanged ±10%.

   ## Rollback
   - `lastActiveAt`/`nodeCount` are denormalized; rollback is a re-run
     of the previous (correct-as-of-old-logic) backfill, kept at
     `20260101-recompute-project-stats`.
   - For corruption, restore the affected rows from PITR (last full
     backup + WAL replay to T-1m before job start).
   ```

2. **Create state-tracking table** (if not present):

   ```sql
   CREATE TABLE IF NOT EXISTS backfill_state (
     job_id text PRIMARY KEY,
     last_cursor text,
     processed bigint NOT NULL DEFAULT 0,
     errors bigint NOT NULL DEFAULT 0,
     started_at timestamptz NOT NULL DEFAULT now(),
     updated_at timestamptz NOT NULL DEFAULT now(),
     finished_at timestamptz
   );
   ```

3. **Implement the job.** Use a worker (BullMQ) so we get retries +
   observability for free; one-off scripts are forbidden.

   ```ts
   // packages/jobs/backfills/20260512-recompute-project-stats/job.ts
   import { db, schema, sql } from '@teskel/db';
   import { logger } from '@teskel/shared/logger';
   import { trace } from '@opentelemetry/api';

   const JOB_ID = '20260512-recompute-project-stats';
   const BATCH_SIZE = 500;
   const SLEEP_MS = 50;
   const SHUTDOWN = { stop: false };

   process.on('SIGTERM', () => { SHUTDOWN.stop = true; });
   process.on('SIGINT', () => { SHUTDOWN.stop = true; });

   export async function run({ dryRun }: { dryRun: boolean }) {
     const tracer = trace.getTracer('backfill');
     await tracer.startActiveSpan(JOB_ID, async (span) => {
       try {
         await db.execute(sql`
           INSERT INTO backfill_state (job_id) VALUES (${JOB_ID})
           ON CONFLICT (job_id) DO NOTHING`);

         while (!SHUTDOWN.stop) {
           const cursor = await getCursor();
           const batch = await db.execute(sql`
             SELECT id, org_id FROM projects
             WHERE id > ${cursor ?? '00000000-0000-0000-0000-000000000000'}
             ORDER BY id
             LIMIT ${BATCH_SIZE}`);
           if (batch.length === 0) break;

           for (const row of batch) {
             // Use withTenant — RLS still enforced even from the worker.
             const tx = db.withTenant(row.org_id);
             const stats = await tx.execute(sql`
               SELECT
                 (SELECT MAX(updated_at) FROM workflow_nodes
                   WHERE project_id = ${row.id}) AS last_active,
                 (SELECT COUNT(*) FROM workflow_nodes
                   WHERE project_id = ${row.id}) AS node_count
             `);

             if (!dryRun) {
               await tx.execute(sql`
                 UPDATE projects
                 SET last_active_at = ${stats[0].last_active},
                     node_count = ${stats[0].node_count},
                     updated_at = now()
                 WHERE id = ${row.id}`);
             }
             await advanceCursor(row.id);
             span.addEvent('row_processed', { 'project.id': row.id });
           }

           await sleep(SLEEP_MS);
           await heartbeat();
         }

         await markFinished();
         logger.info({ jobId: JOB_ID }, 'backfill_finished');
       } finally {
         span.end();
       }
     });
   }

   async function getCursor(): Promise<string | null> { /* read backfill_state */ }
   async function advanceCursor(id: string): Promise<void> { /* update backfill_state */ }
   async function heartbeat(): Promise<void> { /* update updated_at */ }
   async function markFinished(): Promise<void> { /* set finished_at */ }
   const sleep = (ms: number) => new Promise((r) => setTimeout(r, ms));
   ```

4. **Always go through `withTenant(orgId)`.** Backfills run with a
   service principal; without `withTenant`, RLS still enforces, but
   the policies are written assuming tenant context is set. Don't
   bypass.

5. **Idempotency.** A backfill MUST be safe to re-run from any point.
   - Compute deterministically from source rows.
   - Skip rows already at the target state (cheap check before
     UPDATE).
   - If you can't be idempotent, you are not writing a backfill —
     you're writing a one-shot, and that's a separate, riskier
     pattern that needs an explicit ADR.

6. **Throttling.** Default 500 rows/s. Watch:
   - Postgres CPU.
   - Replica lag (`pg_stat_replication`).
   - PgBouncer pool waits.
   - p95 user-facing query latency.

   If any breaches threshold, the job auto-throttles (doubles
   `SLEEP_MS`). Threshold + breaker logic lives in
   `packages/jobs/lib/throttle.ts`.

7. **Dry-run.** First run with `dryRun: true`, write a sample to
   `backfill_dryrun` table:

   ```sql
   CREATE TABLE backfill_dryrun (job_id text, project_id text, before_json jsonb, after_json jsonb);
   ```

   Compare 1,000 sample rows manually (or with a SQL diff query)
   before flipping `dryRun = false`.

8. **Trigger.** Don't run from a developer's laptop. Trigger via
   Coolify cronjob or a one-off worker pod:

   ```bash
   pnpm --filter @teskel/jobs run start -- --job 20260512-recompute-project-stats --dry-run=true
   ```

9. **Observability.**
   - OTel span per job + per row.
   - Counter metric `backfill_processed_total{job_id, outcome}`.
   - Heartbeat every 30s; alert if stale >5 min while job is supposed
     to be active.
   - Log structured progress every 1,000 rows.

10. **Shutdown semantics.**
    - SIGTERM → finish current row → mark cursor → exit.
    - SIGKILL → on next start, resume from cursor.
    - Never lose more than 1 row of work on a graceful shutdown.

11. **Tests.**
    - Unit: pure compute function (no DB) covers expected inputs.
    - Integration (Testcontainers Postgres): seed 100 rows, run job,
      assert all rows updated, cursor advanced, idempotent re-run is
      a no-op.

12. **Cleanup.** After the backfill is fully complete and verified
    in prod:
    - Mark `finished_at` in `backfill_state`.
    - Keep the job code in repo (audit trail) — never delete.
    - Open a follow-up to remove the now-obsolete state table only
      after 90 days.

## Pitfalls

- Updating millions of rows in a single transaction — locks +
  bloat, dangerous on prod.
- Forgetting `ORDER BY id LIMIT N` — Postgres returns rows in
  unspecified order, cursor can repeat.
- Bypassing `withTenant` — easy to write a "service-level" query
  that ignores RLS and breaks tenant isolation.
- No throttle / no breaker — saturates DB and pages on-call.
- Running on a single thread when shard-able — OK for small
  backfills; for >50M rows, shard by `org_id % N` and run N workers.
- One-shot scripts (`scripts/fix-prod.ts`) — forbidden.
- Logging row contents — likely contains PII.

## Done when

- README.md complete (why, scope, ETA, rollback).
- Job committed under `packages/jobs/backfills/<date>-<slug>/`.
- Dry-run sample reviewed and signed off.
- Real run started after low-traffic window confirmed.
- Heartbeat + dashboards green for the duration.
- Verification queries match expected diff bounds.
- `backfill_state.finished_at` set.
- Postmortem note added to README if anything went sideways.
