---
name: write-rfc
description: Author a Request for Comments in docs/rfc/ to propose a substantial change before writing code. Use for non-trivial design work that needs broad alignment (≥2 days of work, crosses team boundaries, has multiple plausible approaches). RFCs converge on a recommendation; the recommendation typically becomes an ADR.
---

# write-rfc — author a Request for Comments

> Plan ref: [Sec. 71 (ADRs)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#71-adr-architecture-decision-record-awal),
> [AGENTS.md §15 (RFC → ADR flow)](../../../AGENTS.md#15-rfc--adr--plan-update-flow).

> **Hard rule:** if your change conflicts with `AGENTS.md` or
> `TESKEL_FULLSTACK_BUILD_BREAKDOWN.md`, you **must** write an RFC
> first, even if the change seems small. Don't open a code PR that
> contradicts settled rules without an RFC merging first.

## RFC vs ADR vs PR

| Artifact | Purpose | Outcome |
| --- | --- | --- |
| **RFC** | Explore the problem space; converge on an approach | A recommendation, or "defer / reject". Often births an ADR. |
| **ADR** | Record the chosen decision | Immutable, prescriptive |
| **PR** | Implement the work | Code + docs |

If a colleague would say "I'm not sure, let's discuss first" — that's
an RFC. If you'd say "the obvious thing here is X" — open a PR.

## When you need an RFC

- Adding a major new dependency (vendor SDK, infra component).
- Proposing a new architectural layer (e.g., introduce a service mesh).
- Changing a hard rule in AGENTS.md.
- Designing a feature that touches >3 packages or >2 surfaces.
- Backwards-incompatible API change.
- Anything where reviewers might say "wait, this contradicts $X".

## Steps

1. **Write the one-paragraph problem statement first.** If you can't
   describe the problem in one paragraph, you're not ready to write
   the RFC.

2. **Pick the next number.**
   ```bash
   ls docs/rfc/ | grep -Eo '^[0-9]{4}' | sort -n | tail -1
   ```
   `NNNN-title-slug.md`, zero-padded.

3. **Copy the template.**
   ```bash
   cp docs/rfc/0000-template.md docs/rfc/0007-multi-region-data-residency.md
   ```

4. **Fill the sections.**

   ```
   # RFC-0007: Multi-region data residency

   - Status: Draft | Open for comments | Recommended | Rejected | Withdrawn | Implemented
   - Author: @your-name
   - Reviewers: @sec-lead, @sre-lead, @cto, @legal-lead
   - Created: 2026-05-08
   - Target ADR: TBD (will be ADR-NNNN if accepted)
   - Phase impact: Phase 5 (Compliance)
   - Tags: data-residency, security, compliance

   ## Summary
   2–3 sentences a busy lead can read.

   ## Motivation
   Why now. What customer demand or risk forces this. Quote actual
   tickets / sales asks if applicable.

   ## Goals & non-goals
   Goals:
   - Allow EU customers to pin their data to eu-west-1.
   - …
   Non-goals:
   - Region-pinning per workspace (org-level only).
   - …

   ## Detailed proposal
   The actual design. Include:
   - Data model changes.
   - Routing / request flow.
   - Failure modes and mitigations.
   - Operational implications.
   - Migration / rollout.

   Keep this section honest. If something isn't designed yet, say so.

   ## Alternatives considered
   ### Alt A — single-region with encryption per region
   ### Alt B — replicate everywhere, customer chooses primary
   ### Alt C — defer until SOC2 Type 2

   ## Risks & open questions
   - Cost: replication doubles storage spend.
   - Latency: cross-region writes for shared resources.
   - Open: how do we treat embeddings (re-compute per region?).

   ## Security & compliance
   - SOC2 CC1.1, CC6.1 implications.
   - GDPR Art. 44–50 (international transfers).
   - HIPAA — out of scope until Phase 5 ramp.

   ## Cost estimate
   - One-shot: ~30 engineer-weeks.
   - Recurring: ~$8k/mo additional infra.

   ## Rollout plan
   - M0 (current): single-region.
   - M1: schema marker `data_region` on `org`.
   - M2: dual-write in EU.
   - M3: cutover criteria.
   - M4: GA.

   ## Drop-dead criteria
   We will abandon this proposal if:
   - Customer demand <X paying enterprise accounts.
   - Vendor support insufficient.
   - Cost overrun >2× estimate.

   ## Decision request
   Specifically asking: do we accept Alt A, with rollout starting
   Phase 5 W3, blocking go-live until M3 cutover?
   ```

5. **Set status `Open for comments`.** Open the PR. Add to the bottom
   of the description:
   - Comment-by date (default: 7 calendar days).
   - Specific reviewers and what feedback you need from each.

6. **Drive the discussion.** Reply to every substantive comment.
   Update the RFC body to incorporate accepted feedback (with a
   `> changed YYYY-MM-DD: …` annotation). Don't leave large
   debates in PR comments — fold the resolution into the doc.

7. **Hold a sync if needed.** If async discussion stalls > 2 days
   on a major axis, schedule a 30-minute review. Document the
   meeting decision in the RFC.

8. **Resolution.** Three outcomes:
   - **Recommended → ADR.** Mark RFC `Recommended`, then write the
     ADR (`write-adr` skill). Merge RFC and ADR together if possible.
   - **Rejected.** Mark `Rejected`, add a final paragraph explaining
     why, merge as historical record.
   - **Withdrawn.** Mark `Withdrawn`, merge with reason. Equivalent of
     "never mind" — useful when the problem evaporates.

9. **Don't ship until ADR exists.** A `Recommended` RFC is not yet
   law. The ADR is. Until ADR is `Accepted`, the work is not
   blessed for implementation, only for prototyping.

10. **Implementation linkage.** When the work eventually ships:
    - Update RFC status to `Implemented`.
    - Add a "Postscript" section describing what changed in
      implementation vs the RFC (almost always something does).

## Length guidance

| Topic size | Target length |
| --- | --- |
| Single subsystem | ~500–1,500 words |
| Cross-cutting feature | ~1,500–3,000 words |
| Stack-changing decision | ~2,000–4,000 words |

If you're past 4,000 words, split it. RFCs that nobody reads don't
collect feedback.

## Pitfalls

- RFC-driven development for tiny changes — bureaucracy.
- Skipping RFC for big changes — leads to "but we already discussed
  this" loops in PR review.
- Writing the RFC after coding — sometimes okay (architecture spike)
  but be explicit; fold what you learned.
- Vague decision request — reviewers don't know what they're saying
  yes/no to.
- Ignoring rejected alternatives — you'll re-litigate them in 3 months.
- Treating "Recommended" as approval to ship — the ADR is the bar.
- Letting the RFC linger in `Open for comments` for >30 days — close
  it as `Withdrawn` if the answer hasn't crystallized.

## Done when

- File `docs/rfc/NNNN-<slug>.md` exists, status terminal
  (`Recommended` / `Rejected` / `Withdrawn` / `Implemented`).
- Decision recorded in:
  - ADR (if `Recommended` → ADR written).
  - PR comments (if `Rejected` / `Withdrawn`, with reason).
- All open questions either resolved or moved to a follow-up issue.
