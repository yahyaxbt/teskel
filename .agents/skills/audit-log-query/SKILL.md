---
name: audit-log-query
description: Query the hash-chained audit log to answer who-did-what-when, validate the chain, and produce forensic reports. Use during incidents, DSARs, security investigations, and compliance audits.
---

# Skill — Query the audit log

> **Plan refs:** §50 (data protection), §53 (compliance roadmap),
> §94 (Phase 6 audit log).
> **Related skills:** `tenant-offboarding`, `gdpr-data-request`,
> `incident-sev1`, `incident-sev2`, `triage-bug`,
> `write-postmortem`.
> **Hard rules:** [`AGENTS.md` §9](../../AGENTS.md#9-security--data-rules);
> [`docs/data/retention.md`](../../../docs/data/retention.md) Class C.

The audit log is the **append-only, hash-chained ledger** of every
sensitive action in the platform: who did what, to which resource,
when, from where, and with what outcome. It is the artifact of
record for incidents, DSARs, regulator inquiries, and internal
investigations.

This skill is the operator's playbook for **reading** the log
correctly: building queries that scope to a tenant, verifying the
hash chain, and producing reports that auditors will accept.

You will **not** mutate the audit log via this skill. The audit log
is write-once.

---

## When to invoke

- Investigating a Sev1 / Sev2 — "what did the system actually do?"
- Producing a DSAR access export (per
  [`gdpr-data-request`](../gdpr-data-request/SKILL.md) Step 5).
- Verifying tenant offboarding (per
  [`tenant-offboarding`](../tenant-offboarding/SKILL.md) Step 11).
- Responding to a regulator / customer / internal compliance
  request.
- Annual SOC2 / ISO 27001 evidence collection.
- Suspected security breach — "did anyone read this?"
- Customer billing dispute — "what happened on this date?"

You do **not** invoke this for casual debugging. The audit log is
**not** application logs (Loki). Application logs answer "what was
the request"; audit logs answer "what was the *intent*".

---

## What's in the log

Each row schema (illustrative — exact columns evolve via ADR):

```sql
audit_log (
  id            uuid primary key,
  ts            timestamptz not null,           -- UTC; never mutated
  tenant_id     uuid not null,                  -- the affected tenant
  actor_kind    text not null,                  -- user|service|system|integration|automation
  actor_id      uuid,                           -- user id, service principal, etc.
  actor_meta    jsonb,                          -- IP, user_agent, session_id, api_key_id, …
  action        text not null,                  -- 'workflow.run.start', 'rbac.role.granted', …
  resource_kind text not null,                  -- 'workflow', 'user', 'kb_doc', …
  resource_id   uuid,
  outcome       text not null,                  -- success|failure|denied
  attributes    jsonb,                          -- before/after diff, scope, reason, error_code
  request_id    text,                           -- correlates to Loki / Sentry
  span_id       text,
  hash_prev     bytea not null,                 -- previous row's hash (chain link)
  hash          bytea not null,                 -- sha256 of (this row, minus hash, plus hash_prev)
  signature     bytea                           -- per-period HSM signature (Phase 4+)
)
```

**Invariants:**

- Append-only. No `UPDATE`, no `DELETE` from any role except the
  retention sweeper, which deletes whole partitions older than the
  Class C floor.
- `hash_prev` of row N = `hash` of row N−1. The chain starts with
  the genesis row's `hash_prev = 0x00`.
- Within a tenant's slice, sub-chain is also independently
  verifiable (per Phase 4+ implementation).
- The log is replicated read-only to a separate cluster for
  forensics; queries run against the replica.

---

## Steps

### 1. Confirm authority + scope

Before opening the console, confirm:

- The **investigator** (you) has `audit.read` permission for the
  scope (per [`docs/security/rbac-matrix.md`](../../../docs/security/rbac-matrix.md)).
- The **request** has a written ticket: incident, DSAR, regulator,
  or compliance audit. *No casual queries.*
- The **scope** is bounded: tenant, time window, actor, action,
  resource. Open-ended queries are a red flag.
- If the scope is **cross-tenant**, you need an additional
  approver (per cross-tenant operations allowlist in
  [`docs/architecture/multi-tenancy.md`](../../../docs/architecture/multi-tenancy.md) §8).

Every query you run is itself audit-logged. Don't probe.

### 2. Pick the right interface

| Interface | When |
| --- | --- |
| **Audit Console UI** (in-app, RBAC-gated) | Quick queries, customer-facing self-serve, Pro+ tenants with `audit.read`. |
| **Read-only SQL replica** | Forensic deep-dives, ad-hoc joins, large exports. |
| **Audit CLI** (`teskel audit query …`) | Operator scripting; emits structured output for downstream tools. |
| **Vendor BI tools** | **Forbidden.** The audit log does not leave the trust boundary. |

Most queries should be UI- or CLI-driven. Drop to SQL only when
the question genuinely requires it.

### 3. Build a scoped query

Always start with `tenant_id` and `ts` bounds. Examples:

**Who changed an RBAC role on tenant X in the last 7 days?**

```sql
select ts, actor_kind, actor_id, actor_meta, action, resource_id, attributes, outcome
from audit_log
where tenant_id = :tenant_id
  and ts >= now() - interval '7 days'
  and action like 'rbac.%'
order by ts asc;
```

**What did user U do across their workspace?**

```sql
select ts, action, resource_kind, resource_id, outcome
from audit_log
where tenant_id = :tenant_id
  and actor_kind = 'user'
  and actor_id = :user_id
  and ts between :start and :end
order by ts asc;
```

**Did anyone read KB doc D?**

```sql
select ts, actor_kind, actor_id, actor_meta->>'ip' as ip, outcome
from audit_log
where tenant_id = :tenant_id
  and resource_kind = 'kb_doc'
  and resource_id = :doc_id
  and action in ('kb.doc.read', 'kb.doc.export')
order by ts asc;
```

**Was a workflow run triggered from outside the workspace?**

```sql
select ts, actor_kind, actor_id, actor_meta, attributes
from audit_log
where tenant_id = :tenant_id
  and action = 'workflow.run.start'
  and resource_id = :run_id;
```

Notes:

- Always set a time window. The replica is large; unconstrained
  queries time out.
- Use the partial indexes per `tenant_id` + `ts` + `action`. The
  EXPLAIN plan should show an index scan; if not, narrow further.
- Never `select *` on a large slice; export structured columns.

### 4. Cross-reference with other systems

Audit log alone often answers *what*; pair with other sources for
*why*:

| Question | Audit log + … |
| --- | --- |
| What did the user *see*? | Audit (intent) + Loki (request log) + screen recording (rare) |
| What did the LLM *return*? | Audit (call recorded) + Langfuse (full trace + token I/O) |
| Did the webhook *deliver*? | Audit (outbox emission) + outbox table + vendor delivery report |
| Did the queue actually *run*? | Audit (job acked) + BullMQ history + worker logs |
| Was the data *exfiltrated*? | Audit (read events) + R2 access logs + DNS egress logs |

Quote the matching `request_id` / `span_id` from the audit row to
join with Loki / Sentry / Langfuse.

### 5. Verify the hash chain

For any forensic export, verify the chain segment **before** sharing
results:

```bash
teskel audit verify-chain \
  --tenant 01J9X3... \
  --start "2026-04-01T00:00:00Z" \
  --end   "2026-05-01T00:00:00Z" \
  --output ./tmp/chain-report.json
```

The verifier:

1. Iterates rows in `ts` order within the slice.
2. Recomputes `hash` for each row from `(content || hash_prev)`.
3. Asserts equality with stored `hash`.
4. Asserts `hash_prev[N] == hash[N−1]`.
5. (Phase 4+) Verifies the per-period HSM signature.

If verification fails:

- **Stop.** Do not share any export.
- File a Sev1 immediately (see
  [`incident-sev1`](../incident-sev1/SKILL.md)). Audit-log
  corruption is one of the most serious classes of incident.
- Escalate to Security + Platform leadership.
- Capture the failing row + the rows around it for forensic
  analysis (read-only).

If verification succeeds, attach the chain report to the export.
Auditors expect to see it.

### 6. Anonymize before sharing

Before any export leaves the trust boundary:

- Hash actor IPs (SHA-256 + per-export salt).
- Redact `actor_meta.user_agent` to coarse buckets.
- Strip session tokens entirely.
- Replace email-shaped strings inside `attributes` with hash.
- Convert internal IDs to opaque tokens if the recipient doesn't
  need them.

The CLI has `--anonymize <profile>` flags for common contexts
(DSAR, regulator, customer dispute, internal). Pick the right one.

### 7. Produce the report

Standard report format:

- **Header**: query metadata (scope, time window, run by, on
  behalf of, ticket id).
- **Chain verification**: pass / fail / partial coverage.
- **Summary stats**: count of rows, unique actors, action
  histogram.
- **Body**: rows in chronological order, redacted appropriately.
- **Footer**: hash of the report file itself, signed by the
  operator's key.

Output file lives at `/secure-export/audit/<tenant>/<ticket>.jsonl`
plus a `.report.md` companion. Both are encrypted at rest.

Sharing path is per the request type:

- **DSAR**: per [`gdpr-data-request`](../gdpr-data-request/SKILL.md)
  delivery channel.
- **Regulator**: per Legal's chain of custody.
- **Internal investigation**: stays in `secure-export`; viewers
  RBAC-gated.
- **Customer dispute**: customer-facing summary in their language;
  raw rows on request.

### 8. Common forensic patterns

**Who logged in from a new IP?**

```sql
select ts, actor_id, actor_meta->>'ip' as ip
from audit_log
where tenant_id = :tenant_id
  and action = 'auth.login'
  and not (actor_meta->>'ip')::inet <<= any (
    select known_subnet from tenant_known_subnets where tenant_id = :tenant_id
  )
order by ts desc;
```

**Did a service principal escalate scope?**

```sql
select ts, actor_id, attributes
from audit_log
where tenant_id = :tenant_id
  and actor_kind = 'service'
  and action = 'rbac.scope.elevated'
order by ts desc;
```

**Marketplace install + later refund — sequence?**

```sql
select ts, action, attributes
from audit_log
where tenant_id = :tenant_id
  and resource_kind = 'marketplace_listing'
  and resource_id = :listing_id
order by ts asc;
```

**LLM gateway calls outside business hours (cost spike check):**

Use Langfuse first (cost is captured there). Use audit log to
correlate to the *who* (which user / workflow / API key).

### 9. Performance tips

- Stay within tenant + time bounds.
- Filter by `action` prefix when you can — `action like 'rbac.%'`
  uses the partial index.
- Use `actor_id` filters before `attributes ->>` filters; JSONB
  ops are slower.
- Page large exports (`limit` + `offset` is fine on indexed
  ordering for read-only forensics).
- Run heavy queries during off-peak; the replica is shared.

### 10. Retention boundaries

Per [`docs/data/retention.md`](../../../docs/data/retention.md)
Class C:

- 3 years default; 7 years on Business+ plans.
- DSAR erasure does **not** redact the audit log of the erasure
  itself; it does pseudonymize the subject's actor_id within audit
  rows older than the deletion event, per the privacy policy.
- Backups (per Class H) preserve the audit log for the WAL window.

A query that returns zero rows for a known event might mean:

- The event didn't happen (good answer).
- The event predates the tenant's retention window (record this in
  the report).
