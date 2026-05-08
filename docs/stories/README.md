# Stories

Story-driven work tracker. A **story** is the unit of work an agent
or engineer picks up and ships. Phase briefs decompose into stories.
Stories decompose into PRs.

> Author with the [`kickoff-phase`](../../.agents/skills/kickoff-phase/SKILL.md)
> skill (which produces the initial set of stories) or by manually
> filing one against an existing phase.

## Why stories at all?

Three reasons:

1. **Reviewable scope.** A story is small enough that one human +
   one agent can completely understand it before code lands.
2. **Skill alignment.** Every story names the skill(s) it consumes.
   That makes it self-checking ("did the agent follow `add-table`?").
3. **Phase exit criteria mapping.** Each story closes 0–N exit
   criteria. The phase is done when every criterion is closed.

## File layout

```
docs/stories/
  README.md
  0000-template.md            # copy this for new stories
  P0-001-bootstrap-monorepo.md
  P0-002-add-drizzle.md
  P0-003-better-auth-setup.md
  …
  P1-…
```

ID format: `P<phase>-<NNN>-<kebab-slug>`. Numbers reset per phase.

## Lifecycle

```
[ Draft ]
   open PR with story file
[ Open ]   ← phase lead reviews, accepts
   agent picks up
[ In progress ]
   PRs land
[ In review ]
   exit criteria verified
[ Done ]
   story file moves to "## Done" section in phase brief
[ Cancelled ]
   reason recorded; doc kept
```

A story is **Done** when:

- Linked PR(s) merged.
- Acceptance criteria all checked.
- Telemetry / runbook updated where applicable.
- The skill(s) it consumed list it as completed (informally — the
  story file is the record).

## Template

See [`0000-template.md`](./0000-template.md). At minimum:

- ID, phase, owner.
- One-paragraph user story (or operational story).
- Acceptance criteria (verifiable).
- Skills consumed.
- Risks + assumptions.
- Linked exit criteria from `current-phase.md`.

## Index

> Populated as stories are filed. Phase 0 first wave (planned, see
> Plan §88):

| ID | Title | Phase | Status | Skill(s) |
| --- | --- | --- | --- | --- |
| _placeholder — first stories land at Phase 0 kickoff_ | | | | |

A skeleton example for Phase 0 (filed at kickoff):

| ID | Title | Skill(s) |
| --- | --- | --- |
| P0-001 | Bootstrap monorepo (pnpm + Turborepo) | `bootstrap-monorepo` |
| P0-002 | Add Drizzle + Postgres + RLS scaffolding | `add-table` |
| P0-003 | Wire Better Auth + session middleware | `add-package` + ad-hoc |
| P0-004 | Configure Coolify deploy + CI/CD | `add-package` + ad-hoc |
| P0-005 | Wire Sentry + OTel + PostHog | `add-package` + ad-hoc |
| P0-006 | First runbook: API high-error-rate | `add-runbook` |
| P0-007 | First feature flag wiring + kill-switch | `add-feature-flag` |
| P0-008 | First RBAC role + permissions registry | `add-rbac-role` |
| P0-009 | ADR-0001..0010 first wave | `write-adr` ×10 |
| P0-010 | Phase 0 release v0.1.0 | `release-canary` |

## Conventions

- **One story = one outcome.** If you find yourself listing two
  outcomes, split.
- **Stories don't morph.** If scope expands, close current story,
  open a follow-up. Linked via `Follow-up of: P0-NNN`.
- **Stories link to PRs, not the other way around.** PR descriptions
  reference the story ID. Story doc gets updated as PRs land.
- **Stories outlive their PRs.** Even after merge, the story file
  stays (it's the rationale for the work).
- **Stories don't replace the plan.** They are the implementation
  record; the plan is the strategy.

## Estimation

We don't story-point. We size by:

- **Tiny** — <1 day, 1 PR.
- **Small** — 1–3 days, 1–3 PRs.
- **Medium** — 1 week, 3–5 PRs.
- **Large** — needs to be split. Open as an [RFC](../rfc/README.md).

If a "Medium" stretches past a week, declare it Large retroactively
and split.

## Related

- [`kickoff-phase`](../../.agents/skills/kickoff-phase/SKILL.md) —
  produces the initial story set per phase.
- [`docs/phases/`](../phases/README.md) — where phase briefs live.
- [`.agents/state/current-phase.md`](../../.agents/state/current-phase.md)
  — exit criteria stories close.
