---
name: tenant-offboarding
description: Offboard a tenant — pause, archive, export, and (eventually) erase a workspace's data per retention policy and DSAR rules. Use when a customer cancels, churns, or files a deletion request.
---

# Skill — Tenant offboarding

> **Plan refs:** §47 (backup/DR), §50 (data protection), §53
> (compliance), §62 (billing).
> **Related skills:** `gdpr-data-request`, `db-restore-pitr`,
> `data-backfill-job`, `audit-log-query`, `rotate-secret`.
> **Hard rules:** [`docs/data/retention.md`](../../../docs/data/retention.md);
> [`AGENTS.md` §9](../../AGENTS.md#9-security--data-rules).

Offboarding a tenant is the **inverse** of onboarding. It is one
of the highest-risk operations the platform performs because:

- Mistakes are usually irreversible (data is gone).
- Mistakes can be **silently** wrong (data lingers when it
  shouldn't, or is deleted when it shouldn't).
- The customer is often **emotional** (churn, dispute, layoff) and
  the comms tone matters.

Always run this skill **two-handed** (operator + reviewer). No
single person closes a tenant.

---

## When to invoke

| Trigger | Path |
| --- | --- |
| Customer clicks "Cancel" in billing UI | §3.A — soft cancel |
| Subscription unpaid > 30 d after retries exhausted | §3.B — payment cancel |
| Owner clicks "Delete workspace" | §3.C — owner-initiated delete |
| DSAR erasure request (Article 17 GDPR / equivalent) | §3.D — DSAR erasure (also see [`gdpr-data-request`](../gdpr-data-request/SKILL.md)) |
| ToS / AUP violation, takedown | §3.E — enforced offboarding |
| Tenant migration to self-host (Phase 6) | §3.F — migration |
| Internal sandbox / demo cleanup | §3.G — internal cleanup |

If unsure which path applies, **stop and ask** in
`#sec-engineering`. The wrong path can violate legal floors or
delete data that should have been retained.

---

## Pre-conditions

- You have a **request ticket** with the trigger, the workspace
  ID, the requester (and their authorization to make the request
  if applicable), and any retention overrides.
- You have **dual-control** approval: operator + reviewer.
- You are running from an **operator console**, not from your
  laptop's shell.
- You have read [`docs/data/retention.md`](../../../docs/data/retention.md)
  in the last 30 days.

---

## Steps (high-level — adapt to the specific path)

### 1. Verify identity and authority

For customer-initiated paths (A, C, D, F):

- The requester is an **owner** of the workspace, or
- The requester is an **authorized signer** under the DPA / contract,
  or
- The requester is a **DSAR subject** with verified identity
  (passport / govt ID via support; never via email alone).

For DSAR specifically: the requester must be the **data subject**
(end user of the workspace's app, not the tenant owner) — the
tenant owner cannot delete their end-users via this path; use
the per-tenant DSAR pipeline.

For enforced offboarding (E): authorization is from Trust & Safety
+ Legal (joint sign-off).

### 2. Snapshot the audit log

Before any state change:

```sql
-- Capture relevant audit_log entries to a forensic export
copy (
  select * from audit_log
  where tenant_id = :tenant_id
    and ts >= now() - interval '7 years'
  order by ts asc
) to '/secure-export/<tenant>-audit.jsonl' with (format jsonl);
```

This export goes to encrypted cold storage; **never** purged below
the 7-year floor for Class C. The hash chain remains valid even if
the live audit log is rotated.

### 3. Pause everything

Set tenant `status = 'paused'`. This:

- Blocks new logins for non-owner members.
- Pauses all scheduled workflow runs.
- Drains in-flight runs (lets them complete, no new ones).
- Disables outbound webhooks (events still recorded for replay).
- Disables outbound email send (still queued; replayable on
  re-activation).
- Returns `TENANT_PAUSED` for any API call beyond owner read.

Pause is a **reversible** state. It is the right starting point for
*every* offboarding path.

### 4. Notify

Send the **right** comm for the path:

- A: "We're sad to see you go. Your data is preserved for 30 days;
  you can reactivate by clicking <link>. After 30 days, retention
  rules apply."
- B: "Your subscription wasn't renewed; we've paused your
  workspace. You have 30 days to update billing."
- C: "Your workspace is scheduled for deletion in 30 days unless
  you reactivate."
- D: "We've received your data deletion request. You'll receive a
  completion confirmation within the SLA defined in our Privacy
  Policy."
- E: "Per our Acceptable Use Policy, your workspace has been
  suspended pending review. You'll receive a determination within
  14 days."
- F: "Your migration export is being prepared; ETA <X>."
- G: no comm needed (internal).

Comms go through the [`add-email-template`](../add-email-template/SKILL.md)
pipeline using a versioned template. Never compose ad-hoc.

### 5. Generate exports

For paths A, C, D, F: produce a **portable export** before any
deletion:

- Workflows + prompts + KB docs as JSON / Markdown.
- Page bundles (Puck JSON).
- Members + RBAC mapping (with email hashes only — no plaintext
  emails in the export).
- Billing summary + invoice list.
- Audit log slice (the export from §2, minus internal-only fields).

Encrypt with a per-export key; signed download URL valid 7 days;
audit-logged on download.

For path E (enforced): legal decides whether to provide an export.
Default is **no** until they sign off.

### 6. Wait the grace window (if applicable)

| Path | Grace window | What happens during |
| --- | --- | --- |
| A — soft cancel | **30 d** | Reversible; reactivation restores everything. |
| B — payment | **30 d** | Same as A. |
| C — owner delete | **30 d** | Same as A. |
| D — DSAR erasure | **0 d** | No grace; per-regulation deadline (typically 30 d to *complete*). |
| E — enforced | **0 d**, but data preserved 90 d for legal hold review. | |
| F — migration | per contract | Customer schedules cutover. |
| G — internal | **0 d**. | |

A grace window is **not silent**: the workspace owner gets a
reminder at day 7, day 14, day 25.

### 7. Erase (the destructive step)

When the grace window elapses, the **scheduled erasure job** runs.
It is run by a worker, not a human — humans approve the schedule;
the worker performs.

Order of erasure (per [`docs/data/retention.md`](../../../docs/data/retention.md)):

1. **Class B (usage)** — workflows, runs, KB docs, embeddings,
   pages, blocks. R2 objects deleted with version-history purged.
2. **Class F (security)** — sessions, API keys, login events.
3. **Class A (identity)** — user records, photo blobs, profile
   metadata.
4. **Class E (marketing)** — cohort exposures, A/B logs (PostHog
   cohort delete via API).
5. **Class C (operational)** — logs and traces are not deleted
   per-tenant; they purge naturally per retention.
6. **Class D (financial)** — **NOT deleted**; pseudonymized for
   the affected subject, retained per legal floor.

Each step:

- Runs as a transactional batch with `withTenant`.
- Writes a `retention_runs` row with totals.
- Emits an audit log entry (under a global "retention" tenant so it
  isn't deleted as part of the erasure).
- Cascades through FKs as defined in migrations.

**Failure handling:** if any step fails, the worker stops and pages
on-call. Partial erasure is **never** silently committed.

### 8. Clean up sidekicks

Things that don't live in the main DB and need explicit cleanup:

- Redis cache keys prefixed with the tenant ID — TTL-driven, but
  also explicitly purged.
- BullMQ queues — drain and purge job records.
- Sentry projects / project keys (if per-tenant).
- Better Stack heartbeats / monitors (if per-tenant).
- PostHog cohorts.
- Langfuse traces (per Langfuse retention API).
- Vendor accounts: Stripe customer (delete via API after final
  invoice; some metadata remains for tax).
- Vendor sub-resources: OAuth grants, webhook subscriptions, R2
  buckets if dedicated, custom domains in Cloudflare.
- DNS records pointing at custom domains.
- Marketplace listings authored by this tenant — see §10.

A checklist tool runs each cleanup and asserts "0 remaining" before
declaring offboarding complete.

### 9. Special case: secrets

Any secret the tenant exposed to the platform (OAuth tokens,
webhook signing keys, API keys, BYOK encryption keys) is rotated /
revoked **before** identity-record erasure. See
[`rotate-secret`](../rotate-secret/SKILL.md). This prevents an
attacker from later replaying a credential that points to a deleted
tenant.

### 10. Special case: marketplace creator

If the tenant **published templates**, those templates are not
auto-removed. Buyers may still depend on them. Choices:

| Choice | Behavior |
| --- | --- |
| Transfer ownership | Tenant agrees in writing; templates rewired to new creator. |
| Retire listings | Tenant agrees; existing installs keep working; new buyers blocked; listing badged "retired". |
| Hard takedown | Only on legal / ToS grounds. Triggers refund + buyer migration plan. |

This is decided **before** Step 7. Erasing the tenant without a
plan for their marketplace listings is a Sev1.

### 11. Confirmation + closeout

Send the requester a final comm:

- "Workspace `<id>` (and all its data) has been deleted on
  `<UTC>`. Reference: `<DSAR/erasure ticket id>`."
- For DSAR: include the regulator-required summary of what was
  deleted (categories, not contents).
- For enforced: comm is approved by Legal first.

Close the ticket with:

- Operator + reviewer names.
- Timestamps of pause, comms, exports, erasures, cleanups.
- Final `retention_runs` totals.
- Audit-log signature snapshot from §2 + a fresh signature
  post-erasure.

### 12. Quarterly review

Operator + Reviewer + Privacy lead review the last 90 d of
offboardings:

- How many of each path?
- Any erasures that took > SLA?
- Any cleanup steps that needed manual intervention?
- Any "data leftover" findings (e.g. caches with TTL gone awry)?

Findings produce ADRs / runbooks / new sweeps.

---

## Pitfalls

- **Single-handed offboarding.** Always two-handed. No exceptions.
- **Forgetting Stripe customer deletion** → tax records remain
  intact (good), but ongoing webhooks for stale events get
  swallowed (bad — shows up as `STRIPE_TENANT_NOT_FOUND` in logs).
- **Hard-deleting Class D data** thinking it's "PII." Class D is
  pseudonymized, not deleted. Read [`docs/data/retention.md`](../../../docs/data/retention.md) §5.
- **Letting a marketplace creator's templates orphan.** Buyers
  break silently. Always decide in §10.
- **Skipping the audit-log snapshot.** If you can't prove what was
  deleted, you can't satisfy a regulator.
- **Erasing during grace** because someone "really, really wants
  it." The grace window is policy, not ceremony.
- **PostHog / Langfuse residue.** Vendors have their own retention
  APIs; missing those is the most common "data leftover" finding.
- **Deleting a tenant during a Sev1.** Don't. Pause, fix the
  incident, then schedule erasure.

---

## Done when

- Tenant `status = deleted` (or `paused` if in grace).
- All cleanup-tool checks "0 remaining".
- `retention_runs` shows the row counts deleted.
- Audit-log snapshots stored (pre + post).
- Requester notified with reference ID.
- For DSAR: deletion confirmed within regulator SLA.
- Marketplace listings disposition resolved.
- Quarterly review agenda updated with this offboarding.

---

## References

- [`docs/data/retention.md`](../../../docs/data/retention.md).
- [`docs/security/rbac-matrix.md`](../../../docs/security/rbac-matrix.md).
- [`docs/billing/plans.md`](../../../docs/billing/plans.md) §5.
- [`docs/legal/README.md`](../../../docs/legal/README.md).
- [`gdpr-data-request`](../gdpr-data-request/SKILL.md),
  [`audit-log-query`](../audit-log-query/SKILL.md),
  [`rotate-secret`](../rotate-secret/SKILL.md),
  [`add-email-template`](../add-email-template/SKILL.md).
- Plan §47, §50, §53, §62.
