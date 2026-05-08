# Phases

TESKEL is built in 7 phases (0–6) over ~60 weeks. Each phase has a
brief, exit criteria, and a release. Current state lives in
[`.agents/state/current-phase.md`](../../.agents/state/current-phase.md).

> Author phase briefs with the
> [`kickoff-phase`](../../.agents/skills/kickoff-phase/SKILL.md)
> skill. Close phases by flipping every exit criterion to `verified:
> true` and tagging a release.

## Layout

```
docs/phases/
  README.md                   # this file
  00-foundation/brief.md
  01-core-workflow-ai/brief.md
  02-visual-builder-sandbox/brief.md
  03-marketplace-creator/brief.md
  04-enterprise-pre-ga/brief.md
  05-compliance-multi-region/brief.md
  06-platform-extension/brief.md
  _retro/                     # post-phase retros
    00-foundation.md
    …
```

## Index

| Phase | Slug | Status | Brief | Plan section |
| --- | --- | --- | --- | --- |
| 0 | foundation | not started | [`00-foundation/brief.md`](./00-foundation/brief.md) | [§88](../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#88-phase-0--foundation--minggu-14) |
| 1 | core-workflow-ai | future | [`01-core-workflow-ai/brief.md`](./01-core-workflow-ai/brief.md) | [§89](../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#89-phase-1--core-workflow--ai--minggu-510) |
| 2 | visual-builder-sandbox | future | [`02-visual-builder-sandbox/brief.md`](./02-visual-builder-sandbox/brief.md) | [§90](../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#90-phase-2--visual-builder--sandbox--minggu-1116) |
| 3 | marketplace-creator | future | [`03-marketplace-creator/brief.md`](./03-marketplace-creator/brief.md) | [§91](../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#91-phase-3--marketplace--creator--minggu-1722) |
| 4 | enterprise-pre-ga | future | [`04-enterprise-pre-ga/brief.md`](./04-enterprise-pre-ga/brief.md) | [§92](../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#92-phase-4--enterprise--pre-ga--minggu-2336) |
| 5 | compliance-multi-region | future | [`05-compliance-multi-region/brief.md`](./05-compliance-multi-region/brief.md) | [§93](../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#93-phase-5--compliance--multi-region--minggu-3748) |
| 6 | platform-extension | future | [`06-platform-extension/brief.md`](./06-platform-extension/brief.md) | [§94](../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#94-phase-6--platform--ekosistem--minggu-4960) |

Brief files are added by the
[`kickoff-phase`](../../.agents/skills/kickoff-phase/SKILL.md) skill
when the phase begins. Until then, the plan section is the
authoritative document.

## Status definitions

- **not started** — no work yet; brief may be a draft.
- **active** — listed in `current-phase.md`; team is shipping.
- **closing** — feature work done, exit criteria being verified.
- **closed** — exit criteria all verified; release tagged; retro
  filed.
- **paused** — explicitly suspended; document the trigger and resume
  criteria.
- **future** — sequential, hasn't started yet.

## Brief template

Each `brief.md` has:

```
# Phase N — <name>

- Status: not started | active | closing | closed | paused
- Window: 2026-XX-XX → 2026-YY-YY (target)
- Plan section: §NN
- Lead: @…
- Tracker tag / project: <link>

## North Star
The one outcome that defines success for this phase.

## In scope
- …
## Out of scope
- …

## Key stories (linked to tracker)
- ID — title
- …

## Exit criteria
- [ ] Criterion 1 (verifiable)
- [ ] Criterion 2 (verifiable)

## Risks
- Risk → mitigation

## ADRs expected
- ADR-XXXX — title

## Skills used heavily
- `add-package`, `add-table`, …

## Release & cutover
- Tag: vN.X.0
- Watch window: T+0 → T+30 days
- Rollback plan: …

## Resources
- Plan §NN
- Related runbooks
- Owner contacts
```

## Phase lifecycle

```
[draft brief] → kickoff-phase skill → [active]
   → exit criteria verified → [closing]
   → release tagged + retro filed → [closed]
```

Closed phases are immutable; corrections are amendments noted in
the retro.

## Retros

After every phase, file `docs/phases/_retro/<phase-slug>.md`. Same
structure as `incident-sev2` retros: timeline, what worked, what
didn't, action items for the next phase.

## Related

- [`.agents/state/current-phase.md`](../../.agents/state/current-phase.md)
  — machine-readable state.
- [`kickoff-phase`](../../.agents/skills/kickoff-phase/SKILL.md) —
  skill for opening a phase.
- [Plan §66–73](../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#66-roadmap-by-phase) —
  phase definitions.
