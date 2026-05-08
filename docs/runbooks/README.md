# Runbooks

Operational playbooks linked from production alerts. Every alert has a
`runbook_url` annotation that resolves to a file under this directory.

> Author with the [`add-runbook`](../../.agents/skills/add-runbook/SKILL.md)
> skill. Quarterly review with the [`gameday-drill`](../../.agents/skills/gameday-drill/SKILL.md)
> skill.

## Layout

```
docs/runbooks/
  README.md                  # this file
  api/                       # apps/api alerts
    high-error-rate.md
    p95-latency.md
    rate-limit-exhausted.md
  workers/                   # BullMQ workers + cron
    queue-stalled.md
    cron-heartbeat-stale.md
    dlq-growing.md
  db/                        # Postgres
    high-connection-count.md
    replica-lag.md
    long-running-tx.md
    backup-verification-failed.md
  ai/                        # AI gateway / Langfuse
    openrouter-degraded.md
    prompt-budget-exceeded.md
    eval-regression.md
  integrations/              # third-party
    stripe-webhook-failures.md
    resend-bounce-spike.md
    posthog-ingest-lag.md
  cron/                      # one runbook per cron.<id>
    billing-reconcile-invoices.md
  infra/                     # platform-level
    coolify-deploy-stuck.md
    redis-memory-pressure.md
    r2-egress-spike.md
    cdn-cache-hit-low.md
  security/
    auth-anomaly.md
    audit-chain-broken.md
    suspected-data-exfil.md
```

## Index (alphabetical)

| Slug | Path | Linked alerts | Owner |
| --- | --- | --- | --- |
| _placeholder — populated as alerts are wired_ | | | |

This index is auto-generated nightly from the runbook frontmatter
(planned: `pnpm teskel runbooks index`). Manual edits are
overwritten.

## Conventions

- One runbook per **alert family**. If two alerts trigger the same
  response, they share a runbook.
- File slug = the alert name lowercased (`HighErrorRate` →
  `high-error-rate.md`).
- Required template enforced by CI lint (see
  [`add-runbook`](../../.agents/skills/add-runbook/SKILL.md)).
- Every runbook has a public URL of the form
  `https://teskel.dev/docs/runbooks/<area>/<slug>` baked into the
  alert as `runbook_url`.

## Quarterly review

Every 90 days, the on-call rotation walks the index and:

1. Marks runbooks as **fresh** (referenced & accurate), **stale**
   (commands not run in 90 days), or **dead** (alert removed).
2. Rerun any **stale** runbook commands against staging in a
   [`gameday-drill`](../../.agents/skills/gameday-drill/SKILL.md)
   slot.
3. Removes **dead** runbooks; the alert that referenced them was
   already removed (CI lint catches drift).

The review notes go in `docs/runbooks/_reviews/<YYYY-Q#>.md`.

## Writing a new runbook

Use the skill: [`add-runbook`](../../.agents/skills/add-runbook/SKILL.md).
The skill enforces the template, the diagnostic-query format, and
the alert-wiring step.

## Related

- [`incident-sev1`](../../.agents/skills/incident-sev1/SKILL.md) —
  major incident response.
- [`incident-sev2`](../../.agents/skills/incident-sev2/SKILL.md) —
  partial-impact incident response.
- [`release-hotfix`](../../.agents/skills/release-hotfix/SKILL.md) —
  emergency production fix.
- [`db-restore-pitr`](../../.agents/skills/db-restore-pitr/SKILL.md) —
  database point-in-time recovery.