- Pseudonymization removed the actor identity (record this; the
  *action* row is still there).

Never assume "no rows" means "didn't happen" without checking
retention.

### 11. Operator hygiene

- Run from the operator console, not your laptop. The console
  audit-logs *itself*, so your queries appear in the chain.
- Store exports in `secure-export`, never in your home directory.
- Delete exports from your local machine after the ticket closes.
- Never paste audit rows into Slack / chat / email; share via the
  encrypted report path.
- If you accidentally view PII you didn't need to, document it in
  the ticket (small disclosures aren't violations; un-documented
  ones are).

### 12. Reporting back

After the ticket closes:

- Attach the report file path to the ticket.
- Note the chain verification result.
- Note any anomalies even if not part of the original question
  (e.g. you noticed an old `hash_prev` mismatch dating to a
  migration — file a Sev2 to investigate).
- For DSAR: confirm the requester received the export within SLA.
- For incidents: link the audit-log evidence into the postmortem
  (per [`write-postmortem`](../write-postmortem/SKILL.md)).

---

## Pitfalls

- **Querying without a ticket.** Every probe is audit-logged. A
  ticket-less probe shows up in audit-log governance review.
- **Forgetting the time window.** Unconstrained queries can leak
  cross-tenant data via timing or cause the replica to fall behind.
