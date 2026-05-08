# New engineer onboarding

> **Status:** Living. Update whenever a tool, process, or core
> architecture changes.

Welcome. This guide takes you from "git clone" to "first PR merged"
in **two weeks** without depending on a teammate's calendar.

If anything here is wrong, your first PR is fixing it.

---

## Day 0 — accounts & access (before your first day if possible)

You should already have an invite to:

| Tool | Purpose | Owner |
| --- | --- | --- |
| GitHub org `yahyaxbt` (and the `teskel` repo) | code + reviews + actions | Engineering |
| Linear (or the project tracker du jour) | stories, sprints | PM |
| Notion | high-level docs, contracts, hire packets | People |
| Slack / Discord | day-to-day comms | People |
| Coolify dashboard (read-only initially) | deploys, env vars | Platform |
| Sentry, Loki/Grafana, PostHog, Langfuse, Better Stack | observability | Platform |
| 1Password vault `Engineering` | local dev secrets, vendor accounts | Security |
| Stripe (test mode) | sandbox payments | Backend |
| Vendor consoles (OpenRouter, Resend, Cloudflare, R2, E2B, Liveblocks) | as needed | Per service owner |

If any invite is missing on Day 1, file a ticket via the people-ops
channel; do **not** ask a random teammate.

---

## Day 1 — set the stage (90–120 min)

### 1. Read the canonical docs in order

Spend the first half-day reading. **Resist the temptation** to
clone and start hacking. The fastest way to ship the wrong thing is
to skip these.

1. [`README.md`](../../README.md) — repo intro + status.
2. [`AGENTS.md`](../../AGENTS.md) — **operating manual**, hard
   rules. Re-read once a quarter; it's the contract.
3. [`TESKEL_FULLSTACK_BUILD_BREAKDOWN.md`](../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md)
   — master plan. Read the **Table of Contents** + Sec. 1–10
   (strategy) + the section for your team's current focus.
4. [`docs/architecture/c4.md`](../architecture/c4.md) — system shape.
5. [`docs/architecture/multi-tenancy.md`](../architecture/multi-tenancy.md)
   — **non-negotiable**.
6. [`docs/architecture/threat-model.md`](../architecture/threat-model.md).
7. [`docs/api/conventions.md`](../api/conventions.md) +
   [`docs/api/idempotency.md`](../api/idempotency.md) — every
   handler's contract.
8. [`docs/observability/slo-sli.md`](../observability/slo-sli.md) —
   what counts as good.
9. [`CONTRIBUTING.md`](../../CONTRIBUTING.md) — how to ship a PR.
10. [`docs/phases/README.md`](../phases/README.md) → the brief for
    the **current phase** (see
    [`.agents/state/current-phase.md`](../../.agents/state/current-phase.md)).

### 2. Mental model

By the end of Day 1, you should be able to answer (without
googling):

- What is the **North Star** of TESKEL? (Plan §1.)
- Name the **5 ICPs**. (Plan §6.)
- What is the **stack** at each layer (web, API, workers, DB)?
- What does the **outbox** do, and why?
- What does `db.withTenant` do, and why is it required?
- What is an **idempotency key** and where is one required?
- What does the **AI gateway** sit between?
- Where do **prompts** live, and why are they versioned?
- What is **RLS** and how is it applied to every table?
- What's the **release cadence**, and what triggers an auto-rollback?

If you can't, the docs are the bug — fix on Day 2.

### 3. Slack / Discord channels to join

| Channel | Purpose |
| --- | --- |
| `#engineering` | day-to-day general |
| `#deploys` | every release, every canary |
| `#incidents` | live during incidents only |
| `#observability-alerts` | Better Stack pages, throwaway alerts |
| `#sec-engineering` | security / privacy / compliance |
| `#design-system` | UI primitives + Storybook |
| `#prompts` | prompt engineering + evals |
| `#marketplace` | Phase 3+ |
| `#wins` | celebrate ship |
| `#random` | non-work |

---

## Day 2 — clone, install, run

### 1. Toolchain (pinned)

