# Current Phase

> Updated: 2026-05-08 by maintainer.
> Format: machine-readable line first, then human notes.

```yaml
phase: 0
phase_name: Foundation
phase_started_at: TBD          # ISO date when first commit of phase 0 lands
phase_window: Week 1-4
status: planning               # one of: planning | active | freeze | done
plan_section: 88               # Plan reference (TESKEL_FULLSTACK_BUILD_BREAKDOWN.md Sec. 88)
release_target: v0.1.0
freeze_window: false
allowed_scopes:
  - monorepo
  - db
  - auth
  - rls
  - deploy
  - ci
  - observability
out_of_scope:
  - workflow_runtime
  - marketplace
  - billing
  - sandbox
  - ai_gateway_runtime
  - visual_builder
  - sdk
  - cli
  - saml_sso
  - multi_region
```

## What this means for agents

You are in **Phase 0 — Foundation**. Allowed work is limited to the
scopes above. If a request asks you to implement something marked
`out_of_scope`, **stop and surface to a human owner** before writing
code. See [`AGENTS.md` §12](../../AGENTS.md#12-phase-awareness--scope-control)
and [`AGENTS.md` §17](../../AGENTS.md#17-escalation-triggers).

## Exit criteria for Phase 0

To close Phase 0 and tag `v0.1.0`, all of the following must be true
(see Plan Sec. 88.4):

- [ ] `https://app.teskel.app` serves a hello-world build from `main`.
- [ ] Sign-up + sign-in (email/password and magic-link) works end-to-end
      with 2FA opt-in.
- [ ] Cross-tenant isolation test passes (Org A cannot read Org B data
      via Postgres RLS).
- [ ] CI green: `pnpm turbo run lint typecheck test build` passes on
      `main` in ≤6 minutes.
- [ ] Sentry, OpenTelemetry, Loki, Grafana, PostHog all wired and
      receiving events from a live request.
- [ ] ADR-0001 (Better Auth), ADR-0002 (Drizzle), ADR-0003 (BullMQ),
      ADR-0009 (Next.js), ADR-0010 (Coolify) all merged at
      `docs/adr/`.
- [ ] Grafana "Phase 0 Health" dashboard live with 5xx, latency p95, DB
      connections, Redis memory, error rate per service.
- [ ] Backup + restore drill executed once on staging.

## When Phase 0 is done

The maintainer will:

1. Tag `v0.1.0`.
2. Update this file: `phase: 1`, set `phase_name: Core Workflow + AI`,
   `phase_window: Week 5-10`, `plan_section: 89`, refresh
   `allowed_scopes`.
3. Open the Phase 1 kickoff PR.
