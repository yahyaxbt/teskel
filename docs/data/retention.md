# Data retention & deletion

> **Status:** Stable. Reviewed quarterly + after every privacy /
> security event.
>
> **Governed by:** Plan §50 (data protection), §53 (compliance
> roadmap). Operationalized by `gdpr-data-request` and
> `data-backfill-job` skills.

This document is the **source of truth** for how long TESKEL keeps
each data class, and how it disposes of data when retention expires
or a deletion is requested. It binds engineering, ops, and support.

If a class of data is not listed here, **it is not retained** — it
must not be persisted at all. Adding a new persistent class is a PR
to this file plus a story in the affected phase.

---

## 1. Principles

1. **Minimize.** Don't store what we don't need.
2. **Tag at write.** Every persistent row has its retention class
   either as a column or via the table-level default.
3. **Delete on schedule.** Retention is enforced by automation, not
   discipline.
4. **Honor user requests.** Deletion requests (DSAR) take precedence
   over retention floors *except* where law forces a floor.
5. **Audit deletes.** Every retention sweep emits a row to
   `retention_runs` with totals.
6. **Encrypt PII at rest.** Hashing is preferred for fields that need
   matching but not display.

---

## 2. Data classes

| Class | Description | Examples |
| --- | --- | --- |
| **A. Identity** | Account-level PII | name, email, phone, profile photo, billing address |
| **B. Usage** | Per-tenant business state | workflows, runs, prompts, KB docs, blocks, pages |
| **C. Operational** | System-internal records that aren't customer content | logs, traces, metrics, audit log, RLS sanity events |
| **D. Financial** | Billing-related | invoices, payments, payouts, tax records |
| **E. Marketing** | Funnel + analytics | PostHog events, attribution, A/B exposures |
| **F. Security** | Auth + sec events | login events, 2FA challenges, session tokens, API key usage |
| **G. Sensitive** | Regulated / extra-care | health (Phase 6 / HIPAA), payment cards (we don't store; tokenize via Stripe) |
| **H. Backups** | DR copies | Postgres WAL + base backups, R2 object versions |
| **I. Models** | Vendor-side data | LLM provider logs (varies by provider) |

---

## 3. Retention matrix

| Class | Item | Retention | Disposal | Notes |
| --- | --- | --- | --- | --- |
| A | User account record (`users`) | While account active **+ 30 d after deletion** | Hard delete (cascades). | 30 d window allows account recovery. |
| A | Org membership (`org_members`) | While member **+ 7 d** | Hard delete. | Audit log retains the action. |
| A | Profile photos in R2 | Same as user account. | Bucket lifecycle rule. | |
| A | KYC docs (creators, Phase 3+) | **5 y** after relationship ends | Hard delete. | Required by Stripe Connect / local KYC law. |
| B | Workflows (`workflows`) | While org active **+ 30 d** | Hard delete (all runs, steps, KB associations). | Trash bin gives 30 d recovery. |
| B | Workflow runs (`workflow_runs`) | **90 d** rolling | Soft delete (status flip, then nightly hard delete). | User can opt to extend to 1 y on Pro+. |
| B | Workflow step inputs / outputs | **90 d** rolling, **truncated** at 30 d (keep metadata, drop payload) | Truncate then delete. | Reduces storage spike. |
| B | KB documents (`kb_documents`) | While org active **+ 30 d** | Hard delete + R2 object. | Embeddings cascade-delete. |
| B | Embeddings (`kb_chunks`) | Tied to source doc. | Cascade delete. | |
| B | Templates (`marketplace_listings`, drafts) | While creator active **+ 90 d** | Soft delete; metadata retained for legal trail. | Sold templates: see Class D. |
| B | Customer's app sessions / form submissions | Per buyer's tenant policy. | Per their `data/retention.md`. | Out of scope for this matrix. |
| C | App logs (Loki) | **30 d** | Loki retention policy. | No PII allowed in logs. |
| C | Traces (OpenTelemetry / Sentry) | **30 d**; perf samples 7 d | Auto-purge. | |
| C | Metrics (Grafana) | **400 d** at full resolution; downsample after | Native retention. | For YoY comparison. |
| C | Audit log (`audit_log`) | **3 y** by default; **7 y** on Business+ | Hard delete only by retention job. | Hash-chained; cannot be edited. |
| C | RLS sanity events | **90 d** | Auto-purge. | Should be 0 in prod. |
| D | Invoices, payments | **7 y** (regulatory) | Never deleted while in retention window; archived to R2 cold after 1 y. | Indonesian + EU + US tax minima. |
| D | Stripe payouts mirror | **7 y** | Same as invoices. | |
| D | Refunds / chargebacks | **7 y** | Same. | |
| E | PostHog events | **24 mo** | PostHog retention. | Funnel analyses still work via cohort snapshots. |
| E | Attribution data | **24 mo** | Same. | |
| E | A/B exposure / flag eval logs | **6 mo** | Auto-purge. | |
| F | Login events | **180 d** | Auto-purge. | Available for security investigation. |
| F | 2FA challenge logs | **180 d** | Auto-purge. | |
| F | Session tokens | TTL **30 d** sliding; **90 d** absolute | Auto-purge on expiry. | Refresh extends. |
| F | API key usage (last-used) | While key active. | Deleted with key. | |
| G | Health / clinical data (Phase 6 HIPAA) | Per BAA — **6 y** minimum. | Per BAA. | Region-pinned; CMK. |
| G | Payment card data | **Never stored.** | n/a — Stripe tokenization. | |
| H | Postgres WAL | **35 d** rolling | WAL-G expiry. | Allows PITR within window. |
| H | Postgres base backups | Daily for 35 d, weekly for 90 d, monthly for 365 d. | WAL-G. | |
| H | R2 object versions | **35 d** version history. | Bucket lifecycle. | |
| I | LLM provider logs | Per provider terms. We choose **no-train** providers by default. | Per provider. | Documented in `docs/legal/`. |

Anything not in this table is **ephemeral** — must not survive
process restart or transaction commit.

---

## 4. Per-region & per-plan overrides

| Region / plan | Difference |
| --- | --- |
| **EU** tenants (Phase 4+) | All Class A,B,F,G data residency in EU region. Deletion windows shortened to **30 d** floor for Class C audit log if local DPA requires it (Belgian DPA precedent). |
| **US-HC** tenants (Phase 6 HIPAA) | Class G expanded to 6 y minimum; medical attestations stored. |
| **Free** plan | Class B retention shortened to **30 d** rolling for runs (saves cost). |
| **Pro+** plan | Class B retention extended to **1 y** for runs (per opt-in). |
| **Enterprise** plan | Negotiated. Stored in DPA; engineering must surface in product UI. |

---

## 5. DSAR (data subject access / erasure)

When a user files a DSAR (per `gdpr-data-request` skill):

| Request type | Retention rule | Override |
| --- | --- | --- |
| Access (export) | n/a — read-only. | Honor regardless of retention. |
| Portability | n/a — read-only. | Honor. |
| Rectification | Updates row in place; audit-logged. | Honor. |
| **Erasure** | Overrides Class A,B,E,F floors. | **Cannot** override Class D (financial), Class C audit log of the erasure itself, or legal-hold cases. |
| Restriction | Marks row, blocks processing; auto-purge later as if erased. | Honor. |
| Objection | Removes from marketing/training; data persists per usual retention. | Honor. |

For Class D (financial) data linked to an erased subject, we
**pseudonymize**: replace name + email with placeholder, retain
amount + tax record. This satisfies regulatory floors without
holding identifiable data.

---

## 6. Legal hold

When law enforcement / counsel issues a legal hold:

1. Suspend retention sweeps for the affected tenant + classes.
2. Write a `legal_holds` row with scope + originator + ticket.
3. Resume sweeps when the hold lifts.

Legal holds override **all** retention rules (above and below the
default). Implemented via a `holds_until` column on the affected
tables; retention sweep query excludes rows under hold.

---

## 7. Disposal procedure

| Mode | When | Mechanism |
| --- | --- | --- |
| **Soft delete** | Allow grace period or trash bin. | Set `deleted_at`. RLS / scopes filter out. |
| **Hard delete** | Retention expired or DSAR erasure. | `DELETE` SQL; cascade where defined; storage cleanup separate job. |
| **Truncate** | Class B step payload after 30 d. | Set payload to `null`; keep metadata. |
| **Pseudonymize** | Class D when linked to erased subject. | Replace identifying columns with placeholder; retain rest. |
| **Encrypt-and-forget** (planned) | Long-term archive Class D. | Encrypt with key escrow; delete key when retention ends. Phase 6. |

Storage cleanup (R2 objects, embeddings, queue payloads) runs
**after** the DB row is gone. The `audit-log-query` skill helps
operators trace what was deleted when.

---

## 8. Sweeps

| Sweep | Cadence | Owner | Implementation |
| --- | --- | --- | --- |
| Class B run / step retention | nightly 02:00 UTC | Backend | BullMQ Repeatable; per-tenant quota. |
| Class C log + trace retention | continuous | Platform | Loki / OTel collector retention. |
| Class A inactive accounts | monthly | Identity | One-shot job; emits report. |
| Class D archive to cold | quarterly | Backend | R2 lifecycle + DB compaction. |
| Class F session pruning | hourly | Identity | BullMQ Repeatable. |
| Class H WAL expiry | continuous | Platform | WAL-G. |
| Backup verification | weekly | Platform | `gameday-drill` skill, scenario "PITR". |

Each sweep emits to `retention_runs` and to Loki:

```jsonc
{
  "sweep": "workflow_runs_retention",
  "tenant_id": "all",        // or specific tenant for DSAR-driven sweeps
  "started_at": "...",
  "ended_at": "...",
  "rows_scanned": 1234567,
  "rows_deleted": 1234,
  "errors": []
}
```

Failures alert; `incident-sev2` if rows-deleted is suspiciously high
or low vs prior runs.

---

## 9. Backups & PITR (Class H)

- **WAL** retained 35 d rolling.
- **Base backups** daily for 35 d, weekly 90 d, monthly 365 d.
- **R2** object versioning 35 d.
- **Encrypted** at rest; per-region keys; cross-region copy for DR.
- **Tested**: `gameday-drill` PITR scenario quarterly; `db-restore-pitr`
  skill is the playbook.

A subject erasure does **not** retroactively rewrite backups. Any
restore that crosses an erasure boundary triggers a re-erasure pass
on the restored cluster, audit-logged.

---

## 10. Data-class header on every table

When you `add-table`, the migration includes a comment header:

```sql
comment on table workflow_runs is
  'class:B; retention:90d-rolling; pii:no; dsar:cascade;';
```

Lint job parses the comments and builds a current matrix. A table
without this comment fails CI.

---

## 11. Reporting

- A **per-tenant retention report** is downloadable from the
  workspace settings (Phase 4): "what we hold and for how long",
  per data class.
- A **per-incident retention report** (e.g. after a DSAR) goes to
  the requester within the SLA.
- An **annual retention review** is filed as an ADR each calendar
  year, capturing changes for audit (SOC2 / ISO).

---

## 12. References

- [`docs/security/secrets.md`](../security/secrets.md).
- [`docs/security/rbac-matrix.md`](../security/rbac-matrix.md).
- [`docs/legal/README.md`](../legal/README.md) — privacy, ToS, DPA.
- `gdpr-data-request`, `data-backfill-job`, `db-restore-pitr`,
  `gameday-drill` skills.
- Plan §50, §53.
