---
name: add-runbook
description: Author a Grafana/Better Stack alert-attached runbook in docs/runbooks/. Each runbook documents triggers, blast radius, dashboards, immediate mitigations, escalation, and postmortem expectations. Use when a new alert is added or an incident reveals a missing runbook.
---

# add-runbook — write a runbook

> Plan ref: [Sec. 41 (On-call & runbooks)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#41-on-call--runbooks),
> [Sec. 39 (Observability)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#39-observability-stack),
> incident-sev1 SKILL.md.

> **Hard rule:** every alert must link to a runbook URL in its
> `runbook_url` annotation. Alerts without runbooks fail
> alert-rule CI lint.

## File location

`docs/runbooks/<area>/<short-slug>.md`

Examples:
- `docs/runbooks/api/high-error-rate.md`
- `docs/runbooks/db/replica-lag.md`
- `docs/runbooks/queue/dlq-overflow.md`
- `docs/runbooks/ai/openrouter-saturation.md`

## Required template

Copy this template and fill it in. Don't omit sections; mark
"N/A" instead.

```markdown
# Runbook: <Alert name>

> Severity: Sev1 | Sev2 | Sev3
> Owner: @yahyaxbt (rotate via on-call)
> Linked alert(s): <Grafana alert URL or rule path>
> Linked dashboard(s): <Grafana dashboard URL>
> Last reviewed: 2026-MM-DD

## What it means

1–2 sentences explaining what the alert is detecting in plain
English. Avoid Greek-letter-only descriptions.

## Why it matters (blast radius)

Who is affected, and how:
- User-facing? Which flows?
- Internal? Which services?
- Revenue impact? (e.g., billing path → all paid orgs)
- Data integrity risk?

## Immediate triage (≤5 min)

1. Acknowledge the alert in PagerDuty / Better Stack.
2. Open the linked dashboard.
3. Check the last 30 min: deploys, feature-flag flips, infra events.
4. Confirm the alert is real (not a flapping false positive). If
   false positive, silence ≤30 min and tune the rule in a follow-up.

## Mitigation playbook

In order of preference (least invasive first):

1. **<Mitigation 1 name>**
   - When: <when this is the right action>
   - Steps:
     ```
     <exact command / link / dashboard click>
     ```
   - Verify: <how to confirm it worked>

2. **<Mitigation 2 name>**
   - ...

3. **Rollback** (last resort if all else fails)
   - Trigger: <criteria>
   - Steps: see `release-canary` SKILL → "Rollback".

## Escalation

| Trigger | Action |
| --- | --- |
| Mitigation 1 + 2 don't work in 15 min | Page secondary on-call |
| Customer-facing impact >5 min | Open Sev1 (see `incident-sev1` SKILL) |
| Data corruption suspected | Page DB owner immediately |
| Vendor outage suspected | Check vendor status page; mention in incident channel |

## Diagnostic queries

Drop in the exact PromQL/LogQL/SQL the responder needs:

```
# error rate by route, last 30m
sum by (route) (rate(http_requests_total{status=~"5.."}[5m]))
  / sum by (route) (rate(http_requests_total[5m]))

# top noisy users last 1h
SELECT user_id, COUNT(*) FROM api_audit_log
WHERE ts > now() - interval '1 hour'
GROUP BY user_id ORDER BY 2 DESC LIMIT 10;
```

## Common false positives

- <Cause 1>: identifier + how to confirm.
- <Cause 2>: ...

## Recent incidents

| Date | Sev | Cause | PR / postmortem |
| --- | --- | --- | --- |
| 2026-04-12 | Sev2 | Coolify deploy stuck | docs/postmortems/2026-04-12-coolify.md |

## Glossary

(Only if the runbook uses team-specific jargon.)
```

## Steps

1. **Identify the alert.** If the alert doesn't exist yet, write it
   first in `infra/grafana/alerts/<slug>.yaml` with a `runbook_url`
   that points to the runbook you're about to author.

2. **Pick the slug + path.** `<area>/<short-slug>.md`. Keep the slug
   imperative or symptom-named, not metric-named:
   - good: `db/replica-lag.md`, `queue/dlq-overflow.md`
   - avoid: `pg_replication_lag_seconds_above_60.md`

3. **Fill the template.** Don't write fluff. The runbook must let an
   on-call who has never seen this alert succeed in <15 minutes.

4. **Make mitigations concrete.** Every mitigation has a literal
   command or button-click path. "Check if Postgres is slow" is not
   a mitigation. "Run `pg_stat_statements_reset()` in DB console
   from Coolify → Database → Console; verify p95 query time on
   `db-overview` dashboard drops below 200ms within 5 minutes" is.

5. **Test it.** Have someone unfamiliar with the alert walk through
   the runbook in dry-run. Time them. If they can't get to step 3 in
   2 minutes, rewrite.

6. **Wire it to the alert.**
   ```yaml
   # infra/grafana/alerts/api-high-error-rate.yaml
   - alert: api_high_error_rate
     expr: ...
     for: 5m
     annotations:
       summary: "API 5xx error rate >5% for 5m"
       runbook_url: "https://github.com/yahyaxbt/teskel/blob/main/docs/runbooks/api/high-error-rate.md"
       dashboard_url: "https://grafana.teskel.dev/d/api-overview"
   ```

7. **Add to the runbook index.** `docs/runbooks/README.md` — table
   of contents grouped by area. CI fails if a runbook isn't listed.

8. **Quarterly review.** Every runbook has a "Last reviewed" date.
   Quarterly, walk through all runbooks; update commands that drift,
   delete dead links, refresh dashboard URLs.

## Examples worth modeling on

- `docs/runbooks/api/high-error-rate.md` — generic template, good
  starting point.
- `docs/runbooks/db/replica-lag.md` — DB-specific.
- `docs/runbooks/queue/dlq-overflow.md` — async/job-specific.
- `docs/runbooks/ai/openrouter-saturation.md` — AI vendor-specific.
- `docs/runbooks/billing/stripe-webhook-failures.md` — payments.

## Pitfalls

- "Restart the service" as the only mitigation — restart is a
  symptom, not a fix. Diagnose first.
- Runbook that's just a metric definition + nothing actionable.
- Runbook that's a 2,000-word essay — on-call won't read it at 3am.
- Stale commands that no longer work — quarterly review or the
  runbook becomes a liability.
- Linking to internal-only docs that on-call can't access from a
  phone — keep critical info inline in the runbook.

## Done when

- Runbook lives at `docs/runbooks/<area>/<slug>.md`.
- Alert rule references the runbook URL.
- Index page updated.
- Dry-run by a non-author succeeds in <15 min.
- Last reviewed date set.
