---
name: triage-bug
description: Triage an incoming bug report — assign severity, reproduce, find root cause, classify, and route to the right owner. Use this whenever a bug arrives via GitHub issue, customer ticket, or paging system.
---

# Skill — Triage a bug

> **Plan refs:** §41 (on-call & runbooks), §75 (failure modes
> & rollback), §97 (pitfalls per phase).
> **Related skills:** `incident-sev1`, `incident-sev2`,
> `write-postmortem`, `audit-log-query`, `review-pr`.

A triage decides three things, in order:

1. **How severe is this?** (Sev1 / Sev2 / Sev3 / Sev4.)
2. **Who owns the fix?** (Surface owner per [`docs/runbooks/`](../../../docs/runbooks/).)
3. **What is the smallest reproduction?**

The output of triage is **never** a fix. The output is an actionable
ticket with severity, owner, repro, expected impact, and links to
the supporting evidence. The fix happens after.

---

## When to invoke

- A new bug appears in GitHub issues (Bug template).
- A customer ticket arrives via support inbox.
- A pager fires (Better Stack / incident.io alert).
- A teammate DMs "is this a bug?" (translate to issue first; do not
  triage in DM).
- A new failure shows up in Sentry / Loki dashboards without a
  matching ticket.

You do **not** invoke this skill while actively fighting a fire —
in a Sev1 you are *responding*, not *triaging*. Use
[`incident-sev1`](../incident-sev1/SKILL.md) instead.

---

## Severity rubric

| Severity | Definition | Response |
| --- | --- | --- |
| **Sev1** | Customer-facing outage or data loss; SLO red; affects ≥ 5% of tenants OR a **top-tier tenant**¹. | Page on-call immediately. See [`incident-sev1`](../incident-sev1/SKILL.md). |
| **Sev2** | Major feature broken; workaround exists; affects < 5% of tenants. SLO yellow. | Same-day fix; canary today. |
| **Sev3** | Minor feature broken or cosmetic; degrades UX; safe to bundle in next train. | Next train (≤ 1 week). |
| **Sev4** | Nuisance, paper-cut, dev-only, or "as designed but confusing." | Backlog with target phase. |

If a triage candidate spans two severities (e.g. cosmetic for most
tenants but data-loss for a long-tail enterprise), pick the **higher**
and document the rationale.

> ¹ **"Top-tier tenant"** = any tenant on the **Business** or
> **Enterprise** plan (per [`docs/billing/plans.md`](../../../docs/billing/plans.md)),
> *or* any tenant explicitly flagged `priority = true` in the
> internal ops console (used for design partners, regulated
> deployments, and contractually committed accounts). When in
> doubt, treat as top-tier.

---

## Steps

### 1. Acknowledge fast

Within **15 minutes** of arrival, leave a comment on the report:

- "Triaging — assigning by EOD."
- "Pulled into Sev1 — see incident #N."
- "Closing as duplicate of #M."
- "Closing as not-a-bug — see [link]; please reopen if the
  behavior repeats with a fresh report."

