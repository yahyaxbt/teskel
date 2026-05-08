---
name: add-cron-job
description: Add a scheduled (cron) job. Picks BullMQ Repeatable for simple intervals, Inngest cron for step-based workflows, or Coolify scheduled task for infra-level commands. Covers timezone, jitter, idempotency, observability, kill-switch, and DST handling.
---

# add-cron-job — add a scheduled job

> Plan ref: [Sec. 27 (Workflow runtime)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#27-workflow-runtime--bullmq--inngest),
> [Sec. 26 (Idempotency)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#26-idempotency-retry-and-dlq),
> [Sec. 39 (Observability)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#39-observability-stack).

## Decide the runner

| Runner | When |
| --- | --- |
| **BullMQ Repeatable** | Simple interval / cron, single side-effect job. Default. |
| **Inngest cron function** | Step-based, fan-out, reliability features (`step.run`, `step.sleep`). |
| **Coolify scheduled task** | Infra-level (DB vacuum, Loki retention sweep). |
| **Postgres `pg_cron`** | DB-internal maintenance only. Avoid for app-level work. |

Default: **BullMQ Repeatable**. Move to Inngest when the job has >1
external side-effect or needs durable retries between steps.

## Steps

1. **Pick a job id.** `cron.<domain>.<verb>`, kebab-case. Examples:
   `cron.billing.reconcile-invoices`, `cron.kb.refresh-stale-embeddings`.

2. **Define the schedule.**
   - Cron expression (`0 */1 * * *` = hourly), 5 fields.
   - Timezone: **always UTC**. Translate user-facing schedules at the
     application layer; the engine runs UTC.
   - Add jitter (`±10% of period`) to avoid thundering herds.

3. **Implement the job (BullMQ).**

   ```ts
   // packages/jobs/cron/billing-reconcile-invoices.ts
   import { worker, queue } from '@teskel/queue';
   import { db, schema, sql } from '@teskel/db';
   import { logger } from '@teskel/shared/logger';
   import { trace } from '@opentelemetry/api';
   import { isFlagEnabled } from '@teskel/shared/flags/server';

   const JOB_ID = 'cron.billing.reconcile-invoices';

   export async function register() {
     await queue.add(JOB_ID, { tick: 0 }, {
       repeat: { pattern: '0 */1 * * *', tz: 'UTC', immediately: false },
       jobId: JOB_ID,                  // dedup repeat
       removeOnComplete: { age: 86_400, count: 200 },
       removeOnFail: { age: 7 * 86_400, count: 500 },
     });
   }

   worker(JOB_ID, async () => {
     if (!(await isFlagEnabled('cron.billing.reconcile-invoices', { global: true }))) {
       logger.info({ jobId: JOB_ID }, 'cron_disabled_by_flag');
       return;
     }

     const tracer = trace.getTracer('cron');
     return tracer.startActiveSpan(JOB_ID, async (span) => {
       try {
         // ... business logic ...
         await db.execute(sql`UPDATE invoices SET ... WHERE ...`);
       } finally {
         span.end();
       }
     });
   });
   ```

4. **Idempotent within a window.** A cron firing twice (e.g., during
   deploy churn) must produce the same result. Use:
   - Natural-key UPSERTs.
   - "Already done?" check at the top.
   - DB advisory lock for jobs that must not overlap:
     ```ts
     await db.execute(sql`SELECT pg_advisory_xact_lock(${hashJobId(JOB_ID)})`);
     ```

5. **Single-leader execution.** BullMQ Repeatable already enqueues
   one job per tick across the cluster. For long-running cron jobs,
   add an in-job leader lock to be safe.

6. **Kill-switch.** Every cron must check `isFlagEnabled` first.
   Allows ops to disable a misbehaving cron without rolling code.

7. **Inngest variant** (when you need step.run / step.sleep / fan-out):

   ```ts
   import { inngest } from '@teskel/queue/inngest';

   export const reconcileInvoices = inngest.createFunction(
     { id: 'cron.billing.reconcile-invoices', concurrency: { limit: 1 } },
     { cron: '0 */1 * * *' },
     async ({ step }) => {
       const orgs = await step.run('list-orgs', () => listOrgsWithOpenInvoices());
       await Promise.all(orgs.map((o) =>
         step.invoke(`reconcile-${o.id}`, { function: reconcileOrg, data: { orgId: o.id } })));
     },
   );
   ```

8. **Backfill / catch-up.** Inngest supports `replay` natively;
   BullMQ does not — implement an idempotent "catch-up window" inside
   the job that processes any work missed since the last
   `processed_through_at` watermark.

9. **DST / timezone gotchas.**
   - Engine timezone = UTC.
   - User-facing schedules ("daily at 9am Asia/Jakarta") translated in
     application code, not in the cron expression.
   - For UTC-offsets that don't observe DST (Asia/Jakarta = +07:00
     year-round), translation is constant. For DST-observing zones,
     the application must recompute every run.
   - Avoid scheduling at 02:00–03:00 local time anywhere — DST jumps.

10. **Observability.**
    - Counter `cron_runs_total{job_id, outcome}`.
    - Histogram `cron_run_duration_seconds{job_id}`.
    - Heartbeat metric refreshed at each run.
    - Alert: heartbeat stale > 1.5× expected period → page.
    - Alert: failure rate >5% over 1h → page (lower sev).
    - Grafana dashboard "Cron · Overview" auto-includes new jobs.

11. **Secrets / data access.** A cron has no user identity. If it
    needs to act on tenant data, it must:
    - Iterate orgs explicitly.
    - Use `db.withTenant(orgId)` for each org.
    - Never execute as a superuser bypass of RLS.

12. **Tests.**
    - Unit: business logic with mocked DB.
    - Integration: run handler twice in sequence; verify idempotent
      result; verify advisory lock prevents overlap.
    - Schedule sanity: cron expression resolves to expected times in
      `node-cron` parser.

13. **Documentation.** Add to `docs/runbooks/cron/<job-id>.md`:
    - What it does.
    - Schedule.
    - Last-known-good run frequency.
    - How to disable (flag) and how to manually trigger.
    - Runbook for failure modes.

14. **Manual trigger.** Admin CLI:
    ```bash
    pnpm teskel cron run --job cron.billing.reconcile-invoices
    ```

## Pitfalls

- Scheduling cron at `0 0 * * *` for many jobs — thundering herd.
  Stagger or add jitter.
- Forgetting kill-switch — bad cron loops indefinitely.
- Running heavy work on a single instance with no leader lock — two
  instances both run when one is mid-deploy.
- Hard-coded server-local timezone — breaks when infra moves regions.
- Long-running cron (>30 min) without checkpointing — risk of
  redoing work on retry.
- Cron job that's actually a backfill — use `data-backfill-job` skill
  instead.

## Done when

- Job registered with cron + jitter + dedup `jobId`.
- Idempotent + flag-gated.
- Heartbeat + alerts wired.
- Runbook written.
- Tests cover idempotency + lock.
- Manual trigger CLI works.
