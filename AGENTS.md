# AGENTS.md — TESKEL AI Agent Operating Manual

> Read this file **first** before making any change. This file is the
> contract between TESKEL maintainers and any AI coding agent (Devin,
> Cursor, Claude Code, GitHub Copilot, Continue.dev, Aider, custom agent)
> that operates on this repository.
>
> The companion master plan is
> [`TESKEL_FULLSTACK_BUILD_BREAKDOWN.md`](./TESKEL_FULLSTACK_BUILD_BREAKDOWN.md).
> If anything in this file conflicts with the plan, **the plan wins** —
> raise a PR amending the plan first if you disagree.

---

## Table of Contents

1. [TL;DR Quick Start](#1-tldr-quick-start)
2. [Project Identity](#2-project-identity)
3. [Source of Truth & Documents Hierarchy](#3-source-of-truth--documents-hierarchy)
4. [Repository Map (Target Structure)](#4-repository-map-target-structure)
5. [Stack & Library Decisions (Settled)](#5-stack--library-decisions-settled)
6. [Tooling & Local Commands](#6-tooling--local-commands)
7. [Coding Conventions](#7-coding-conventions)
8. [Architecture Hard Constraints](#8-architecture-hard-constraints)
9. [Security & Data Rules](#9-security--data-rules)
10. [AI / LLM Rules](#10-ai--llm-rules)
11. [Decision Cheatsheet (Mirror of Plan Sec. 99)](#11-decision-cheatsheet-mirror-of-plan-sec-99)
12. [Phase Awareness & Scope Control](#12-phase-awareness--scope-control)
13. [PR Workflow (Branch, Commit, Review)](#13-pr-workflow-branch-commit-review)
14. [Definition of Ready / Done](#14-definition-of-ready--done)
15. [Working with the Plan (RFC → ADR)](#15-working-with-the-plan-rfc--adr)
16. [Forbidden Actions Without Human Approval](#16-forbidden-actions-without-human-approval)
17. [Escalation Triggers](#17-escalation-triggers)
18. [Where to Find Things (Quick Index)](#18-where-to-find-things-quick-index)
19. [Skills Library](#19-skills-library)
20. [Agent-Specific Notes (Devin / Cursor / Claude / Copilot)](#20-agent-specific-notes-devin--cursor--claude--copilot)
21. [Versioning & Updating This File](#21-versioning--updating-this-file)

---

## 1. TL;DR Quick Start

```text
1. READ:  TESKEL_FULLSTACK_BUILD_BREAKDOWN.md        ← master plan (immutable, edit via ADR)
          AGENTS.md (this file)                      ← rules of engagement
          .agents/state/current-phase.md             ← what scope is allowed RIGHT NOW
2. SCOPE: Stay inside the current phase. No Phase 4 work in Phase 0.
3. STACK: Decided. See §5. Don't re-debate. To overturn → write ADR (§15).
4. BEFORE COMMIT: pnpm install && pnpm turbo run lint typecheck test
5. BRANCH: devin/<timestamp>-<slug>  (or your own author handle)
6. PR: one concern, ≤400 LOC, fill the template, get 1+ review.
7. FORBIDDEN without explicit approval: see §16.
8. STUCK / UNCLEAR: stop and ask a human (§17).
```

If you can only remember three things:

- **Plan is law.** RLS, queue, auth, billing patterns are fixed.
- **Tenant isolation is non-negotiable.** Every tenant query goes
  through `withTenant()`. RLS-exempt code requires explicit reviewer ack.
- **Side effects via outbox.** No external API calls inside a request
  handler — emit an event, let a worker handle it.

---

## 2. Project Identity

- **Name**: TESKEL.
- **What it is**: AI Startup Factory + Infrastructure Platform — an
  operating system for building, launching, monetizing, and scaling
  AI-native businesses end-to-end.
- **What it is NOT**: a website builder, an AI wrapper, a generic
  template marketplace, or an n8n / Vercel clone.
- **Default tenant region**: Singapore (Indonesia / SEA primary;
  EU / US optional via region pinning, Phase 6).
- **Primary languages**:
  - Code & ADR: English.
  - User-facing UI: Indonesian + English (i18n; see Plan Sec. 15).
  - Internal docs: free-form (Indonesian or English).
- **Tone (user-facing)**: pragmatic, builder-first, Indonesian
  founder-friendly. Avoid hype.

---

## 3. Source of Truth & Documents Hierarchy

When sources disagree, **higher in this list wins**:

1. Live ADRs in `docs/adr/` (latest accepted ADRs override older plan
   text).
2. `TESKEL_FULLSTACK_BUILD_BREAKDOWN.md` (master plan).
3. This file (`AGENTS.md`).
4. `README.md` and other top-level READMEs.
5. Inline code comments / JSDoc / TSDoc.
6. Skill files in `.agents/skills/`.
7. Issue / PR descriptions.

If you find drift between (1)–(3), **fix it in the same PR** by adding a
docs commit. Do not let drift accumulate.

---

## 4. Repository Map (Target Structure)

```text
.
├── apps/
│   ├── web/              # Next.js 15 (App Router + RSC) — primary user UI
│   ├── api/              # Hono-based public API (Phase 1+)
│   └── docs/             # Mintlify or Docusaurus docs site
├── packages/
│   ├── auth/             # Better Auth wrapper + plugins
│   ├── db/               # Drizzle schema, RLS helpers, client
│   ├── ai/               # OpenRouter gateway + prompt registry + Langfuse
│   ├── queue/            # BullMQ queues + workers
│   ├── runner/           # Workflow runtime (executes node graphs)
│   ├── ui/               # shadcn/ui design system primitives + tokens
│   ├── shared/           # Cross-cutting types, Zod schemas, analytics events
│   ├── sdk/              # Public TypeScript SDK
│   ├── cli/              # `teskel` CLI (commander-based)
│   ├── template-engine/  # Manifest parser + installer + signing
│   ├── billing/          # Stripe + Stripe Connect adapters (Phase 3)
│   └── testing/          # Test utilities, fixtures
├── seeds/templates/      # Source of seed marketplace templates
├── infra/                # Coolify configs, Dockerfiles, Helm charts
├── docs/                 # Internal docs
│   ├── adr/              # Architecture Decision Records (numbered)
│   ├── runbooks/         # On-call runbooks
│   └── rfc/              # Pre-decision proposals
├── .agents/              # AI agent operational state
│   ├── state/            # current-phase.md, owners.md
│   └── skills/           # Reusable agent procedures
├── .github/              # PR template, CI workflows, CODEOWNERS, copilot rules
├── AGENTS.md             # This file
├── CLAUDE.md             # Pointer to AGENTS.md (for Claude Code)
├── .cursorrules          # Pointer to AGENTS.md (for Cursor)
├── README.md             # Public-facing project intro
└── TESKEL_FULLSTACK_BUILD_BREAKDOWN.md   # Master plan
```

If a directory does not exist yet, you may create it — but follow the
structure exactly. Do **not** invent parallel directories
(`server/`, `frontend/`, `lib/`, `src/`).

---

## 5. Stack & Library Decisions (Settled)

These are decided. Don't propose alternatives in code; if you must
challenge one, write an ADR (template at `docs/adr/0000-template.md`).

| Layer | Choice | Plan Ref | ADR |
| --- | --- | --- | --- |
| Frontend framework | Next.js 15 (App Router + RSC) | Sec. 18 | ADR-0009 |
| Language | TypeScript strict mode | Sec. 17 | — |
| UI primitives | shadcn/ui + Tailwind CSS + Framer Motion | Sec. 10, 18 | — |
| State (client) | Zustand (UI state) + TanStack Query (server state) | Sec. 18 | — |
| Forms / validation | React Hook Form + Zod | Sec. 18 | — |
| Backend framework | Hono | Sec. 19 | — |
| ORM | Drizzle (Prisma fallback only via ADR) | Sec. 21 | ADR-0002 |
| Database | PostgreSQL 16 + pgvector | Sec. 21 | — |
| Multi-tenancy | Postgres RLS via `current_setting('app.current_org')` | Sec. 22 | ADR-0007 |
| Auth | Better Auth (org + 2FA + passkey + magic-link plugins) | Sec. 46 | ADR-0001 |
| Queue / jobs | BullMQ (Inngest as secondary for cron / step.ai) | Sec. 28 | ADR-0003 |
| Visual builder | Puck (Craft.js fallback only via ADR) | Sec. 25 | ADR-0005 |
| Sandbox / code exec | E2B (Daytona for long-lived dev envs P1; Modal for GPU P1) | Sec. 32 | ADR-0008 |
| Real-time | Liveblocks managed (Yjs raw self-host as fallback) | Sec. 26 | ADR-0004 |
| AI gateway | OpenRouter (multi-model proxy) | Sec. 23 | — |
| AI observability | Langfuse | Sec. 23 | ADR-0006 |
| Vector store | pgvector default; Qdrant per Enterprise opt-in | Sec. 21, 30 | — |
| Storage | Cloudflare R2 | Sec. 27 | — |
| Email transactional | Resend | Sec. 31 | — |
| Billing / subscriptions | Stripe Billing | Sec. 60 | — |
| Marketplace payouts | Stripe Connect Standard | Sec. 57 | — |
| Deploy / orchestration | Coolify (self-managed Docker) | Sec. 36 | ADR-0010 |
| Logs + metrics | Loki + Grafana (Cloud or self-host) | Sec. 39 | — |
| Tracing | OpenTelemetry → Grafana Tempo | Sec. 39 | — |
| Errors | Sentry | Sec. 39 | — |
| Product analytics | PostHog | Sec. 63 | — |
| Status page | Better Stack or Statuspage.io | Sec. 41, 65 | — |
| Secret manager (team) | 1Password or Doppler | Sec. 38 | — |
| Issue tracker | Linear or GitHub Projects | Sec. 87 | — |

If a library does **not** appear in this table and the plan, treat
adding it as requiring an ADR (or at minimum, a paragraph in the PR
description explaining why and confirming bundle/license fit).

---

## 6. Tooling & Local Commands

```bash
# Install deps (use frozen lockfile)
pnpm install --frozen-lockfile

# Dev — all apps in parallel
pnpm dev

# Dev — one app
pnpm --filter web dev
pnpm --filter api dev

# Lint, typecheck, test (run before every commit)
pnpm turbo run lint typecheck test

# Build everything
pnpm turbo run build

# Database
pnpm --filter @teskel/db drizzle-kit generate     # generate migration from schema diff
pnpm --filter @teskel/db drizzle-kit push         # dev only — push schema directly
pnpm --filter @teskel/db migrate                  # apply pending migrations to env

# Workers
pnpm --filter @teskel/queue worker:dev            # local BullMQ workers

# CLI
pnpm --filter @teskel/cli build && node packages/cli/dist/index.js <cmd>

# Pre-commit (after `pre-commit install`)
pre-commit run --all-files
```

Pinned tool versions (see Plan Sec. 17 / 87.2):

```bash
mise use --global node@20.18.0
mise use --global pnpm@9.12.3
mise use --global postgresql@16.6
mise use --global redis@7.4
```

CI runs the same `pnpm turbo run lint typecheck test build` and
must be green before merge.

---

## 7. Coding Conventions

### TypeScript

- `strict: true`. No `any`. No `// @ts-ignore` without an inline
  `// REASON:` explanation.
- Prefer narrow types over `unknown`. If you reach for `getattr/setattr`-
  style dynamic access, you don't understand the type yet — go look.
- Use **Zod** for runtime schemas at every boundary (HTTP request,
  webhook, env). Derive TypeScript types via `z.infer<typeof X>`.
- Prefer composition over inheritance. Classes only for stateful
  services (e.g., `WorkflowExecutor`).

### Naming

- Files: `kebab-case.ts` (`use-session.ts`, `org-switcher.tsx`).
- React components: `PascalCase.tsx` and exported as default + named
  (`export function OrgSwitcher`).
- Functions / variables: `camelCase`.
- Types / interfaces / components: `PascalCase`.
- Constants: `SCREAMING_SNAKE_CASE` only for true constants
  (env values, hardcoded limits). Prefer enums or `as const` literals
  for closed sets.
- Drizzle: `snake_case` columns and tables (`created_at`, `org_id`,
  `workflow_runs`).

### Imports

- Always at the top of the file. No nested imports inside functions or
  classes (except for circular-dep workarounds with a `// REASON:`).
- Use absolute imports via tsconfig aliases (`@/`, `@teskel/*`), never
  `../../../`.
- Group order: external deps → internal aliased (`@teskel/*`, `@/`) →
  relative (`./`). Blank line between groups.

### Comments

- **Default is no comment.** Bias aggressively toward terseness.
- No "what's changing in this PR" comments — that's the PR description.
- If you must comment, describe the *why*, not the *what*.
- `TODO(handle):` / `FIXME(handle):` must include an owner and an issue
  link.

### Errors

- Throw typed `Error` subclasses (`AuthError`, `RateLimitError`,
  `WorkflowAbortError`). Never throw strings or plain objects.
- All external API calls must be wrapped with try/catch + idempotent
  retry policy.
- Log with structured fields: `logger.error({ err, orgId, traceId },
  'workflow.run.failed')`. Don't `JSON.stringify` an error.
- Surface user-facing errors via a unified problem+json shape (RFC 9457).

### Tests

- **Unit**: Vitest. Co-locate `<name>.test.ts` next to source.
- **Integration**: Vitest + Testcontainers (Postgres + Redis). Live in
  `packages/<x>/test/integration/`.
- **E2E**: Playwright in `apps/web/e2e/`.
- Tests are written **with** the change, not after. PRs with new
  features must include tests.
- Don't modify a test to make it pass unless the PR explicitly states a
  test fix is intentional.

### Logs / telemetry

- Every user-visible action emits a PostHog event named `noun.verb`
  (`project.created`, `workflow.run.succeeded`). Schemas live in
  `packages/shared/analytics/events.ts`.
- Add a Grafana dashboard or panel link in the PR if the metric is new.

---

## 8. Architecture Hard Constraints

These are non-negotiable. Violating any without a reviewer-acked
exception **blocks merge**.

1. **Tenant isolation via RLS.** Every query that touches tenant-owned
   data goes through `db.withTenant(orgId)`. Bypassing RLS requires a
   comment `// RLS-EXEMPT: <reason>` plus explicit reviewer approval.
2. **API boundary types.** Public HTTP handlers MUST validate input via
   Zod and return typed responses. No `any` in route signatures.
3. **Server vs Client components.** Default to Server Components. Mark
   `'use client'` only at leaves that genuinely need state, event
   handlers, or browser-only APIs.
4. **Side effects via outbox.** Anything that calls an external system
   (webhook, email, payment, AI) must be enqueued via the outbox table
   → BullMQ worker. Never inside a request handler. (Plan Sec. 19.7)
5. **Idempotency by default.** All mutating endpoints accept an
   `Idempotency-Key` header (or use a deterministic key derived from
   inputs) and dedupe via Redis or `processed_events`.
6. **Prompts in registry.** No inline prompt strings in feature code.
   Use `getPrompt(slot)` from `@teskel/ai`. (Plan Sec. 23)
7. **Migrations are reversible.** Drizzle migrations include both `up`
   and `down`, or a documented backfill plan in the PR.
8. **No env access outside config.** All `process.env.*` reads happen
   inside `packages/shared/config.ts` (Zod-validated) and are exported
   as typed singletons.
9. **No data destruction without paperwork.** User data deletions go
   through `softDelete()` first. Hard-delete only via scheduled job
   with audit log entry.
10. **Single-writer DB.** No two services write the same Postgres row.
    Cross-service consistency goes through events + outbox.

---

## 9. Security & Data Rules

- Never log PII (email, phone, full name, address) in service logs. Hash
  or redact before emitting.
- Never log secrets (API keys, OAuth tokens, JWTs, Stripe keys). The
  logger must scrub fields named `password`, `token`, `secret`, `key`,
  `authorization`.
- Crypto: use Web Crypto / Node `crypto` only. No homegrown crypto.
- Webhooks: verify HMAC signature **before** parsing the body. Replay
  protection via `processed_events`.
- API keys: hashed at rest (argon2id), prefix-checked, scoped, and
  rate-limited per key.
- Storage: signed URLs, expiry ≤15 minutes for downloads, ≤5 minutes
  for uploads.
- Sandbox: deny-by-default egress; allowlist domains per template.
- Cookie domain must include the leading dot for cross-subdomain
  sessions: `cookieDomain: '.teskel.app'`, `secureCookies: true`,
  `sameSite: 'lax'`.
- See Plan Sec. 45–55 for the full threat model and compliance
  roadmap.

---

## 10. AI / LLM Rules

- All LLM calls go through `@teskel/ai`'s gateway (OpenRouter). Direct
  `fetch('https://api.openai.com/...')` is **forbidden**.
- Every call must:
  - Be associated with an `org_id` and a `trace_id`.
  - Emit a Langfuse trace.
  - Increment a Stripe meter event for token spend (Plan Sec. 60).
  - Respect per-org daily and per-tenant rate limits (Plan Sec. 23).
- Prompts live in the registry. Versioned via `prompts(version)` table.
  Treat prompts as code — review every change.
- Promptfoo eval suite runs on every prompt change (CI required).
- AI-generated content rendered to users must pass through the safety
  filter (Plan Sec. 52). Never directly render raw LLM HTML.
- Tools and function-calls must be allowlisted per workflow node;
  default policy is deny-all.
- Hallucination guard: NL→app planner output must be **confirmed by
  the user** before generation (Plan Sec. 93.1, 97).
- Cost guardrails: per-org daily cap enforced at the gateway. Alert at
  70%. Hard stop at 100% with friendly UX (Plan Sec. 44, 89).

---

## 11. Decision Cheatsheet (Mirror of Plan Sec. 99)

When in doubt about a design choice that's not covered elsewhere, use
this table. **Do not re-debate** these.

| Question | Default Answer | Note |
| --- | --- | --- |
| Server Component or Client? | Server, unless interactive state needed | Push `'use client'` to leaves |
| Fetch on server or TanStack Query? | Initial render: server. Real-time / refetch: TQ | Avoid double-fetch |
| Raw SQL or Drizzle query builder? | Drizzle. Raw SQL only for analytics | Always parameterize |
| Inngest or BullMQ? | Cron / scheduled / `step.ai` → Inngest. Real-time short jobs, fan-out → BullMQ | Plan Sec. 28 |
| Better Auth: built-in plugin or custom? | Built-in first; custom only if real gap | ADR required for custom |
| DB write directly or via outbox? | Outbox if it triggers external side effect | Plan Sec. 19.7 |
| Liveblocks managed or Yjs raw? | Liveblocks default | Yjs only for big docs / self-host |
| Sandbox provider? | E2B default. Daytona for long-lived envs. Modal for GPU | Plan Sec. 32 |
| pgvector or Qdrant? | pgvector default. Qdrant per-Enterprise opt-in | >50M chunks → Qdrant |
| Custom UI component or shadcn/ui? | shadcn/ui; extend via composition | Avoid forks |
| Manual migration or `drizzle-kit generate`? | `generate` always; manual only for special indexes | Test up + down |
| Cache in Redis or React Query only? | TanStack on client; Redis for cross-instance | Avoid double-invalidation |
| New runtime dep — when allowed? | Replaces ≥50 LOC of ours **and** MIT/Apache | ADR if >100KB bundle |
| Self-host or managed cloud? | Cloud until $X/month, then evaluate | Avoid running 5 different vendors |
| ADR or RFC? | ADR for decisions with year+ consequences; RFC for explorations | See `docs/adr/0000-template.md` |

---

## 12. Phase Awareness & Scope Control

The current phase is recorded in
[`.agents/state/current-phase.md`](./.agents/state/current-phase.md).
Read it **first**.

Allowed scope per phase (summary; full detail in Plan Sec. 66):

| Phase | In Scope | Out of Scope |
| --- | --- | --- |
| 0 — Foundation | monorepo, DB, Drizzle, Better Auth, RLS, Coolify deploy, CI, Sentry/OTel/PostHog, ADR-0001/2/3/9/10 | workflow runtime, marketplace, billing, sandbox, AI gateway logic |
| 1 — Core Workflow + AI | AI gateway, prompt registry, React Flow studio, BullMQ runner, KB pgvector, marketplace browse, 3 seed templates | visual builder, sandbox, paid marketplace, SAML |
| 2 — Visual Builder + Sandbox | Puck, blocks, E2B sandbox, project export, SDK + CLI v0, AI assistant intra-app | Stripe Connect, refund flow, vertical packs |
| 3 — Marketplace + Creator | Stripe Connect, listing wizard, moderation, buy/install, refund, reviews, Verified badge, programmatic SEO | SAML, multi-region, SOC2 |
| 4 — Scale + Enterprise | SAML/SCIM, audit log + SIEM, multi-region replica, GameDay, SOC2 Type 1 | NL→app planner, vertical packs |
| 5 — AI Native + Verticals | NL→app, inline copilot, 5 vertical packs, SOC2 Type 2 | ISO 27001, HIPAA |
| 6 — Enterprise Plus | ISO 27001, HIPAA + BAA, region pinning, self-host Helm, EU AI Act mapping | (open) |

If you are asked to implement Phase 3 features while the repo is in
Phase 0: **stop and surface to a human owner before writing code**.
Suggest creating a feature flag stub if the work is genuinely required
ahead of time.

---

## 13. PR Workflow (Branch, Commit, Review)

### Branching

- Format: `<author>/<short-slug>` or `devin/<timestamp>-<slug>`.
- Off `main`. Trunk-based; short-lived (≤1 week).
- One concern per branch. Aim for ≤400 LOC diff.

### Commits

- Conventional Commits: `feat:`, `fix:`, `chore:`, `refactor:`, `docs:`,
  `test:`, `perf:`, `build:`, `ci:`.
- Imperative present tense ("Add ...", not "Added ...").
- Body explains *why*, not *what*. Wrap at 80 cols.

### PRs

- Use the PR template at `.github/pull_request_template.md`. Fill in
  every section.
- Link the issue / story (`Closes #123` or `Refs TESKEL-456`).
- Screenshots / recordings for any UI change.
- Telemetry note (events / dashboards added).
- Migration plan + rollback plan if DB schema or env config changes.
- Reviewer: at least 1 from CODEOWNERS for each touched area.
- CI must be green; do not bypass.

### Merging

- Squash merge by default. Merge commits only for release branches.
- Delete the branch after merge.
- If the merge touches deployment-relevant files, watch the auto-deploy
  for ≥10 minutes before walking away.

---

## 14. Definition of Ready / Done

### Definition of Ready (before you start coding)

- [ ] User story has acceptance criteria.
- [ ] UX wireframe / spec exists if user-facing.
- [ ] Dependencies cleared (other PRs merged).
- [ ] Confidence estimate ≤3 days of work.
- [ ] Telemetry plan written.

### Definition of Done (before you mark the PR ready)

- [ ] Code merged-ready (`main` rebased, no conflicts).
- [ ] Lint + typecheck + unit + integration tests green.
- [ ] E2E happy-path covered if user-facing.
- [ ] Telemetry added (event + dashboard panel).
- [ ] Docs / CHANGELOG updated.
- [ ] Feature flag added (default OFF) if user-visible and risky.
- [ ] Rollback plan in PR description.
- [ ] CODEOWNER reviewed.
- [ ] Manual smoke test on preview env (link in PR).

---

## 15. Working with the Plan (RFC → ADR)

The master plan is **immutable except via ADR-driven amendments**.

To change a documented decision:

1. Open an RFC at `docs/rfc/NNN-<short-slug>.md` (template in
   `docs/rfc/0000-template.md`). Describe context, options, and
   recommended decision.
2. Get +1 from at least 2 senior engineers (`CODEOWNERS`).
3. Promote to ADR at `docs/adr/NNN-<short-slug>.md` (template in
   `docs/adr/0000-template.md`).
4. In the same PR that lands the ADR, **update the plan section** to
   reflect the new decision, with a footnote linking the ADR.

Editorial fixes (typos, broken anchors, link rot) can go in directly —
no RFC needed.

---

## 16. Forbidden Actions Without Human Approval

Never do any of these without explicit written approval from a human
maintainer (`@yahyaxbt` or another `OWNERS`-listed maintainer):

- Force push to `main` or `release/*`.
- Skip pre-commit hooks (`--no-verify`).
- Modify environment variables in production.
- Run destructive DB commands (`DROP TABLE`, `TRUNCATE`, `DELETE`
  without `WHERE`).
- Disable RLS on any tenant table.
- Add a runtime dependency that increases bundle size by >100KB without
  an ADR.
- Send transactional emails from `*.teskel.app` at >10k volume.
- Change Stripe live keys, webhook secrets, or rotate signing keys.
- Push container images to a public registry.
- Modify `LICENSE`, `SECURITY.md`, or `CODE_OF_CONDUCT.md`.
- Disable any required CI check.
- Bypass branch protection.
- Approve your own PR.

If unsure → **ask first**.

---

## 17. Escalation Triggers

Stop and ask the human owner if **any** of these is true:

1. A required decision is **not** documented in the plan, an ADR, or
   this file.
2. The required change conflicts with the plan or an ADR.
3. A test or CI fails in a non-obvious way (after 1 honest fix attempt).
4. A library version bump breaks API contracts.
5. A change touches >5 packages or >500 LOC of net diff.
6. A change touches `packages/auth/`, `packages/billing/`, or
   `packages/template-engine/` (signing).
7. A change introduces a new external service or vendor.
8. A migration is non-reversible (no `down` and no documented backfill).
9. A change disables, weakens, or works around any §8 hard constraint.
10. Cost / performance impact is unclear and could be material
    (>10% infra cost or >100ms p95 latency).

When escalating: state the question, list the options you considered,
recommend one, and tag the owner.

---

## 18. Where to Find Things (Quick Index)

| Need | Look at |
| --- | --- |
| Architectural overview / C4 | Plan Sec. 16 |
| Monorepo / build tooling | Plan Sec. 17 |
| Frontend architecture | Plan Sec. 18 |
| Backend architecture | Plan Sec. 19 |
| API design conventions | Plan Sec. 20 + Appendix B |
| DB schema patterns + DDL | Plan Sec. 21 + Appendix A |
| RLS policies | Plan Sec. 22 + Appendix A |
| AI infra + prompt registry | Plan Sec. 23 |
| Workflow runtime semantics | Plan Sec. 24 + Appendix D |
| Visual builder | Plan Sec. 25 |
| Real-time / collab | Plan Sec. 26 |
| Storage | Plan Sec. 27 |
| Queues / jobs | Plan Sec. 28 |
| Sandbox | Plan Sec. 32 |
| SDK + CLI | Plan Sec. 33–34 |
| Environments / deploy | Plan Sec. 35–36 |
| CI/CD | Plan Sec. 37 |
| Secrets / config | Plan Sec. 38 |
| Observability | Plan Sec. 39 |
| SLI / SLO / SLA | Plan Sec. 40 |
| Incident response / runbooks | Plan Sec. 41 + `docs/runbooks/` |
| Backup / DR | Plan Sec. 42 |
| Performance budgets | Plan Sec. 43 |
| FinOps | Plan Sec. 44 |
| Threat model / STRIDE | Plan Sec. 45 |
| AuthN / AuthZ | Plan Sec. 46 |
| RBAC matrix | Plan Sec. 47 |
| API keys | Plan Sec. 48 |
| Secrets vault / encryption | Plan Sec. 49 |
| Webhook security | Plan Sec. 50 |
| Code sandbox security | Plan Sec. 51 |
| AI guardrails | Plan Sec. 52 |
| Compliance roadmap | Plan Sec. 53 |
| Marketplace architecture | Plan Sec. 56 |
| Creator economy | Plan Sec. 57 |
| Template manifest spec | Plan Sec. 58 + Appendix C |
| Marketplace moderation | Plan Sec. 59 |
| Pricing / billing impl | Plan Sec. 60 |
| Onboarding | Plan Sec. 61 |
| SEO | Plan Sec. 62 |
| Analytics events | Plan Sec. 63 + `packages/shared/analytics/events.ts` |
| A/B testing | Plan Sec. 64 |
| Roadmap (Phase 0–6) | Plan Sec. 66 |
| Detailed epics / stories | Plan Sec. 68 |
| Build walkthrough per phase | Plan Sec. 88–94 |
| Final cutover playbook | Plan Sec. 95 |
| Day-2 ops cadence | Plan Sec. 96 |
| Common pitfalls per phase | Plan Sec. 97 |
| Promotion checklist per story | Plan Sec. 98 |
| Quick decisions | Plan Sec. 99 + §11 here |
| Env vars reference | Plan Appendix E |
| Pre-launch checklist | Plan Appendix F |
| ADR / RFC template | Plan Appendix G + `docs/adr/0000-template.md` |

---

## 19. Skills Library

Reusable agent procedures live in `.agents/skills/<name>/SKILL.md`. Each
skill is a checklist for a recurring operation. **Use them.** Don't
reinvent the wheel.

Currently available skills:

- [`add-package`](./.agents/skills/add-package/SKILL.md) — scaffold a
  new internal `packages/*` package.
- [`add-table`](./.agents/skills/add-table/SKILL.md) — add a Drizzle
  table with RLS policies and migration.
- [`add-workflow-node`](./.agents/skills/add-workflow-node/SKILL.md) —
  register a new workflow node type end-to-end.
- [`release-canary`](./.agents/skills/release-canary/SKILL.md) — promote
  a canary deploy to 100%.
- [`incident-sev1`](./.agents/skills/incident-sev1/SKILL.md) — Sev1
  incident response runbook.

For **Devin**: invoke via the `skill` tool. For other agents: read the
`SKILL.md` and follow the steps.

---

## 20. Agent-Specific Notes (Devin / Cursor / Claude / Copilot)

### Devin

- Uses `AGENTS.md` natively. Also reads `.agents/skills/*/SKILL.md`.
- Branch naming `devin/<timestamp>-<slug>` is canonical.
- Use `git_create_pr` (not `gh` CLI). See `Pull Request Workflow` system
  note.

### Cursor

- Reads `.cursorrules` and / or `.cursor/rules/*.mdc`. Both files in this
  repo are pointers to `AGENTS.md` — keep them in sync if you change
  this file.
- Indexing: trust the workspace index for file locations; verify against
  §4 if uncertain.

### Claude Code

- Reads `CLAUDE.md` (pointer to this file). MCP tools may also be
  configured per-project.
- For long edits, prefer `MultiEdit` over multiple `Edit` calls.

### GitHub Copilot Chat / Copilot Workspace

- Reads `.github/copilot-instructions.md` (pointer to this file).
- Inline suggestions follow the `.editorconfig` and `.prettierrc`
  settings — keep them aligned with this file.

### Aider / Continue.dev / custom agents

- No standardized config file. Operator should pass `AGENTS.md` as a
  system message or include it in the context.

---

## 21. Versioning & Updating This File

This file follows semver-ish numbering at the top of the document
(below). Bump:

- **Major** when removing or inverting a rule (e.g., changing default
  ORM).
- **Minor** when adding a rule or section.
- **Patch** for editorial fixes.

Always link the corresponding plan section / ADR when adding or
changing a rule.

```text
Version: 1.0.0
Last reviewed: 2026-05-08
Owners: see CODEOWNERS (root)
```
