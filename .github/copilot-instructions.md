# GitHub Copilot Instructions for TESKEL

GitHub Copilot Chat / Workspace: load and follow [`AGENTS.md`](../AGENTS.md)
in the repo root for every interaction in this workspace. The contents
below are a condensed mirror; `AGENTS.md` is the canonical version.

## Read first

1. `AGENTS.md` — operating manual.
2. `TESKEL_FULLSTACK_BUILD_BREAKDOWN.md` — master plan.
3. `.agents/state/current-phase.md` — current phase / allowed scope.

## Stack (settled — do not propose alternatives)

Next.js 15 (App Router + RSC), TypeScript strict, Tailwind + shadcn/ui,
Zustand + TanStack Query, Hono, Drizzle ORM, Postgres 16 + pgvector,
Postgres RLS for multi-tenancy, Better Auth, BullMQ (+ Inngest for
cron), Puck (visual builder), E2B (sandbox), Liveblocks + Yjs (realtime),
OpenRouter (AI gateway), Langfuse (AI observability), Cloudflare R2,
Resend (email), Stripe Billing + Connect, Coolify (deploy), Loki +
Grafana, Sentry, PostHog.

## Hard rules (mirror of `AGENTS.md` §8)

- Tenant isolation via Postgres RLS; tenant queries via
  `db.withTenant(orgId)`. RLS-exempt requires inline reason + reviewer
  ack.
- API boundary types via Zod; no `any` handlers.
- Default to Server Components; push `'use client'` to leaves.
- Side effects via outbox table → BullMQ worker, never inside a request
  handler.
- Idempotency by default on mutating endpoints.
- Prompts in registry only (`getPrompt(slot)`).
- Migrations reversible.
- Env access via `packages/shared/config.ts` (Zod-validated) only.

## Forbidden without explicit human approval

Force push to `main`, `--no-verify`, destructive SQL, disabling RLS,
deps >100KB without ADR, Stripe live key changes, public registry
pushes, modifying LICENSE / SECURITY, disabling CI checks, approving
own PR. See `AGENTS.md` §16.

## Coding conventions

- Files `kebab-case.ts`, components `PascalCase.tsx`, columns/tables
  `snake_case`.
- No `any`, no `// @ts-ignore` without `// REASON:` comment.
- Imports at top, group external → aliased (`@teskel/*`, `@/`) → relative.
- Default to no comments; if you comment, describe *why*.
- Tests with the change, not after; Vitest + Testcontainers + Playwright.

## When stuck

Stop and ask the human owner if a decision isn't covered by the plan,
an ADR, or `AGENTS.md`. See `AGENTS.md` §17 for the full escalation
trigger list.