- **Trusting "no rows" without checking retention.** Class C floor
  is 3 y; older events legitimately won't be there.
- **Skipping chain verification on exports.** Auditors will ask;
  if you can't prove integrity, the export is worthless.
- **Leaking actor IPs / user-agents in customer-facing reports.**
  Always anonymize per the recipient.
- **Treating audit log as a debug log.** Loki is for app logs.
  Audit log is for *intent*; spamming it with low-value events
  inflates cost and dilutes signal.
- **Running heavy queries on the primary cluster.** Use the
  read-only replica.
- **Editing the audit log.** Never. If a row is wrong, append a
  *correction* row (`action = 'audit.correction'`) referring to
  the prior row. The chain stays intact.

---

## Done when

- The query was scoped (tenant + time window + action filter).
- The hash chain was verified for the slice.
- The report was generated, anonymized to the recipient's profile,
  and stored in `secure-export`.
- The originating ticket links to the report path.
- For DSARs / regulators: the recipient received the export within
  SLA.
- Operator's own queries appear in the audit log of the
  investigated tenant.

---

## References

- [`AGENTS.md` §9](../../AGENTS.md#9-security--data-rules).
- [`docs/data/retention.md`](../../../docs/data/retention.md) Class C.
- [`docs/architecture/multi-tenancy.md`](../../../docs/architecture/multi-tenancy.md)
  §8 (cross-tenant operations).
- [`docs/security/rbac-matrix.md`](../../../docs/security/rbac-matrix.md).
- `gdpr-data-request`, `tenant-offboarding`, `incident-sev1`,
  `incident-sev2`, `triage-bug`, `write-postmortem`.
- Plan §50, §53, §94.
