# Owners

> Updated: 2026-05-08. Source for `CODEOWNERS` and escalation routing.

## Maintainers (root owners)

| Handle | Role | Areas | Timezone |
| --- | --- | --- | --- |
| @yahyaxbt | Founder, Tech Lead | All — final decision authority | Asia/Jakarta (UTC+7) |

## Area owners (set per phase as the team grows)

| Area | Files / packages | Owner | Backup |
| --- | --- | --- | --- |
| Plan / ADR / RFC | `TESKEL_FULLSTACK_BUILD_BREAKDOWN.md`, `docs/adr/`, `docs/rfc/` | @yahyaxbt | TBD |
| Auth | `packages/auth/` | @yahyaxbt | TBD |
| DB / migrations | `packages/db/`, `**/*.sql` | @yahyaxbt | TBD |
| Billing / Stripe | `packages/billing/` | @yahyaxbt | TBD |
| AI / LLM | `packages/ai/` | @yahyaxbt | TBD |
| Workflow runtime | `packages/runner/`, `packages/queue/` | @yahyaxbt | TBD |
| UI / design system | `packages/ui/`, `apps/web/` | @yahyaxbt | TBD |
| Infra / deploy | `infra/`, `.github/workflows/` | @yahyaxbt | TBD |
| Security / compliance | `SECURITY.md`, `docs/runbooks/` | @yahyaxbt | TBD |

## Escalation matrix

| Severity | First responder | Escalation 30 min | Escalation 60 min |
| --- | --- | --- | --- |
| Sev1 (prod down) | On-call | Tech Lead | Founder |
| Sev2 (degraded) | On-call | Area owner | Tech Lead |
| Sev3 (single user issue) | Support | On-call | Area owner |

Pager rotation tool: TBD (PagerDuty or Better Stack on-call). Set in
Phase 4.

## How to update this file

1. Open a PR titled `chore(owners): <change>`.
2. Tag both the outgoing and incoming owner.
3. Update `CODEOWNERS` in the same PR if the area routing changes.