We use [`mise`](https://mise.jdx.dev) to pin local toolchain. After
cloning, `mise install` reads `.mise.toml` and installs the right
versions of:

- Node.js (LTS — see file).
- pnpm.
- Python (for some scripts and Promptfoo).
- Postgres CLI.
- Various small utilities.

If your machine already has incompatible versions, that's fine —
`mise activate` shadows them per-shell.

### 2. Clone

```bash
git clone https://github.com/yahyaxbt/teskel.git
cd teskel
mise install
pnpm install
```

(In Phase 0 W1 the bootstrap PR lands the actual scaffolding; until
then the repo is docs-only.)

### 3. Local services

```bash
docker compose -f infra/docker-compose.dev.yml up -d
```

This brings up Postgres, Redis, MinIO (R2 stand-in), MailHog (Resend
stand-in). Add seeds:

```bash
pnpm db:migrate
pnpm db:seed
```

### 4. Run the stack

```bash
pnpm dev
```

This starts the web app on `http://localhost:3000`, the API on
`http://localhost:3001`, and workers in the background. Log in as
the seeded user; explore.

### 5. Run the tests

```bash
pnpm test            # unit
pnpm test:int        # integration (touches DB)
pnpm test:e2e        # Playwright; first run downloads browsers
```

If anything fails at this point, it is **not** your bug — it is the
docs failing to anticipate your environment. File an issue.

---

## Day 3–5 — first PR

### 1. Pick a starter task

From your team lead, or from the `good-first-issue` label in Linear.
A starter PR should:

- Touch ≤ 3 files.
- Land in ≤ 1 day of work.
- Use one of the existing **skills** end-to-end (see
  [`.agents/skills/`](../../.agents/skills/)). Examples:
  - `add-package` — scaffold a tiny shared utility.
  - `add-table` — add a small table you've actually been asked for.
  - `add-feature-flag` — gate a UI tweak behind a flag.

### 2. Branch + skill

```bash
git checkout -b user/<initial>-<slug>
```

Open the matching skill's `SKILL.md` and follow it **step-by-step**.
Skills are deliberately prescriptive. Do not improvise on the first
PR.

### 3. PR template

Open the PR using the [`pull_request_template.md`](../../.github/pull_request_template.md).
Fill **every** checkbox. If a box doesn't apply, mark it `n/a` with
a one-line reason.

### 4. Get a review

CODEOWNERS routes the request automatically. Address every comment;
mark them resolved when done.

### 5. Land it

When CI is green and you have an approval, merge using **squash and
merge** (the default). Watch the Coolify deploy to staging.

---

## Week 1 — extend your circle

By end of Week 1, you should have:

- 1 merged PR.
- Read the **Phase brief** that's currently active.
- Run through `incident-sev2` skill in your head with a teammate
  (a *fire drill*, not a real incident).
- Subscribed to the dashboards in
  [`docs/observability/dashboards.md`](../observability/dashboards.md)
  for your team's surface.
- One ADR or RFC review under your belt (it's fine to read and
  comment "+1" — the muscle is the read).

---

## Week 2 — solo

By end of Week 2, you should:

- Have authored or co-authored 3–5 PRs.
- Be on a rotation if your team has one (release captain, on-call
  shadow, design review).
- Have shipped one **doc fix** in this directory or in
  [`docs/`](../). New eyes find the most stale lines.

If you don't feel solo by end of Week 2, that's a signal to escalate
to your manager — not a sign that you're slow. The onboarding loop
is supposed to be tight.

---

## What to do when stuck

In order:

1. **Search the repo** (`rg`, `grep`, GitHub UI). The chance the
   answer is in `docs/` or a `SKILL.md` is high.
2. **Read the matching skill.** If your task fits one of the 30+
   skills in `.agents/skills/`, follow it.
3. **Search Slack history.** We probably had this conversation in
   `#engineering` before.
4. **Ask in `#engineering`** with context (what you tried,
   error message, what you expect).
5. **Ping your buddy / lead.** Direct DM is fine for blocking.
6. **Page the on-call** only if production is actually broken. See
   [`incident-sev1`](../../.agents/skills/incident-sev1/SKILL.md) /
   [`incident-sev2`](../../.agents/skills/incident-sev2/SKILL.md).

---

## What to do when *you* see something wrong in docs

Open a PR. Even a one-line fix. It gets merged within the day in 90%
of cases. Doc PRs do not need a story.

---

## Things that will trip you up

| Trap | Fix |
| --- | --- |
| Calling `db.select` outside `withTenant`. | Read [`multi-tenancy.md`](../architecture/multi-tenancy.md). |
| Importing a vendor SDK directly in product code. | Read [`add-integration` SKILL](../../.agents/skills/add-integration/SKILL.md). |
| Skipping the `Idempotency-Key` on a mutating handler. | Read [`idempotency.md`](../api/idempotency.md). |
| Logging an email or raw user content. | The lint rule will flag it; if not, refactor anyway. |
| Hand-editing `apps/api/openapi.yml`. | Generated; edit the Zod schema. |
| Passing a closure-captured `tenantId` into a worker. | Workers read from `job.data` every time. |
| Treating "Free plan" specially in product code. | Read entitlement registry; never special-case in handlers. |
| Adding a new top-level npm dep without review. | PR + reviewer comment "why this dep, why this version". |

---

## Calendar (optional)

Block these on Day 1:

- Daily standup (team).
- Weekly PR review hour (your team).
- Bi-weekly all-hands.
- Monthly architecture review.
- Quarterly retro.
- On-call shadow (if joining backend / platform).

---

## Final word

You are expected to **break things** in your first month. The
guardrails (RLS, idempotency, error budgets, canary, runbooks) exist
so that an honest mistake never reaches a customer. Optimize for
shipping small, reading docs, and asking questions early.

Welcome to TESKEL.
