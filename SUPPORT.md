# Support

This page is the **first stop** for anyone who has a question about
TESKEL. Pick the bucket that matches your situation.

---

## I'm a customer / end user

| What you need | Where to go |
| --- | --- |
| Product help, "how do I…?" | https://docs.teskel.app (when live) |
| Bug report (your account / data not behaving as expected) | https://app.teskel.app → "Help" → "Report a bug" |
| Billing question (invoice, refund, payment failed) | billing@teskel.app |
| Security concern (suspected vulnerability, account compromise) | security@teskel.app — see [SECURITY.md](./SECURITY.md) |
| Marketplace dispute (template doesn't work, refund) | marketplace-support@teskel.app |
| Privacy / GDPR data request (DSAR) | privacy@teskel.app — see [docs/legal/README.md](./docs/legal/README.md) |
| Press / media | press@teskel.app |
| Sales / Enterprise | sales@teskel.app |

Response SLAs depend on plan tier — see
[docs/billing/plans.md](./docs/billing/plans.md) §2.

---

## I'm a contributor (open source / community)

| What you need | Where to go |
| --- | --- |
| Build / setup question | [docs/onboarding/new-engineer.md](./docs/onboarding/new-engineer.md) |
| Architecture / "should I…?" question | [AGENTS.md](./AGENTS.md) decision cheatsheet, or open an [RFC](./docs/rfc/) |
| Bug report on the codebase | GitHub issue using the **Bug** template |
| Feature request | GitHub issue using the **Feature request** template |
| RFC / proposal for a substantial change | [docs/rfc/](./docs/rfc/) — see [`write-rfc` skill](./.agents/skills/write-rfc/SKILL.md) |
| Security vulnerability | **Do not** open a public issue. See [SECURITY.md](./SECURITY.md). |
| Code of Conduct concern | conduct@teskel.app — see [CODE_OF_CONDUCT.md](./CODE_OF_CONDUCT.md) |
| Anything else | open a GitHub Discussion |

We do **not** accept support requests via DM to individual
maintainers. The ticketing channels above ensure your request
reaches the right team and is tracked.

---

## I'm an AI coding agent (Cursor, Claude Code, Copilot, Devin, …)

Read first:

1. [AGENTS.md](./AGENTS.md)
2. [docs/onboarding/new-agent.md](./docs/onboarding/new-agent.md)
3. [.agents/state/current-phase.md](./.agents/state/current-phase.md)

Then pick the [skill](./.agents/skills/) that matches your task.

If your task does not fit any skill, **stop** and:

- Either write an [RFC](./docs/rfc/) proposing a new skill or a
  deviation, **or**
- Ask the human reviewer in the PR description before opening any
  code change.

---

## What support is *not*

- We are not a 24/7 hotline for free-tier customers — see plan
  SLAs.
- We do not respond to support emails about competitor products or
  general AI questions.
- We do not give investment advice, tax advice, or legal advice;
  please consult a qualified professional for those.

---

## Status & incidents

- Public status page: https://status.teskel.app (when live).
- Incident comms during a Sev1: status page + email to affected
  tenants. See [`incident-sev1` skill](./.agents/skills/incident-sev1/SKILL.md).
- Pre-incident drills: see [`gameday-drill` skill](./.agents/skills/gameday-drill/SKILL.md).
