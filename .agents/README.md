# `.agents/` — agent navigation hub

This directory is the operational nerve center for AI coding agents
working on TESKEL. Humans should also browse it; everything is
written for both audiences.

> First time here? Start with the **agent operating manual**:
> [`AGENTS.md`](../AGENTS.md). This file is the navigation index.

## What's in here

```
.agents/
  README.md          # this file
  state/             # current state — read on every session start
    current-phase.md # active phase + allowed_scopes + exit criteria
    owners.md        # area → owner mapping (mirror of CODEOWNERS)
  skills/            # reusable, step-by-step procedures
    <skill>/SKILL.md
  glossary.md        # TESKEL-specific terms an agent should know
```

## Read order on a fresh session

1. **[`AGENTS.md`](../AGENTS.md)** (full operating manual) — once,
   then skim periodically. Hard rules in §8 are non-negotiable.
2. **[`state/current-phase.md`](./state/current-phase.md)** — what
   phase are we in? what's in scope?
3. **[`state/owners.md`](./state/owners.md)** — who reviews each
   area?
4. **The relevant skill(s)** under [`skills/`](./skills/) — see
   index in [`AGENTS.md` §19](../AGENTS.md#19-skills-library).
5. **[`glossary.md`](./glossary.md)** if you're unsure what a
   TESKEL-specific term means.

## Skills

A **skill** is a Markdown checklist for a recurring operation that
the team has agreed on a Right Way To Do. Skills are mandatory: if
your task matches a skill, you follow it step-by-step.

Skills are categorized in [`AGENTS.md` §19](../AGENTS.md#19-skills-library).
Lookup table: ["I'm about to..." → skill](../AGENTS.md#quick-lookup-table).

If you can't find a skill for your task:

1. Check whether the task is genuinely recurring. If yes, propose a
   new skill in your PR (one extra file under `.agents/skills/<slug>/SKILL.md`).
2. If it's a one-off, just open the PR — but if reviewers find
   themselves explaining the same checklist twice, expect to be
   asked to write the skill afterward.

## State

`state/` is the only mutable area in `.agents/` from a process
perspective:

- `current-phase.md` is updated by the
  [`kickoff-phase`](./skills/kickoff-phase/SKILL.md) skill at every
  phase transition. Until then, it is the **single source of
  truth** on what work is allowed.
- `owners.md` is mirrored from `.github/CODEOWNERS` and updated
  alongside it.

Skills under `skills/` are append/edit-only via PR — do not delete
without an [RFC](../docs/rfc/README.md).

## Glossary

`glossary.md` defines TESKEL-specific terms (`workflow`, `template`,
`block`, `slot`, `tenant`, `org`, `prompt slot`, `runner`,
`outbox`, etc.). Read it once on first session; refer back when a
term feels ambiguous.

## Per-agent pointers

The repo includes thin pointer files for popular agents:

| Agent | Pointer file |
| --- | --- |
| Claude Code | [`/CLAUDE.md`](../CLAUDE.md) |
| Cursor | [`/.cursorrules`](../.cursorrules) |
| GitHub Copilot Workspace | [`/.github/copilot-instructions.md`](../.github/copilot-instructions.md) |
| Devin (this session's author) | reads the whole repo by default |

All pointers redirect to [`AGENTS.md`](../AGENTS.md). Cursor and
Copilot also include a hard-rules summary so the agent doesn't have
to follow links.

## When something contradicts

If a skill, ADR, plan section, RFC, or your common sense disagree:

1. **ADR** wins on architecture decisions.
2. **AGENTS.md hard rules** win on operational decisions.
3. **Plan** wins on strategy.
4. **Skill** wins on the **how**.
5. **Common sense** loses.

If you genuinely think a hard rule is wrong, that's an [RFC](../docs/rfc/README.md),
not a PR.

## Outside `.agents/`

- [`AGENTS.md`](../AGENTS.md) — operating manual.
- [`TESKEL_FULLSTACK_BUILD_BREAKDOWN.md`](../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md)
  — strategic plan.
- [`docs/adr/`](../docs/adr/) — accepted decisions.
- [`docs/rfc/`](../docs/rfc/) — proposals.
- [`docs/phases/`](../docs/phases/) — phase briefs.
- [`docs/runbooks/`](../docs/runbooks/) — incident runbooks.
- [`docs/security/`](../docs/security/) — secrets registry, RBAC
  matrix.
- [`docs/stories/`](../docs/stories/) — story-driven work tracker.
- [`CONTRIBUTING.md`](../CONTRIBUTING.md) — human contributor guide.
- [`SECURITY.md`](../SECURITY.md) — security disclosure policy.
