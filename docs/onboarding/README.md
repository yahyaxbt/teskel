# Onboarding

> **Status:** Living. Updated whenever the stack, tools, or
> processes change.

This directory exists so that a **brand new contributor** — human or
AI — can become productive in TESKEL without pinging a teammate.

There are two audiences:

| Document | Audience |
| --- | --- |
| [`new-engineer.md`](./new-engineer.md) | A new human engineer joining the team — Day 1, Week 1, Week 2. |
| [`new-agent.md`](./new-agent.md) | A new AI coding agent (Cursor, Claude Code, Copilot, Devin) starting a session in this repo. The minimum context bundle to read before touching code. |

If you're an existing contributor, you don't need to read these
docs day-to-day — but you **do** need to keep them current. When you
change tooling, processes, or core architecture, update the relevant
section in the **same PR** that lands the change.

---

## What new contributors should NOT have to figure out alone

- Where to find the master plan.
- How the repo is organized.
- How to run things locally.
- What the hard rules are.
- What "done" looks like for a PR.
- Who reviews what.
- How to escalate.

If a new contributor is asking any of these, the docs failed and a
PR is owed.

---

## Cross-references

The onboarding docs **link** to canonical sources rather than
duplicating them. The canonical sources are:

- [`AGENTS.md`](../../AGENTS.md) — operating manual, hard rules.
- [`README.md`](../../README.md) — public-facing repo intro.
- [`CONTRIBUTING.md`](../../CONTRIBUTING.md) — contribution
  guidelines.
- [`SECURITY.md`](../../SECURITY.md) — security policy.
- [`TESKEL_FULLSTACK_BUILD_BREAKDOWN.md`](../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md)
  — master plan.
- [`.agents/`](../../.agents/) — agent state, glossary, skills.
- [`docs/architecture/`](../architecture/), [`docs/security/`](../security/),
  [`docs/observability/`](../observability/), [`docs/data/`](../data/),
  [`docs/api/`](../api/), [`docs/release/`](../release/),
  [`docs/marketplace/`](../marketplace/), [`docs/billing/`](../billing/),
  [`docs/runbooks/`](../runbooks/), [`docs/phases/`](../phases/),
  [`docs/stories/`](../stories/), [`docs/adr/`](../adr/),
  [`docs/rfc/`](../rfc/) — domain docs.

If a link in this directory rots, fix it in the next PR.
