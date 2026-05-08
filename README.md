# TESKEL

> **AI Startup Factory + Infrastructure Platform** — an operating system
> for building, launching, monetizing, and scaling AI-native businesses
> end-to-end.

TESKEL is **not** another website builder, AI wrapper, generic template
marketplace, or n8n / Vercel clone. It is a vertically integrated
platform that takes a founder from idea → live product → paying
customers, with first-class AI workflows, a creator-driven template
marketplace, and enterprise-grade ops baked in.

| | |
| --- | --- |
| Status | **Planning → Phase 0 (Foundation)** |
| Default region | Singapore (Indonesia / SEA primary) |
| License | Proprietary (TBD per maintainer) |
| Maintainer | [@yahyaxbt](https://github.com/yahyaxbt) |

---

## Read these first

If you are a **human contributor** (or onboarding):

1. [`TESKEL_FULLSTACK_BUILD_BREAKDOWN.md`](./TESKEL_FULLSTACK_BUILD_BREAKDOWN.md)
   — the master plan (≈5k lines, 9 parts, 99 sections + appendices). It
   is the single source of truth for product strategy, design,
   architecture, ops, security, growth, build plan, risks, and
   reference. Treat it as immutable; changes go through ADR.
2. [`AGENTS.md`](./AGENTS.md) — the operating manual for both human and
   AI contributors. Coding conventions, hard architecture constraints,
   forbidden actions, escalation triggers, decision cheatsheet.
3. [`.agents/state/current-phase.md`](./.agents/state/current-phase.md)
   — what's in scope right now. Stay inside the current phase.

If you are an **AI coding agent** (Devin / Cursor / Claude Code /
Copilot / Aider): read [`AGENTS.md`](./AGENTS.md) first. The relevant
agent-specific shim files (`CLAUDE.md`, `.cursorrules`,
`.github/copilot-instructions.md`) all point back to `AGENTS.md`.

For recurring tasks, follow the matching skill in
[`.agents/skills/`](./.agents/skills/) — 37 skills cover scaffolding
the monorepo, adding a package / table / API route / webhook / cron /
email / integration / page / UI component / builder block / prompt
slot / workflow node / feature flag / RBAC permission / e2e test /
runbook, publishing a marketplace template, rotating secrets, running
data backfills, restoring the DB, releasing a canary or hotfix,
responding to Sev1 / Sev2 incidents, processing GDPR requests,
running GameDay drills, authoring ADRs / RFCs / phase kickoffs,
running LLM eval suites, reviewing PRs, triaging bugs, querying
the audit log, off-boarding tenants, and writing postmortems.
See the [skills library index](./AGENTS.md#19-skills-library) for the
full lookup table.

### Where everything lives (quick map)

| Need | Path |
| --- | --- |
| Master plan | [`TESKEL_FULLSTACK_BUILD_BREAKDOWN.md`](./TESKEL_FULLSTACK_BUILD_BREAKDOWN.md) |
| Operating manual | [`AGENTS.md`](./AGENTS.md) |
| Current phase scope | [`.agents/state/current-phase.md`](./.agents/state/current-phase.md) |
| Skills (37) | [`.agents/skills/`](./.agents/skills/) |
| Agent navigation | [`.agents/README.md`](./.agents/README.md) |
| Glossary | [`.agents/glossary.md`](./.agents/glossary.md) |
| Architecture (C4, multi-tenancy, threat model) | [`docs/architecture/`](./docs/architecture/) |
| Architecture decisions (ADR) | [`docs/adr/`](./docs/adr/) |
| Design proposals (RFC) | [`docs/rfc/`](./docs/rfc/) |
| Phase briefs | [`docs/phases/`](./docs/phases/) |
| Stories (work units) | [`docs/stories/`](./docs/stories/) |
| Runbooks (alerts) | [`docs/runbooks/`](./docs/runbooks/) |
| Observability (SLO/SLI, dashboards) | [`docs/observability/`](./docs/observability/) |
| Release process (canary, hotfix) | [`docs/release/`](./docs/release/) |
| API conventions + idempotency | [`docs/api/`](./docs/api/) |
| Data retention + classes | [`docs/data/`](./docs/data/) |
| Marketplace manifest spec | [`docs/marketplace/`](./docs/marketplace/) |
| Plans + entitlement matrix | [`docs/billing/`](./docs/billing/) |
| Onboarding (engineer + agent) | [`docs/onboarding/`](./docs/onboarding/) |
| Legal / privacy / DPA notes | [`docs/legal/`](./docs/legal/) |
| Secrets registry | [`docs/security/secrets.md`](./docs/security/secrets.md) |
| RBAC matrix | [`docs/security/rbac-matrix.md`](./docs/security/rbac-matrix.md) |
| Contribution guide | [`CONTRIBUTING.md`](./CONTRIBUTING.md) |
| Code of Conduct | [`CODE_OF_CONDUCT.md`](./CODE_OF_CONDUCT.md) |
| Support paths | [`SUPPORT.md`](./SUPPORT.md) |
| Security policy | [`SECURITY.md`](./SECURITY.md) |

---

## What's in this repo

```text
.
├── apps/                   # Next.js web app, Hono API, docs site (built per phase)
├── packages/               # auth, db, ai, queue, runner, ui, shared, sdk, cli, etc.
├── seeds/templates/        # Source of seed marketplace templates
├── infra/                  # Coolify configs, Dockerfiles, Helm charts
├── docs/                   # ADR, RFC, runbooks
├── .agents/                # AI agent operational state + skills
├── .github/                # PR template, CI workflows, CODEOWNERS, copilot rules
├── AGENTS.md               # AI agent operating manual
├── CLAUDE.md               # Claude Code pointer
├── .cursorrules            # Cursor pointer
├── README.md               # This file
└── TESKEL_FULLSTACK_BUILD_BREAKDOWN.md   # Master plan
```

The full target structure is in [`AGENTS.md` §4](./AGENTS.md#4-repository-map-target-structure).
Folders not yet created exist only in plan form until the relevant phase
begins.

---

## Stack at a glance

(Settled defaults; see [`AGENTS.md` §5](./AGENTS.md#5-stack--library-decisions-settled)
for the full table and ADR links.)

- **Frontend** — Next.js 15 (App Router + RSC), TypeScript strict,
  Tailwind + shadcn/ui, Zustand + TanStack Query.
- **Backend** — Hono, Drizzle ORM, PostgreSQL 16 + pgvector, Postgres
  RLS for multi-tenancy.
- **Auth** — Better Auth (organizations, 2FA, passkey, magic-link).
- **Async** — BullMQ (Inngest as secondary for cron / step.ai).
- **AI** — OpenRouter gateway, Langfuse observability, prompt registry,
  Promptfoo evals.
- **Visual builder** — Puck (Craft.js as fallback).
- **Sandbox** — E2B (Daytona / Modal in later phases).
- **Realtime** — Liveblocks + Yjs.
- **Storage / email / billing** — Cloudflare R2, Resend, Stripe Billing
  + Stripe Connect.
- **Deploy / observability** — Coolify, Loki + Grafana, Sentry, PostHog,
  Better Stack.

---

## Quick local commands

```bash
# Once monorepo is scaffolded (Phase 0)
pnpm install --frozen-lockfile
pnpm dev                                     # all apps in parallel
pnpm turbo run lint typecheck test build    # what CI runs
pnpm --filter @teskel/db drizzle-kit generate
pnpm --filter @teskel/db migrate
```

Pinned tool versions (`mise` recommended):

```bash
mise use --global node@20.18.0 pnpm@9.12.3 postgresql@16.6 redis@7.4
```

---

## Roadmap (high-level)

| Phase | Window | Theme |
| --- | --- | --- |
| 0 | Week 1–4 | Foundation — monorepo, auth, RLS, deploy, CI, observability |
| 1 | Week 5–10 | Core workflow + AI — gateway, prompt registry, BullMQ runner, KB, browse-only marketplace |
| 2 | Week 11–16 | Visual builder + sandbox — Puck, E2B, project export, SDK + CLI |
| 3 | Week 17–22 | Marketplace + creator economy — Stripe Connect, listing, refund, reviews |
| 4 | Week 23–34 | Scale + enterprise — SAML/SCIM, multi-region, GameDay, SOC2 Type 1 |
| 5 | Week 35–46 | AI native UX + verticals — NL→app planner, vertical packs, SOC2 Type 2 |
| 6 | Week 47–60 | Enterprise plus — ISO 27001, HIPAA, region pinning, self-host Helm, EU AI Act |

Detail per phase + exit gates: see Plan Sec. 66 + Sec. 88–94 (build
walkthrough).

---

## Contributing

- Open an RFC at `docs/rfc/NNN-<slug>.md` to challenge a documented
  decision before changing code.
- Open a PR with the [PR template](./.github/pull_request_template.md)
  filled out. Branch naming: `<author>/<slug>` or
  `devin/<timestamp>-<slug>`.
- One concern per PR, ≤400 LOC diff.
- CI must be green; CODEOWNER review required.
- Read [`AGENTS.md` §13–14](./AGENTS.md#13-pr-workflow-branch-commit-review)
  for the full workflow and DoD.

For AI agents: see [`AGENTS.md` §20](./AGENTS.md#20-agent-specific-notes-devin--cursor--claude--copilot)
for agent-specific notes.

---

## License

Proprietary. To be finalized by maintainer before public launch.
