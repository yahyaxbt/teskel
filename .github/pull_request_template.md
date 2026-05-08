<!--
Thanks for sending a PR! Please fill in the sections below. Sections
marked "required" must be completed before review. PRs that touch
multiple concerns should be split — one concern per PR.

Reference docs:
- AGENTS.md §13–14 (PR workflow + DoD)
- TESKEL_FULLSTACK_BUILD_BREAKDOWN.md Sec. 70, 95, 98
-->

## Summary (required)

<!-- 1–3 sentences. What does this PR do? Why now? -->

## Linked work

- Issue / story: <!-- e.g., Closes #123 / Refs TESKEL-456 -->
- Related ADR / RFC: <!-- e.g., docs/adr/0007-rls-strategy.md -->
- Plan section affected: <!-- e.g., Sec. 22 (Multi-Tenancy & RLS) -->

## Type of change (required)

- [ ] `feat` — new feature
- [ ] `fix` — bug fix
- [ ] `chore` — maintenance, refactor, deps
- [ ] `docs` — docs / plan / ADR update
- [ ] `test` — test only
- [ ] `perf` — performance
- [ ] `build` — build / tooling
- [ ] `ci` — CI / workflow

## Phase scope check (required)

Current phase: see [`.agents/state/current-phase.md`](.agents/state/current-phase.md).

- [ ] Change is in scope for the current phase, OR
- [ ] Change is intentionally ahead of phase and gated behind a
      feature flag (default OFF). Owner approved: @____

## Architecture compliance (required)

- [ ] Tenant queries go through `db.withTenant(orgId)` (or comment
      `// RLS-EXEMPT: <reason>` with reviewer ack).
- [ ] HTTP boundary types validated by Zod.
- [ ] No `process.env.*` reads outside `packages/shared/config.ts`.
- [ ] Side effects (webhooks, emails, payments, AI calls) go via
      outbox + worker, not inline.
- [ ] Mutating endpoints accept idempotency-key.
- [ ] Prompts come from the registry (`getPrompt(slot)`).

## Database changes

- [ ] No DB schema changes, OR
- [ ] Drizzle migration generated (`drizzle-kit generate`).
- [ ] Down migration written (or backfill plan documented below).
- [ ] Tenant-isolation test added/updated (if new tenant-owned table).

<details>
<summary>Migration plan / rollback</summary>

<!-- Describe how to apply, how to roll back, data-loss risk. -->

</details>

## Tests (required)

- [ ] Unit tests added/updated (`vitest`).
- [ ] Integration tests added/updated if multi-component (`vitest +
      Testcontainers`).
- [ ] E2E test added/updated if user-facing (`Playwright`).
- [ ] `pnpm turbo run lint typecheck test` is green locally.

## Telemetry

- [ ] Events added/updated in `packages/shared/analytics/events.ts`.
- [ ] Sentry / Grafana dashboards updated (or no change needed).
- [ ] Langfuse trace tagged correctly for any new LLM calls.

## Feature flag (if user-visible)

- [ ] Not user-visible — N/A.
- [ ] Feature flag added: `flag.<scope>.<name>`. Default: OFF. Rollout
      plan documented below.

## Screenshots / recordings (if UI change)

<!-- Drag in before/after screenshots or short Loom/recording link. -->

## Rollback plan (required for non-trivial changes)

<!-- How do we revert if this fails in prod? -->

## Reviewer checklist

Reviewer should verify before approving:

- [ ] PR is one concern, ≤400 LOC of net diff (or justified).
- [ ] Architecture compliance section is honest and complete.
- [ ] Tests cover the change at the right level.
- [ ] No secrets / PII in code or logs.
- [ ] Docs / changelog updated as needed.
- [ ] Phase scope is respected.
- [ ] No forbidden actions performed (see `AGENTS.md` §16).
