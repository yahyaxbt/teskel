---
name: gdpr-data-request
description: Process a GDPR/UK-GDPR/CCPA/PDP-Indonesia data subject request (DSAR) — access (export), erasure (delete), rectification, or portability. Covers identity verification, scope discovery, secure delivery, deletion across all stores, audit logging, and SLA tracking.
---

# gdpr-data-request — handle a data subject request

> Plan ref: [Sec. 49 (PII / privacy)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#49-pii-and-privacy),
> [Sec. 51 (Audit log)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#51-audit-log),
> [Sec. 53 (Compliance roadmap)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#53-compliance-roadmap),
> [Sec. 50 (Data retention)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#50-data-retention).

> **Hard rule:** every step is **audit-logged with the operator's
> identity**. Never delete or export anything outside the documented
> automation; manual SQL bypasses the audit trail and breaks
> compliance posture.

## Request types

| Type | Right | Default SLA |
| --- | --- | --- |
| **Access (DSAR)** | "Tell me what you have on me" | 30 days (GDPR) / 45 days (CCPA) |
| **Portability** | "Export my data in machine-readable form" | 30 days |
| **Rectification** | "Fix incorrect data" | 30 days |
| **Erasure (RTBF)** | "Delete me" | 30 days |
| **Restriction** | "Stop processing while we dispute" | 30 days |
| **Objection** | "Stop using my data for X" | 30 days |

UK-GDPR mirrors EU. CCPA: 45 days, extendable. PDP Indonesia: 14
business days. Always go with the **strictest** applicable
jurisdiction (default: 14 business days for action, formal answer
within 30 days).

## Pre-flight

1. **Receive the request.** Channels:
   - In-app `Settings → Privacy → Submit request` (preferred; auto-
     verifies identity through current login).
   - Email to `privacy@teskel.dev` (manual identity verification
     required).
   - Regulator forwarded request.

   All channels create a row in `data_subject_requests` with status
   `received` and starts the SLA clock.

2. **Verify identity.**
   - In-app + recent login + 2FA: identity verified.
   - Email channel: send signed challenge link to the registered
     email; if email is the disputed identifier, require government
     ID upload to dedicated reviewer (not stored long-term).
   - For requests on behalf of a minor or via legal representative,
     escalate to legal counsel — different procedure.

3. **Disambiguate subject.** A request can be:
   - **Personal** ("export my data" — individual user).
   - **Shared with org** ("delete me" — but org may legally need the
     data; check the legal basis).
   - **Misaddressed** (request actually applies to a different
     controller — TESKEL-as-processor case for enterprise tenants).

   If TESKEL is **processor** (Enterprise plan, customer is
   controller), forward to the customer's privacy contact and
   document; we do not act unilaterally on processor data.

## Steps — Erasure (RTBF) flow

This is the most operationally complex. Other types (access /
portability) follow a similar shape but skip the deletion step.

1. **Open ticket.** `data_subject_requests` row created with type
   `erasure`. Status: `received` → `verifying` → `scoping` →
   `executing` → `verified` → `closed`.

2. **Scope discovery.** Run the discovery query (admin tooling, not
   ad-hoc SQL):

   ```bash
   pnpm teskel privacy scope --user-id=<id>
   ```

   Output: every store, table, and external system holding data
   linked to the subject. Includes:
   - Postgres tables (queryable via `subject_id` map).
   - Object storage (R2): user-uploaded files.
   - KB embeddings (pgvector): chunks containing user content.
   - Logs (Loki retention is 30/90/365d depending on stream).
   - Metrics (cardinality may include user_id; usually no PII).
   - Backups (handled separately — see step 7).
   - Third-party processors (Stripe, Resend, PostHog, Sentry,
     Langfuse, OpenRouter — each with its own deletion flow).

3. **Approve scope.** Two-person rule: requester + privacy reviewer
   sign off. Captured in audit log.

4. **Execute deletion.** Run the deletion job (per-store, idempotent,
   resumable):

   ```bash
   pnpm teskel privacy erase --user-id=<id> --request-id=<dsr_id>
   ```

   For each store the job:
   - Locks the subject record (status = `pending_deletion`).
   - Deletes or anonymizes per the data-handling spec:
     - **Hard delete** for PII fields (name, email, phone, address,
       photo).
     - **Anonymize** for fields needed for aggregate stats (replace
       `user_id` with `anonymized_user_<hash>`).
     - **Retain** for legal holds (audit logs of fraudulent activity,
       financial records under tax/AML rules) — document the
       retention basis in the DSR record.
   - Removes from R2 prefixes `users/<id>/...`.
   - Removes embeddings rows referencing the subject.
   - Triggers third-party deletion via each integration's privacy
     API (Stripe `Customer.delete`, PostHog distinct ID delete,
     Sentry user erasure, Langfuse delete, OpenRouter — usually no
     subject data, but check).

5. **Verify.** After deletion job completes:
   - Re-run scope discovery → expect zero rows in deletable stores.
   - Document any retained data + its legal basis.
   - Status: `verified`.

6. **Notify subject.** Email confirmation:
   - What was deleted (categories, not specific records).
   - What was retained + reason + retention period.
   - Date.
   - Channel for follow-up.

   Use `add-email-template` skill (template: `dsar-erasure-complete`).

7. **Backups.** TESKEL keeps WAL-G backups for 30 days. Per GDPR
   guidance:
   - Subject is removed from production stores immediately.
   - Subject **persists in backups** until backups age out.
   - If a backup is restored within retention window, the deletion
     job is re-run automatically against the restored state.
   - This policy is documented in our public privacy notice.

8. **Audit log.** Every step writes to `audit_events` with type
   `dsr.erasure.*` (received, verified, scoped, executing, deleted,
   confirmed). Hash-chained per `add-rbac-role` audit conventions.

9. **Close.** Status: `closed`. Total elapsed time stored on the
   `data_subject_requests` row for SLA reporting.

## Steps — Access / Portability flow

1. **Verify identity** (same as above).
2. **Generate export bundle:**
   ```bash
   pnpm teskel privacy export --user-id=<id> --request-id=<dsr_id> --format=jsonl
   ```
   Bundle structure:
   ```
   teskel-export-<dsr_id>.zip
     manifest.json          # what's included, schema versions
     profile.jsonl
     organizations.jsonl
     projects.jsonl
     templates_owned.jsonl
     workflow_runs.jsonl    # only metadata, not raw inputs/outputs unless explicitly opted-in
     uploads/<file>...
     README.md              # plain-English description
   ```
   Sensitive fields (e.g., hashed passwords) omitted; documented in
   manifest as "not included for security".
3. **Deliver securely.** Pre-signed URL valid for 7 days, requires
   subject's auth + 2FA to download. Email contains the link, not
   the file.
4. **Audit log + close** (mirror steps 8–9 above).

## Steps — Rectification flow

1. Verify identity.
2. Identify the disputed field(s).
3. Validate the correction (if it's a controlled field like email,
   trigger email verification on the new value).
4. Update + audit log.
5. Confirm to subject.

## Cross-cutting requirements

- **No PII in code.** All subject lookups go through the privacy
  helpers (`pnpm teskel privacy …` CLI), not direct DB queries.
- **No copying to local.** Export bundles are generated on a
  privacy-isolated worker, written to a vault bucket, and never
  stored on operator workstations.
- **Two-person rule.** Erasure execution requires a second approver
  in admin UI. Cannot be self-approved.
- **SLA tracking.** Dashboard `Privacy · DSARs` shows pending
  requests by age. Alert: any request >25 days old → page privacy
  lead.
- **Regulatory reporting.** Aggregate volume + average response
  time exposed in `docs/compliance/dsar-stats.md`, refreshed weekly.

## Pitfalls

- Manual SQL `DELETE FROM users WHERE id = …` — bypasses scope,
  bypasses audit, may break referential integrity, may miss third-
  party stores.
- Treating processor requests as controller requests — TESKEL
  must forward DSARs received about Enterprise tenant data to that
  tenant; we do not delete on their behalf.
- Forgetting embeddings (pgvector) — they encode user content even
  after the source is deleted.
- Forgetting third parties — Stripe still has the customer, PostHog
  still has the events.
- Sending the export bundle by email attachment — file may exceed
  attachment limits and is unsafe at rest in inbox.
- Confirming erasure before the job verified zero rows — partial
  deletes are common; always verify.
- Not retaining audit logs of the erasure itself — we need proof we
  deleted; the erasure record is itself retained (anonymized) for
  ≥6 years per regulator expectations.

## Done when

- Status `closed` with full timeline in `data_subject_requests`.
- Audit log shows every step with operator identity.
- Subject notified.
- Verification step shows zero rows in deletable stores (erasure)
  or bundle delivered (access).
- Total time within applicable jurisdiction's SLA.
- DSAR stats updated.
