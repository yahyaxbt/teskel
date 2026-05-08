# <P0-NNN — Story title>

- **ID:** P0-NNN
- **Phase:** 0 (foundation)
- **Owner:** @<github-handle>
- **Status:** Draft | Open | In progress | In review | Done | Cancelled
- **Size:** Tiny | Small | Medium
- **Filed:** YYYY-MM-DD
- **Tracker:** <link to issue / project>
- **Related stories:** P0-NNN, P0-NNN
- **Skills consumed:** [`<skill-1>`](../../.agents/skills/<skill-1>/SKILL.md), [`<skill-2>`](../../.agents/skills/<skill-2>/SKILL.md)
- **Linked exit criteria:** `current-phase.md#<criterion-id>`

## Story

> One-paragraph user story (or operational story for non-user-
> facing work).
>
> Format suggestion:
> _As a `<role>`, I want `<thing>` so that `<benefit>`._
>
> For platform/operational stories that don't have an external
> user, just describe the outcome:
> _The platform writes audit events with hash-chained integrity
> so that tamper-resistance can be verified end-to-end._

## Why now

1–3 sentences. Why this story is in scope for this phase, not
later. Cross-link to plan section if applicable.

## Acceptance criteria

Verifiable, binary. Reviewer can check each box without ambiguity.

- [ ] Behavior: …
- [ ] Behavior: …
- [ ] Telemetry: counter `<name>` exists with the expected labels.
- [ ] Telemetry: dashboard panel "<name>" added.
- [ ] Tests: unit + integration covering …
- [ ] Tests: tenant-isolation test if tenant-scoped table.
- [ ] Docs: relevant skill / runbook / matrix updated.
- [ ] Skills followed end-to-end (list above).

## Out of scope

Bullet list. Things reviewers might ask about that this story
explicitly defers. Each item should reference the future story
that picks it up, if known.

- Foo (covered by P0-NNN).
- Bar (deferred to Phase 1).

## Implementation notes

Optional. Pre-decisions or constraints that aren't part of the
acceptance criteria but the implementer should know:

- "Use `db.withTenant` for all writes; do not bypass RLS."
- "Reuse the existing `notifications.email` package; do not add a
  second email path."
- "Avoid introducing a new dependency unless necessary; the
  `add-package` skill applies if you do."

## Risks & assumptions

| Risk | Likelihood | Impact | Mitigation |
| --- | --- | --- | --- |
| … | low / med / high | low / med / high | … |

| Assumption | Validation |
| --- | --- |
| … | We confirmed by … |

## Open questions

- … (resolve before "In progress")

## Linked PRs

> Updated as PRs land. PR description references this story ID.

- #PR — title — status

## Notes during execution

> Free-form notes captured by the implementer. Useful for the
> retro and for the next agent picking up similar work.

—

## Done checklist

- [ ] All acceptance criteria checked.
- [ ] All linked PRs merged.
- [ ] Telemetry visible in dashboards.
- [ ] Skill checklists fully followed.
- [ ] Linked exit criteria flipped to `verified: true` in
      `current-phase.md` (if this story closes one).
- [ ] Story moved to phase brief's "## Done" section.
