# Requests for Comments (RFCs)

RFCs are exploratory designs that converge on a recommendation. They
exist to gather alignment **before** code lands. The output of a
recommended RFC is usually an [ADR](../adr/README.md).

> Author with the [`write-rfc`](../../.agents/skills/write-rfc/SKILL.md)
> skill. Once the RFC is `Recommended`, write the ADR with
> [`write-adr`](../../.agents/skills/write-adr/SKILL.md).

## Layout

```
docs/rfc/
  README.md           # this file
  0000-template.md    # copy this for new RFCs
  0001-…              # one RFC per proposal, numbered
  …
```

## Index

> Populated as RFCs land.

| # | Title | Status | Author | Outcome |
| --- | --- | --- | --- | --- |
| _placeholder — first RFCs land when a non-trivial design surfaces._ | | | | |

## When to write an RFC vs jump to PR

| Symptom | Action |
| --- | --- |
| "I have a clear plan, just need code review" | PR. |
| "There are 2–3 plausible approaches" | RFC. |
| "This contradicts AGENTS.md / plan" | RFC **mandatory**. |
| "I'm changing how a whole subsystem works" | RFC. |
| "I'm adding a major dependency" | RFC. |
| "Quick refactor" | PR. |
| "I want to brainstorm" | Draft RFC, mark `Draft`. |

## Status values

| Status | Meaning |
| --- | --- |
| `Draft` | Author still writing. Not yet open for review. |
| `Open for comments` | Active review window. Author drives discussion. |
| `Recommended` | Converged on an approach. ADR pending. |
| `Implemented` | Code shipped. Postscript section optional. |
| `Rejected` | Decided not to proceed. Recorded for history. |
| `Withdrawn` | Author retracts. Recorded for history. |
| `Stalled` | Open >30 days without consensus. Re-opened or withdrawn. |

## Hard rules

1. **Recommended ≠ Accepted.** A recommended RFC is not yet binding;
   the ADR is. If shipping is urgent, write the ADR same-day.
2. **Don't delete RFCs.** Even `Rejected` and `Withdrawn` RFCs stay
   in the repo as the historical record.
3. **Update the doc, not the comment thread.** When discussion
   resolves something, fold it into the RFC body. Comments are
   process; the doc is product.
4. **Time-box.** Open RFCs longer than 30 calendar days are stale.
   Either close (`Withdrawn`) or escalate to a sync.

## Conventions

- File: `NNNN-kebab-case-slug.md`, zero-padded to 4 digits.
- One question per RFC. Don't bundle.
- Length target: 1,500–4,000 words. Past 4,000, split.
- Always include "Drop-dead criteria" — what would make us abandon
  this proposal. Forces honesty.

## Authoring

Use the [`write-rfc`](../../.agents/skills/write-rfc/SKILL.md)
skill. The skill walks:

1. Pick the next number.
2. Copy `0000-template.md`.
3. Fill sections (Summary → Motivation → Goals → Detailed proposal
   → Alternatives → Risks → Cost → Rollout → Drop-dead → Decision
   request).
4. Open PR with `Open for comments`.
5. Drive discussion; fold accepted feedback into the doc.
6. Resolve to `Recommended` / `Rejected` / `Withdrawn`.
7. If `Recommended`, write the ADR (and update plan if relevant).

## Lifecycle

```
[ Draft ]
   │  open PR + mark "Open for comments"
   ▼
[ Open for comments ]
   │  drive discussion (≤30 days)
   ├──► consensus → [ Recommended ] → write ADR → [ Implemented ]
   ├──► no, after all → [ Rejected ]
   └──► retracted → [ Withdrawn ]
                                    or
                                [ Stalled ] (>30 days, decide manually)
```

## Auto-generated index

Planned: `pnpm teskel rfc index` regenerates the table above from
RFC frontmatter.

## Related

- [`write-rfc`](../../.agents/skills/write-rfc/SKILL.md) — author
  skill.
- [`write-adr`](../../.agents/skills/write-adr/SKILL.md) —
  formalizes a Recommended RFC.
- [`AGENTS.md` §15](../../AGENTS.md#15-rfc--adr--plan-update-flow) —
  the broader RFC → ADR → plan-update flow.
- [ADR index](../adr/README.md) — final, immutable decisions.