The comment is for the reporter (so they know they're heard) and
for the team (so two triagers don't double-handle).

### 2. Read the report end-to-end

Even if the title looks obvious, read the full body. Look for:

- **Specific IDs** (request_id, run_id, trace_id, tenant_id) — these
  are gold for repro.
- **Environment** (prod / staging / canary / local). Bugs in local
  are not bugs; bugs in canary may already be auto-rolling-back.
- **Timestamp** — pivot for log/metric search.
- **Reporter context** — first-time reporter vs ongoing customer
  vs internal teammate vs scanner.
- **PII redaction** — if the report has unredacted PII, redact in
  the comment immediately and ping the reporter privately.

### 3. Classify the surface

Map the report to one of the surfaces in
[`docs/architecture/threat-model.md`](../../../docs/architecture/threat-model.md):

- web (anon / auth)
- API
- workers / workflow runtime
- AI gateway
- sandbox
- marketplace
- identity / auth / RBAC
- storage
- integrations / outbound
- inbound webhooks
- admin
- customer apps (run-time of buyer apps)

The surface decides the **owner**. Owners live in
[`.agents/state/owners.md`](../../state/owners.md).

### 4. Assess severity

Walk the rubric (above). Use these questions:

- Is data being lost / corrupted? **Sev1** unless single-tenant,
  reversible, and the customer is OK with the wait.
- Is auth / RBAC being bypassed? **Sev1**. Always.
- Is billing being mis-charged? **Sev1**. Always.
- Is the platform unable to serve a class of requests at all?
  **Sev1**.
- Is one feature broken with a workaround? **Sev2**.
- Is one feature broken without a workaround? **Sev2** by default;
  promote to Sev1 if widespread.
- Is the bug in a non-default flag rollout? **Sev3** at most;
  toggle the flag off via [`add-feature-flag`](../add-feature-flag/SKILL.md).
- Is the bug in a deprecated path slated for removal? **Sev4**;
  consider closing.

If unsure, **escalate up** — Sev2 with a teammate's eye is cheaper
than Sev3 that blooms.

### 5. Reproduce

Try the reported steps in **staging** if the bug is non-prod-data
sensitive; in a **dedicated test tenant** in prod if it requires
real services. Never reproduce in a customer's tenant without
explicit consent + audit-log entry.

If you cannot reproduce:

- Ask the reporter for a **request_id** / **run_id** if not
  provided.
- Trace it via the [`audit-log-query`](../audit-log-query/SKILL.md)
  skill if it touched mutable state.
- Search Sentry / Loki for the symptom near the reported timestamp.

If still no repro after 30 min:

- Mark `triage:cannot-reproduce`.
- Ask the reporter for one more piece of info (steps, screen
  recording, browser, plan tier).
- If after 7 days no response, close `not-actionable`. If the
  symptom recurs in metrics, reopen.

### 6. Find the root cause (or first hypothesis)

Read **one level deeper** than the symptom:

- Symptom: "Workflow run hangs at step 3."
- One level deeper: "Step 3 is an HTTP node calling vendor X; the
  response timed out and the retry loop is stuck."

Don't try to fix yet — the goal is enough understanding to:

- Confirm the severity.
- Confirm the owner.
- Phrase a story with concrete acceptance criteria.

If the root cause is in a vendor (Stripe, OpenRouter, Resend), find
the vendor's status page link, attach to the ticket, and note the
vendor incident ID. Some Sev2s are "wait for vendor + comms".

### 7. Classify the bug

| Class | Examples |
| --- | --- |
| **Logic** | wrong branch taken, off-by-one |
| **Data** | wrong row updated, RLS bypass, missing FK |
| **Concurrency** | race, lost update, deadlock |
| **Capacity** | timeout, queue backlog, cost spike |
| **UX** | confusing label, wrong copy, missing affordance |
| **Vendor** | dependency outage, API change |
| **Spec** | reality matches code; spec is wrong |
| **Security** | should be reported privately — see [`SECURITY.md`](../../../SECURITY.md) |

Classification feeds the quarterly trend report (see
[`write-postmortem`](../write-postmortem/SKILL.md) §12).

### 8. Decide: fix, defer, or close

| Outcome | When |
| --- | --- |
| **Fix in current train** | Sev1 / Sev2 confirmed; owner has bandwidth. |
| **Fix in next train** | Sev3; no rollback urgency. |
| **Fix in a future phase** | Sev4 + the right fix is out-of-scope for current phase (per [`.agents/state/current-phase.md`](../../state/current-phase.md)). |
| **Defer pending RFC** | The "right fix" needs design (e.g. concurrency model change). File RFC; link both ways. |
| **Close as won't-fix** | The behavior is intentional + documented. Add to FAQ if asked twice. |
| **Close as duplicate** | Existing issue covers it. |
| **Close as not-a-bug** | Misuse, configuration, environment. Be kind; explain. |
| **Close as security** | Privately re-route; close public issue with a polite redirect to [`SECURITY.md`](../../../SECURITY.md). |

### 9. Write the actionable ticket

The ticket must include:

- Severity (with justification).
- Surface + owner.
- Smallest reproduction (steps + expected vs actual).
- Customer impact (how many tenants, what surface, what they see).
- First hypothesis (root cause guess).
- Linked artifacts (Sentry ID, Langfuse trace, Grafana panel link,
  audit-log query).
- Suggested fix sketch (one paragraph; the implementer can override).
- Acceptance criteria (testable bullets).
- Risk if not fixed (silent risk vs visible).

If the ticket can't be filled like this, you don't have enough to
triage; go back to step 5.

### 10. Route + label

- Assign to the **surface owner**, not the reporter.
- Apply labels: `severity:sev1|sev2|sev3|sev4`, `surface:<name>`,
  `class:<class>`.
- Link to phase if applicable: `phase:0..6`.
- Cross-link to dashboards / runbooks / postmortems if any.
- Drop a one-liner in the appropriate `#engineering-*` channel for
  Sev1/Sev2 visibility.

### 11. Communicate back

Reporter gets:

- "Confirmed Sev<N>. Owner: <team>. Fix targeted for: <train>."
- For paying customers: also goes via support tooling.
- For Sev1: a status-page entry per [`incident-sev1`](../incident-sev1/SKILL.md).

### 12. Track to closure

The triager doesn't own the fix, but **does** own the ticket's
state. Stale Sev2s for >5 business days get pinged. Sev1s are
already on incident comms.

When the fix lands:

- Verify the linked PR closes the issue.
- Confirm the test added covers this exact regression.
- For Sev1 / Sev2: ensure a postmortem is filed (see
  [`write-postmortem`](../write-postmortem/SKILL.md)).

---

## Common patterns

| Pattern | Actual root cause | Hint |
| --- | --- | --- |
| "It's slow" | DB query missing index, or LLM cold-start | Compare p50 vs p95; index plan. |
| "It works locally but not in prod" | Env var, RLS context, or worker tenant binding | Check `db.withTenant` is set in worker. |
| "It's flaky" | Race in idempotency or outbox dedup | Replay test; idempotency key audit. |
| "Random user got someone else's data" | RLS test missing for a join — Sev1, escalate immediately. | |
| "LLM gave a wrong answer" | Prompt slot regression; eval drift | [`run-eval`](../run-eval/SKILL.md). |
| "Webhook didn't arrive" | Outbox stuck, signature mismatch, vendor outage | [`add-webhook-receiver`](../add-webhook-receiver/SKILL.md) check. |
| "Refund didn't refund" | Stripe webhook duplicated or missed; reconciliation drift | `audit-log-query`. |
| "Login redirected to wrong tenant" | Session cookie domain or org switcher state | Always Sev1. |

---

## Pitfalls

- **"Just fix it" before triage.** A fix without triage doesn't get
  a postmortem and doesn't show up in trend reports.
- **Triaging in DM.** Always pull into an issue first — DMs have no
  audit, no labels, no severity.
- **Reproducing in customer tenant without consent.** Never. Use
  the audit-log query path, or ask permission.
- **Asking the reporter to gather all the data.** Most reporters
  give you everything they have; *you* have access to logs they
  don't.
- **Promoting Sev3 → Sev1 to "be safe".** Inflates pager fatigue.
  If unsure, Sev2 with a 24h check-in.
- **Closing without a one-line explanation.** Even "won't-fix"
  deserves a kind reason and a link.

---

## Done when

- Severity assigned.
- Surface + owner known.
- Repro recorded (or `cannot-reproduce` documented with what was
  tried).
- First hypothesis written.
- Ticket has labels + acceptance criteria.
- Reporter notified.
- For Sev1 / Sev2 — incident channel + status page updated.

---

## References

- [`AGENTS.md` §17](../../../AGENTS.md#17-escalation-triggers).
- [`docs/observability/slo-sli.md`](../../../docs/observability/slo-sli.md).
- [`docs/observability/dashboards.md`](../../../docs/observability/dashboards.md).
- `incident-sev1`, `incident-sev2`, `write-postmortem`,
  `audit-log-query`, `run-eval`.
- Plan §41, §75, §97.
