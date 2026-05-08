# New AI agent — context bundle

> **Status:** Living. Update whenever AGENTS.md, the skills index,
> or core conventions change.
>
> **Audience:** A fresh AI coding agent (Cursor, Claude Code,
> Copilot, Devin, or any LLM-driven contributor) starting a session
> in this repo.

This is the **minimum** context bundle to read **before** touching
any code. Treat the documents linked here as a reading checklist;
do not skip them.

If you have already read them in this session and remember them,
you may skim. If you are starting a new session, **read them in
order**.

---

## 1. The 30-second pitch

TESKEL = AI Startup Factory + Infrastructure Platform.

- Multi-tenant SaaS where **creators** compose AI workflows + UI +
  data into running apps.
- The platform itself runs the workflows, hosts the apps, and bills.
- A **marketplace** lets creators publish templates and earn revenue
  share via Stripe Connect.
- Compliance-grade ops (SOC2 / ISO / HIPAA / EU AI Act roadmap).

Stack at a glance:

| Layer | Choice |
| --- | --- |
| Web | Next.js 15 App Router (RSC), Tailwind, shadcn/ui, Puck builder |
| API | Hono + Zod + Drizzle |
| Workers | BullMQ (default), Inngest (step.ai), E2B sandbox |
| DB | Postgres 16 + pgvector + RLS |
| Auth | Better Auth (orgs, 2FA, passkey, SAML/SCIM in Phase 4) |
| AI | OpenRouter gateway + Langfuse + Promptfoo evals |
| Real-time | Liveblocks / Yjs (Phase 2+) |
| Storage | Cloudflare R2, Cloudflare CDN |
| Deploy | Coolify on Hetzner; Helm chart for self-host (Phase 6) |
| Obs | Sentry + Loki + Grafana + PostHog + Langfuse + Better Stack |
| Billing | Stripe Billing + Stripe Connect |

The **plan** is the source of truth for everything; the docs below
are the operational layer.

---

## 2. Read in order — the 60-minute bundle

Read these. In order. No shortcuts.

