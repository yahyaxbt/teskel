# Architecture Decision Records (ADRs)

ADRs are immutable records of architectural decisions. They explain
*why* the codebase is the way it is. Future readers — including AI
agents — should be able to follow them without context.

> Author with the [`write-adr`](../../.agents/skills/write-adr/SKILL.md)
> skill. For early-stage debate, use an
> [RFC](../../.agents/skills/write-rfc/SKILL.md) first.

## Layout

```
docs/adr/
  README.md                     # this file
  0000-template.md              # copy this for new ADRs
  0001-better-auth.md           # one ADR per decision, numbered
  0002-drizzle-orm.md
  …
```

## Index

> Populated as ADRs are merged.

| # | Title | Status | Tags | PR |
| --- | --- | --- | --- | --- |
| _placeholder — first ADRs land in Phase 0 (ADR-0001..0010)._ | | | | |

The expected first wave (per Plan §71):

| # | Title | Phase |
| --- | --- | --- |
| ADR-0001 | Better Auth as authentication / session layer | Phase 0 |
| ADR-0002 | Drizzle ORM as DB toolkit | Phase 0 |
| ADR-0003 | BullMQ + Inngest as workflow runtimes | Phase 0 |
| ADR-0004 | Postgres RLS as the only tenant boundary | Phase 0 |
| ADR-0005 | OpenRouter as default AI gateway | Phase 1 |
| ADR-0006 | pgvector as KB vector store | Phase 1 |
| ADR-0007 | Puck as visual builder, Craft.js fallback | Phase 2 |
| ADR-0008 | E2B as default sandbox runtime | Phase 2 |
| ADR-0009 | Next.js 15 + React Server Components default | Phase 0 |
| ADR-0010 | Coolify as deploy orchestrator | Phase 0 |

## Status values

| Status | Meaning |
| --- | --- |
| `Proposed` | Open PR, under review. Not binding yet. |
| `Accepted` | Merged. Binding for all future work. |
| `Superseded by ADR-NNNN` | Older decision replaced. Both ADRs remain in repo. |
| `Deprecated` | No longer applicable; not yet replaced. Use as warning sign. |
| `Rejected` | Considered, rejected, kept for the record. |

## Hard rules

1. **ADRs are immutable.** To change a decision, write a new ADR
   that `Supersedes` the old one. Edit the old one only to flip
   `Status`. Do not rewrite history.
2. **Numbering is sequential and sparse-safe.** Don't reuse numbers,
   even of `Rejected` ADRs.
3. **Every ADR has Options Considered.** A decision with no
   alternatives looks coerced and gets ignored in 6 months.
4. **Compliance / security implications are required** when
   applicable — auditors will ask.

## Conventions

- File: `NNNN-kebab-case-slug.md`, zero-padded to 4 digits.
- One concept per ADR. If you need to settle two decisions, write
  two ADRs.
- Length target: 300–1,500 words. Anything longer is probably an
  RFC.
- Cross-link to plan sections, RFCs, related ADRs, and external
  references (vendor docs, RFCs from elsewhere).

## Authoring

Use the [`write-adr`](../../.agents/skills/write-adr/SKILL.md)
skill. The skill walks:

1. Pick the next number.
2. Copy `0000-template.md`.
3. Fill the sections honestly (including cost / risk).
4. Open PR; review with affected Owners + Security if security-
   sensitive.
5. Merge as `Accepted` or close as `Rejected` (still merge for the
   record).
6. Update plan / AGENTS.md / dependent skills if the decision
   resolves a settled rule.

## Auto-generated index

Planned: `pnpm teskel adr index` regenerates the table above from
ADR frontmatter, run nightly + as a pre-commit hook.

## Related

- [`write-adr`](../../.agents/skills/write-adr/SKILL.md) — author
  skill.
- [`write-rfc`](../../.agents/skills/write-rfc/SKILL.md) — early-
  stage debate; converts to ADR upon resolution.
- [Plan §71](../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#71-adr-architecture-decision-record-awal) —
  expected first ten ADRs.
- [`AGENTS.md` §15](../../AGENTS.md#15-rfc--adr--plan-update-flow) —
  RFC → ADR → plan-update flow.
