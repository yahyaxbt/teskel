---
name: write-adr
description: Author an Architecture Decision Record in docs/adr/ documenting a chosen technology, pattern, or constraint that future agents must respect. Use when settling a stack/architecture/operational decision (e.g., "we use Drizzle, not Prisma"). ADRs are immutable history; new decisions append.
---

# write-adr — author an Architecture Decision Record

> Plan ref: [Sec. 71 (10 ADR awal)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#71-adr-architecture-decision-record-awal),
> [Sec. 22 (Stack settled)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#22-stack-keputusan-default).

> **Hard rule:** ADRs are **immutable** once `Accepted`. To change a
> decision, write a new ADR that `Supersedes: ADR-XXXX`. Never edit
> the original except to flip `Status: Superseded by ADR-YYYY`.

## When you need an ADR

Write an ADR when the decision:

- Affects multiple packages or surfaces.
- Constrains future implementation choices.
- Has plausible alternatives a reasonable engineer would pick.
- Will be questioned 6 months from now ("why did we pick X?").

Examples that ARE ADRs:
- Pick Drizzle vs Prisma vs Kysely.
- Pick BullMQ vs Inngest as primary orchestrator.
- Pick Coolify vs Render vs Fly as deploy target.
- Adopt RLS as the only multi-tenant boundary.
- Adopt Server Components as default (with explicit Client opt-in).

Examples that are NOT ADRs (use code review or RFC):
- Refactor a function.
- Pick a UI library for one component.
- Internal naming convention (use `CONTRIBUTING.md`).
- Day-to-day implementation details.

For bigger, non-binary debates, use an [RFC](../write-rfc/SKILL.md)
first, then convert the conclusion to an ADR.

## Steps

1. **Confirm the gap.** Search `docs/adr/` for prior decisions on the
   same topic. If one exists and is `Accepted`, you're either
   re-opening (write a new ADR that `Supersedes` it) or
   misunderstanding (close out without writing).

2. **Pick the next number.**

   ```bash
   ls docs/adr/ | grep -Eo '^[0-9]{4}' | sort -n | tail -1
   ```

   ADRs are numbered `NNNN-title-slug.md`, zero-padded to 4 digits.

3. **Copy the template.**

   ```bash
   cp docs/adr/0000-template.md docs/adr/0014-postgres-rls-as-tenant-boundary.md
   ```

4. **Fill the template.** Sections:

   ```
   # ADR-0014: Postgres RLS as the only tenant boundary

   - Status: Proposed | Accepted | Superseded by ADR-XXXX | Deprecated
   - Date: 2026-05-08
   - Deciders: @sec-lead, @platform-lead, @cto
   - Tags: security, multi-tenant, database

   ## Context
   2–4 paragraphs. What is forcing a decision? Why now?

   ## Decision
   1–3 sentences in active voice. "We will <do X>."

   ## Options considered
   ### Option A — Postgres RLS (chosen)
   Pros / Cons / Cost / Risk

   ### Option B — Application-level filters
   Pros / Cons / Cost / Risk

   ### Option C — Per-tenant schema
   Pros / Cons / Cost / Risk

   ## Consequences
   Positive: …
   Negative: …
   Mitigation for negatives: …

   ## Compliance & security implications
   Touches: SOC2 CC6.1, ISO 27001 A.5.15, GDPR Art. 32.
   Requires: tenant isolation tests in CI.

   ## Migration / Rollout plan
   - Phase 0: enable on new tables (default).
   - Phase 1: backfill old tables (data-backfill-job per table).
   - Phase 2: enforce via CI lint that all tenant tables have RLS.

   ## Open questions
   - …

   ## References
   - Plan §22, §49.
   - Postgres RLS docs.
   - <links to vendor whitepapers, comparable ADRs in industry>
   ```

5. **Be honest about cost & risk.** An ADR that lists only "pros" for
   the chosen option is suspicious. Reviewers will (and should) push
   back. Acknowledge:
   - Operational cost (training, maintenance).
   - Vendor lock-in / exit cost.
   - Migration cost from current state.
   - Failure modes the decision creates.

6. **List who is bound by this ADR.** Sometimes the ADR applies only
   to a subset (e.g., "only the workflow runtime, not the AI gateway").
   Make scope explicit; otherwise readers assume "everywhere".

7. **Open as PR.** Title: `ADR-NNNN: <slug>`. Status: `Proposed`.

8. **Review.**
   - At least one Owner per affected area (see CODEOWNERS).
   - Engineering lead.
   - Security lead if security-sensitive.
   - Document discussion in PR comments — those are part of the
     record.

9. **Resolution.**
   - **Merged** as `Status: Accepted` → it's the law.
   - **Closed** without merging → write down why in a comment; do
     **not** delete the file if it spent meaningful time in review;
     change status to `Rejected` and merge as historical record.

10. **Update plan.** If the ADR settles a previously open question in
    the plan, update `TESKEL_FULLSTACK_BUILD_BREAKDOWN.md`'s relevant
    section to point at the ADR. If the ADR contradicts the plan,
    that's a bigger discussion — block the ADR until plan owner is
    aligned.

11. **Update derived artifacts.**
    - `AGENTS.md` §6 (Stack) if the ADR finalizes a stack choice.
    - `.agents/state/current-phase.md` exit criteria if it impacts a
      phase gate.
    - Skills that reference the relevant area (e.g., if you change
      tenant model, every skill that mentions `withTenant` is touched).

12. **Supersede flow.** When a new ADR replaces an old one:
    - Old ADR: `Status: Superseded by ADR-NNNN`.
    - New ADR: `Status: Accepted`. Add a "Migration from ADR-XXXX"
      section.
    - Both files stay in the repo (history matters).

## Pitfalls

- Editing an `Accepted` ADR — destroys history. Always supersede.
- ADR with no "Options considered" — looks like a fait accompli; ignored
  in 6 months when context is lost.
- ADR for a one-off implementation detail — clutters `docs/adr/` and
  trains the team to ignore them.
- Writing ADR after the code is shipped — fine occasionally; never as
  a habit. ADR-as-rationalization-after-the-fact undermines trust.
- Vague decision sentence ("We will use modern tools") — useless.
  Write the imperative: "We will use Drizzle ORM 0.30+ for all
  schema migrations and queries."
- Forgetting Compliance/Security section for security-sensitive
  decisions — auditors will ask.

## Done when

- File `docs/adr/NNNN-<slug>.md` exists, follows template.
- Status: `Accepted`, merged.
- Plan section + AGENTS.md §6 cross-referenced if applicable.
- Old ADR (if superseding) is updated to `Superseded by`.
