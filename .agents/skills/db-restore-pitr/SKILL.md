---
name: db-restore-pitr
description: Recover the Postgres database to a specific point in time using WAL-G — covers full restore, partial-table restore via shadow cluster, RPO/RTO calculations, integrity verification, decision tree for in-place vs side-by-side, and tenant-scoped recovery. Use during data-loss incidents, accidental DROP/UPDATE, ransomware, or compliance restore drills.
---

# db-restore-pitr — point-in-time recovery

> Plan ref: [Sec. 41 (Backup & DR)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#41-backup--disaster-recovery),
> [Sec. 39 (Observability)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#39-observability-stack),
> Companion: [`incident-sev1`](../incident-sev1/SKILL.md), [`gameday-drill`](../gameday-drill/SKILL.md).

> **Hard rule:** **never** restore in place over a healthy primary
> while the incident is ambiguous. Default is **side-by-side
> restore** to a shadow cluster, then targeted re-import. In-place
> restore is reserved for full-corruption / total-data-loss
> scenarios with explicit Sev1 commander approval.

## Targets

- **RPO (Recovery Point Objective):** ≤5 min (continuous WAL
  archiving to R2).
- **RTO (Recovery Time Objective):** ≤2 h for full restore on a
  fresh cluster, ≤30 min for a single-table re-import from a
  shadow cluster.

If your incident requires faster than these, you don't have time
for PITR — you need to fail over to the standby (different runbook).

## Decide the restore mode

| Scenario | Mode |
| --- | --- |
| Total primary loss / corruption | **In-place** restore on new primary |
| Region failure | **Failover** to cross-region replica (separate runbook) |
| Accidental `DELETE`/`UPDATE` on one table affecting <1k rows | **Shadow + targeted re-insert** |
| Ransomware / suspected tampering, primary still serves | **Shadow cluster, forensic compare**, then decision |
| One tenant's data corrupted by app bug | **Shadow cluster + tenant re-import** |
| Restore drill (no real incident) | **Shadow cluster, no impact** |

When in doubt: **shadow first**. Costs an hour and a few GB of R2
egress. Safer than rewriting production.

## Pre-conditions

1. **WAL-G is configured.** Verify via:
   ```bash
   wal-g backup-list --json | tail -3
   wal-g wal-show
   ```
   Expect: most recent base backup ≤24h old; WAL stream gaps =0.

2. **Backups verified.** Last successful nightly verification
   (separate cron job runs `wal-g backup-verify`) is green. Check
   Grafana dashboard `Postgres · Backups`.

3. **Vault access.** Restoration requires the
   `WAL_G_S3_PREFIX` + `AWS_*` credentials and the
   `PGENCRYPT_PRIVATE_KEY` for decrypted backups. Stored in 1Password,
   role-gated to SRE on-call.

4. **Incident context.** Sev1 commander or DBA-on-call has approved
   the restore. Restore commands are NOT casual operations.

## Steps — shadow cluster restore

This is the default path; the in-place variation appears at the end.

1. **Determine the recovery target time (RTT).**
   - For accidental change: timestamp **just before** the bad
     transaction. Find via:
     - `audit_events` table.
     - Postgres logs (`log_min_duration_statement` captures DDL).
     - Application telemetry (when did the bug start?).
   - Convert to UTC. Record in incident channel. Example: `RTT =
     2026-05-08 03:14:22 UTC`.

2. **Provision the shadow cluster.**
   - Same Postgres version (pin major+minor; minor diffs can break
     restore).
   - Same extensions (`pgvector`, `pgcrypto`, `pg_partman`, etc.).
   - Sized to comfortably hold the full DB.
   - Network-isolated from production (no client should be able to
     hit it accidentally).
   - Tag: `teskel-shadow-<incident-id>` for cost tracking and
     auto-teardown.

   ```bash
   pnpm teskel db shadow up --incident=INC-2026-05-08-001 --pg-version=16.4
   ```

   This calls Coolify / Terraform under the hood, returns the new
   `DATABASE_URL` (write to a temp file, never echo to logs).

3. **Restore base backup to RTT.**

   ```bash
   PGDATA=/var/lib/postgresql/16/main \
   wal-g backup-fetch ${PGDATA} LATEST_BEFORE_${RTT}
   ```

   Then write `recovery.conf` / `postgresql.auto.conf`:
   ```
   restore_command = 'wal-g wal-fetch %f %p'
   recovery_target_time = '2026-05-08 03:14:22 UTC'
   recovery_target_action = 'pause'
   ```

   Start Postgres in recovery mode. It will replay WAL up to (but
   not past) the RTT and then pause. **Do not promote yet.**

4. **Verify the restored state.**
   - Connect read-only to the shadow.
   - Spot-check: row counts on critical tables vs known-good
     reference (separate metrics export from before the incident).
   - Verify the bad row is **not yet** present (or is still in
     pre-incident state). If it is, your RTT is wrong — pause and
     reconsider.
   - Run integrity sweep:
     ```sql
     -- common smoke
     SELECT max(id), max(created_at), count(*) FROM organizations;
     SELECT max(id), max(created_at), count(*) FROM projects;
     -- domain-specific
     SELECT count(*) FROM workflow_runs WHERE created_at > NOW() - INTERVAL '7 days';
     ```

5. **Promote the shadow to a writable role.**
   ```sql
   SELECT pg_wal_replay_resume();
   SELECT pg_promote();
   ```
   The shadow is now an independent timeline. The production
   timeline is unchanged.

6. **Targeted re-import.** Three sub-flows depending on incident:

   **5a. Single-table mass re-import** (e.g., `templates` was
   wiped):
   ```bash
   # On shadow
   pg_dump -t templates -t template_versions $SHADOW_URL > /tmp/templates.sql

   # Apply to production carefully — wrap in a transaction; first count
   psql $PROD_URL -c "BEGIN; SELECT count(*) FROM templates; -- baseline"
   psql $PROD_URL < /tmp/templates.sql
   ```
   Always: take a snapshot of prod before importing; review diff;
   import inside a transaction so you can `ROLLBACK` on the spot.

   **5b. Per-tenant re-import:**
   ```sql
   -- on shadow
   COPY (SELECT * FROM projects WHERE org_id = 'org_X') TO '/tmp/proj-orgX.csv' CSV HEADER;
   -- on prod
   BEGIN;
   DELETE FROM projects WHERE org_id = 'org_X';   -- if needed
   \copy projects FROM '/tmp/proj-orgX.csv' CSV HEADER
   COMMIT;
   ```
   RLS guidance: re-importing requires bypassing RLS, which means
   using a role with `BYPASSRLS` — this is a privileged operation
   logged via audit.

   **5c. Single-row recovery:** queryable via shadow; manually
   `INSERT … ON CONFLICT DO NOTHING` into prod.

7. **Audit + announce.** Each re-import:
   - Writes an `audit_events` entry (`db.restore.import`) with
     operator, source RTT, table, row count, transaction id.
   - Announced in the incident channel with timestamp.

8. **Tear down the shadow.** Once recovery is verified and the
   incident is closed:
   ```bash
   pnpm teskel db shadow down --incident=INC-2026-05-08-001
   ```
   Shadow data is sensitive — auto-deletion on close is mandatory.
   The deletion itself is audited.

## Steps — in-place restore (full primary loss)

Only if the primary is gone or so corrupted that side-by-side
won't help.

1. Provision a fresh primary cluster with same version + extensions.
2. `wal-g backup-fetch ${PGDATA} LATEST_BEFORE_${RTT}` then
   `recovery_target_time = ${RTT}`.
3. Verify before promote (steps 3–4 above).
4. Promote, then update DNS / connection string to the new primary.
5. Resume application traffic carefully — ramp via canary
   (slow-traffic-up to confirm no replication-lag issue cascades).
6. Reattach a fresh standby to the new primary; once caught up,
   resume normal HA.
7. **Replay missed work.** Application sends between RTT and the
   incident detection time may be lost. Reconcile via:
   - Outbox + audit logs (idempotent).
   - Customer-facing apology + "republish" affordance for any
     content created in the lost window.

## Steps — restore drill (no real incident)

This is the GameDay flow (`gameday-drill` skill); the difference
is:
- No customer impact; no status page update.
- Recovery target is "yesterday at noon".
- Verification rigour is the same.
- After teardown, write a report: did we hit RTO/RPO targets? Were
  there gaps in tooling?

## Cross-cutting requirements

- **Encryption at rest in transit.** WAL-G writes encrypted
  archives to R2. Decryption keys live in 1Password; restore
  process exports them just-in-time, never persists them.
- **Network isolation.** Shadow clusters must be in a private VPC
  segment with no public ingress.
- **Time sync.** All nodes synced to NTP within ±100ms; otherwise
  WAL replay can pick a wrong RTT.
- **Logical replication caveat.** PITR restores physical replicas
  cleanly. If you have logical replication slots, they may need
  re-creation post-restore.

## Observability

- Counter `db_restore_initiated_total{kind=in_place|shadow|drill}`.
- Histogram `db_restore_duration_seconds` (record actual RTO).
- Alert: WAL archive lag > 5 min → page (data sitting outside R2).
- Alert: nightly backup verification failure → page.
- Drill cadence: at least quarterly; results in
  `docs/incidents/drills/`.

## Pitfalls

- Restoring in place during an ambiguous incident — destroys
  evidence and can compound damage.
- Wrong RTT — the bad transaction is also restored. Spend more
  time finding RTT than you think you need.
- Restoring to wrong major version — extensions will refuse.
- Forgetting to terminate replication slots — primary refuses to
  recycle WAL.
- Skipping integrity sweep — re-importing partially corrupted data
  back into prod.
- Leaking the shadow `DATABASE_URL` to logs / Slack — full DB
  exposure.
- Forgetting RLS bypass requires a privileged role + audit entry.
- Public DNS pointing to the new primary before app config is
  updated — apps fail open against the wrong cluster.
- No drill in 6+ months — your tooling is rotted; first time you
  restore in anger, it doesn't work.

## Done when

- Recovery executed via WAL-G with documented RTT.
- Integrity sweep + targeted comparison passed.
- Re-import (if any) wrapped in transactions, audited, announced.
- Shadow cluster torn down.
- Audit log entries present for every privileged action.
- Postmortem includes actual RTO + RPO for the incident.
- For drills: report filed in `docs/incidents/drills/`.