| # | File | What you get |
| --- | --- | --- |
| 1 | [`README.md`](../../README.md) | repo intro, current status, "Where everything lives" table |
| 2 | [`AGENTS.md`](../../AGENTS.md) | **operating manual** — hard rules, decision cheatsheet, forbidden actions |
| 3 | [`.agents/state/current-phase.md`](../../.agents/state/current-phase.md) | **what scope you are allowed in right now** |
| 4 | [`.agents/state/owners.md`](../../.agents/state/owners.md) | who owns what; routes review |
| 5 | [`.agents/glossary.md`](../../.agents/glossary.md) | every term in this repo, defined |
| 6 | [`docs/architecture/c4.md`](../architecture/c4.md) | system shape — context, container, component |
| 7 | [`docs/architecture/multi-tenancy.md`](../architecture/multi-tenancy.md) | RLS + `withTenant` — **non-negotiable** |
| 8 | [`docs/architecture/threat-model.md`](../architecture/threat-model.md) | STRIDE per surface |
| 9 | [`docs/api/conventions.md`](../api/conventions.md) | API contract |
| 10 | [`docs/api/idempotency.md`](../api/idempotency.md) | idempotency contract — applies to handlers, workers, integrations |
| 11 | [`docs/observability/slo-sli.md`](../observability/slo-sli.md) | what counts as good |
| 12 | [`AGENTS.md` §19 — Skills Library](../../AGENTS.md#19-skills-library) | the index of skills you should invoke |

Estimated read time for a careful agent: 45–60 minutes. This pays
itself back in the first PR you avoid messing up.

---

## 3. The decision cheatsheet (verbatim from AGENTS.md)

If a question is on this list, it is **already answered**. Do not
debate; obey. To change one of these decisions, write an RFC →
ADR.

| Question | Answer |
| --- | --- |
| Server or client component? | **Server** by default. `'use client'` at the leaf only. |
| Where does data fetching happen? | Server Components / Server Actions / Hono handlers. **Never** in Client Components. |
| State management? | Server state via RSC + React Query. UI-only state via Zustand / `useState`. **No Redux.** |
| Forms? | React Hook Form + Zod resolver. |
| Validation at handler? | Zod once at the boundary; types flow inward. |
| Database access? | `db.withTenant(tenantId, async tx => …)`. **Never** raw `db.*`. |
| LLM call? | Through the **AI gateway**, via the **prompt registry slot**. **Never** import a vendor SDK in product code. |
| Background work? | BullMQ (default) or Inngest (step.ai). **Never** in-process timers. |
| Email / SMS / Slack / webhooks out? | Outbox pattern → typed integration client → vendor. |
| Code-running? | E2B sandbox via the broker. **Never** run user code in-process. |
| Secret? | Read from env at boundary, validated by Zod once at boot. |
| Feature gate? | PostHog flag via `add-feature-flag` skill. |
| New table? | `add-table` skill — RLS + tests + audit comment. |
| New API route? | `add-api-route` skill — Zod + `withTenant` + idempotency + RBAC. |
| Migration? | Drizzle + expand-then-contract for hot tables. |
| Backfill? | `data-backfill-job` skill — never inline in migration. |
| Inngest vs BullMQ? | Inngest for step.run pipelines you want introspectable; BullMQ for plain workers + cron. |
| pgvector vs Qdrant? | pgvector by default (single DB); Qdrant only after an ADR documents the breaking point. |
| ADR vs RFC? | RFC for "should we?", ADR for "we did". |

---

## 4. The skills you should know (most-used)

A complete list lives in [`AGENTS.md` §19](../../AGENTS.md#19-skills-library).
The most-frequently-invoked, ranked by usage:

1. [`add-api-route`](../../.agents/skills/add-api-route/SKILL.md) — every new endpoint.
2. [`add-table`](../../.agents/skills/add-table/SKILL.md) — every new persistent class.
3. [`add-ui-component`](../../.agents/skills/add-ui-component/SKILL.md) — every new primitive.
4. [`add-page`](../../.agents/skills/add-page/SKILL.md) — every new route.
5. [`add-prompt-slot`](../../.agents/skills/add-prompt-slot/SKILL.md) — every new LLM call site.
6. [`add-feature-flag`](../../.agents/skills/add-feature-flag/SKILL.md) — every risky change.
7. [`add-package`](../../.agents/skills/add-package/SKILL.md) — every new shared utility.
8. [`add-workflow-node`](../../.agents/skills/add-workflow-node/SKILL.md) — every new node kind.
9. [`add-integration`](../../.agents/skills/add-integration/SKILL.md) — every new vendor.
10. [`add-webhook-receiver`](../../.agents/skills/add-webhook-receiver/SKILL.md) — every new inbound event.
11. [`add-cron-job`](../../.agents/skills/add-cron-job/SKILL.md) — every new schedule.
12. [`add-email-template`](../../.agents/skills/add-email-template/SKILL.md) — every new transactional mail.
13. [`add-rbac-role`](../../.agents/skills/add-rbac-role/SKILL.md) / `rotate-secret` — security ops.
14. [`release-canary`](../../.agents/skills/release-canary/SKILL.md) — every train release.
15. [`incident-sev1`](../../.agents/skills/incident-sev1/SKILL.md) / `incident-sev2` — incidents.
16. [`write-adr`](../../.agents/skills/write-adr/SKILL.md) / `write-rfc` — process.

---

## 5. Forbidden actions — repeat after me

This list duplicates [`AGENTS.md` §16](../../AGENTS.md#16-forbidden-actions).
Read it. Memorize it. The CI job will catch most violations, but not
all.

- ❌ Disable RLS on a table.
- ❌ Skip `db.withTenant`.
- ❌ Import a vendor SDK from product code (always go through
  `packages/integrations/<vendor>/`).
- ❌ Make a mutating handler without `Idempotency-Key`.
- ❌ Send an email outside the outbox.
- ❌ Run user code in-process.
- ❌ Hand-edit `apps/api/openapi.yml`.
- ❌ Add a new top-level npm dep without justification in PR.
- ❌ Skip pre-commit hooks (`--no-verify`).
- ❌ Force-push to `main`/`master`.
- ❌ Amend or rewrite history on shared branches.
- ❌ Commit secrets, even ones that look "test".
- ❌ Add a "global" / cross-tenant table without an ADR.
- ❌ Ship a release without canary, except per `release-hotfix`.
- ❌ Edit a merged ADR (write a new one that supersedes).

---

## 6. PR workflow

1. Read [`.agents/state/current-phase.md`](../../.agents/state/current-phase.md).
   If your task is out of scope for this phase, **stop** — file an
   RFC instead.
2. Branch: `devin/<unix-ts>-<slug>` (or `<initial>/<slug>` for human
   contributors).
3. Pick the **skill** that matches your task. Open it. Follow it.
4. Open the PR using
   [`.github/pull_request_template.md`](../../.github/pull_request_template.md).
   Tick every checkbox or annotate `n/a + reason`.
5. Wait for CI green. Do not merge red.
6. Address every CODEOWNER comment.
7. **Squash and merge.**

If you find yourself wanting to deviate, write an RFC. The platform
prefers slow correctness to fast surprises.

---

## 7. Sanity self-test

Before opening your first PR, answer these. If you can't, re-read.

1. The API route I'm adding — what's its idempotency story?
2. The table I'm adding — what's its retention class? Is RLS
   enabled? Did I write the cross-tenant negative test?
3. The integration I'm adding — does it need a kill-switch flag?
   What's the circuit-breaker policy?
4. The prompt I'm adding — what's the eval set? Pass-rate threshold?
5. The migration I'm adding — does it need expand-then-contract?
6. The release I'm taking out — am I in canary, or am I doing a
   hotfix? If hotfix, did I write the rollback plan first?

If any answer is "I don't know," go back to docs. The cost of
asking docs is zero. The cost of getting it wrong in production is
non-zero.

---

## 8. Communication style for AI agents

- Minimize tokens; be terse.
- Use the agent's tool calls in parallel when independent.
- When you decide something architectural, **say** what you decided
  and **why** in the PR description.
- When you skip a step in a skill, **say** which step and **why**.
- Don't promise to "test the app end-to-end" unless you actually
  do.
- When unsure between two paths, ask the human reviewer or the
  matching CODEOWNER, with both options explicit.

---

## 9. Where to leave breadcrumbs

When you discover something the docs didn't tell you, leave a trail:

- A small fact (a gotcha): drop it in [`.agents/glossary.md`](../../.agents/glossary.md)
  or in the matching `SKILL.md` under "Pitfalls".
- A bigger fact (a process change): write an ADR.
- A really big fact (architectural shift): write an RFC.

The goal: the **next** agent's session starts smarter than yours did.

---

## 10. Hard sign

You are now ready to:

- Read [`.agents/state/current-phase.md`](../../.agents/state/current-phase.md).
- Pick a story from [`docs/stories/`](../stories/) (or the user
  prompt) that fits the current phase scope.
- Find the matching skill in [`.agents/skills/`](../../.agents/skills/).
- Branch, follow the skill, open the PR.

You are **not** ready to:

- Improvise an architectural pattern.
- Skip a step in a skill because it "looks unnecessary".
- Write production code for a phase that hasn't started.
- Edit an immutable artifact (`AGENTS.md` settled stack, plan,
  merged ADRs) without going through RFC → ADR.

When in doubt, **ask** rather than **assume**. Welcome.
