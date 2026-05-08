---
name: incident-sev1
description: Sev1 incident response runbook for TESKEL — production down or severely degraded, customer-impacting. Use when paged for Sev1 or when you observe a Sev1-class symptom. Defines the role of Incident Commander, comms cadence, mitigation tree, and postmortem.
---

# incident-sev1 — Sev1 incident response

> Use this when production is down, severely degraded, or there is a
> credible data-loss / security incident. Plan ref:
> [Sec. 41 (Incident Response)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#41-incident-response--runbooks).

## Severity definitions (recap)

- **Sev1**: prod down, data loss imminent, security breach in progress,
  >10% of users affected, payments / billing core broken. Page on-call
  immediately.
- **Sev2**: degraded but not down, single major feature broken,
  <10% users affected, no data loss.
- **Sev3**: single user / single feature edge case, no data risk.

## Roles

- **Incident Commander (IC)**: drives the response. First on-call to
  acknowledge becomes IC.
- **Communicator**: keeps customers + status page updated. Often the IC
  in early phases; delegate when possible.
- **Subject-matter responders**: pulled in by IC as needed.

## Step 0 — Acknowledge (within 5 minutes of page)

```text
:rotating_light: SEV1 ack — IC: @<your-handle>
Symptom: <one line>
Channel: #incident-<utc-timestamp>
```

Open a fresh Slack/Discord channel `#incident-<utc-timestamp>`. Pin the
status, IC, and start time at the top. Do **not** debug in DMs.

## Step 1 — Stabilize (first 15 minutes)

Goals (in order):

1. **Stop bleeding.** If a recent deploy is the suspect cause →
   roll back (see [`release-canary`](../release-canary/SKILL.md)
   "Rollback steps"). It's better to roll back unnecessarily than to
   keep degrading.
2. **Confirm scope.** Check Sentry, Grafana, status page, customer
   reports. Are we looking at:
   - Single region / single service / global?
   - 5xx surge, auth failures, queue backlog, DB errors, AI provider
     outage, Stripe webhook failures?
3. **Update status page** within 10 minutes of confirmation:
   - Set the affected component to "Major outage" or "Degraded".
   - Public message: short, factual, no jargon. Include "we are
     investigating" and a follow-up time.

## Step 2 — Mitigate (T+15m → T+60m)

Run the relevant runbook from `docs/runbooks/`:

- `db-primary-down.md`
- `redis-oom.md`
- `bullmq-backlog.md`
- `ai-provider-outage.md`
- `stripe-webhook-failure.md`
- `sandbox-provider-outage.md`
- `region-outage.md`

If no runbook matches, walk down this tree:

```text
A. Is recent deploy the cause?
   YES → Roll back (release-canary skill, "Rollback steps").
   NO  → continue.
B. Is an external dependency failing?
   YES → Activate the documented circuit breaker / failover (e.g.,
         alternate model in OpenRouter; failover Stripe to retry queue).
   NO  → continue.
C. Is the DB or queue saturated?
   YES → Apply documented scale-out (PgBouncer, BullMQ concurrency,
         add replicas). If saturation is from a runaway tenant →
         apply the per-tenant rate limit kill-switch.
   NO  → continue.
D. Is auth / session broken?
   YES → Check Better Auth + cookie domain config; check identity
         provider status if SAML.
   NO  → continue.
E. Is data being corrupted?
   YES → STOP all writes to the affected table (set service to
         maintenance), restore from latest snapshot, then forward-fix.
   NO  → escalate to broader engineering for diagnosis.
```

Throughout: **post a status update in the incident channel every 15
minutes**, even if "no change".

## Step 3 — Communicate

External:

- Status page update at **T+10m, T+30m, T+60m, then every 30m** until
  resolved.
- Send email to Pro+ paying customers if downtime exceeds 30 minutes.
- Tweet from `@teskel_app` if downtime exceeds 60 minutes (or if
  customers tweet first).

Internal:

- `#incident-<ts>` channel: factual updates only. No speculation.
- `#leadership` ping at T+30m if not resolved.
- Founder ping at T+60m if not resolved or if revenue-critical.

## Step 4 — Resolve

When the symptom clears for ≥15 minutes:

1. Update status page → "Resolved", with a 1-paragraph summary of what
   happened and what was done.
2. Internal channel: `:white_check_mark: Sev1 resolved at <UTC>.
   Postmortem owner: @<handle>. Due: <date+5BD>.`
3. Tag the incident in PagerDuty / Better Stack with cause category.

## Step 5 — Postmortem (within 5 business days)

Open `docs/runbooks/postmortems/YYYY-MM-DD-<short-slug>.md` from the
template. Sections:

1. Summary (2 sentences).
2. Timeline (UTC, 1-line entries).
3. Impact (who, how many, $ if applicable, SLA breach? data loss?).
4. Root cause(s) — primary + contributing.
5. What went well.
6. What went poorly.
7. Action items: owner, due date, Linear link. Each must be tracked.
8. Lessons / playbook / runbook updates.

The postmortem is **blameless**. Focus on systemic fixes, not
individuals.

## Pitfalls

- "Just one more minute" before rolling back — costs more than rolling
  back unnecessarily.
- Debugging in DMs — loses institutional memory.
- Status page silence — customers will assume worst-case.
- Skipping the 15-minute heartbeat in the incident channel.
- Postmortem written but action items never landed — IC follows up at
  T+30 days.

## Forbidden during incident

- Force-pushing to `main`.
- Running destructive SQL on prod ("just to clean up").
- Deploying unrelated changes.
- Rotating critical secrets without a paired update plan.
- Posting partial / speculative root cause to customers.

If you must do any of the above, get explicit approval from the
maintainer in the incident channel and pin the message.

## Done when

- Symptom resolved + 15 minutes clean.
- Status page → Resolved.
- Postmortem owner assigned + due date set.
- Incident channel summary pinned.
