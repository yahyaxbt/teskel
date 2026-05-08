---
name: kickoff-phase
description: Procedure to formally kick off a new build phase (Phase 0..6) — update current-phase.md, write phase brief, derive stories, register exit criteria, set up tracking, and announce. Use whenever the previous phase exit gate has been met.
---

# kickoff-phase — start a new phase

> Plan ref: [Sec. 66 (Roadmap)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#66-roadmap--phase-06-detail),
> [Sec. 67 (Story breakdown)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#67-story-breakdown-template),
> [Sec. 87..94 (Phase walkthroughs)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#87-pre-flight--day-0),
> AGENTS.md §12 (phase awareness).

## Pre-conditions (don't kick off yet if any of these is unchecked)

- [ ] Previous phase exit criteria fully green (verified, not
      asserted).
- [ ] Final cutover/release of previous phase has cleared its post-
      release watch window (see plan §95).
- [ ] All in-flight bug-fix PRs from the previous phase merged or
      explicitly deferred to backlog.
- [ ] Current-phase OKRs / goals agreed with stakeholders.
- [ ] Capacity plan in place (who's working on what; on-call rota).
- [ ] Risks listed for the new phase (see plan §74 risk register).

If any item fails, fix it before invoking this skill.

## Steps

1. **Open a kickoff PR** on a fresh branch
   `devin/<ts>-phase-<N>-kickoff`. This PR collects all the changes
   below in a single review.

2. **Update `.agents/state/current-phase.md`.** This is the source of
   truth that every agent reads first.

   ```yaml
   # .agents/state/current-phase.md
   phase: 1
   phase_name: Core Workflow + AI
   phase_window: Week 5-10
   status: active           # planning | active | feature-freeze | closing
   plan_section: 89
   release_target: v0.2.0
   started_at: 2026-06-15
   exit_due: 2026-07-26

   allowed_scopes:
     - workflow_runtime
     - ai_gateway_runtime
     - prompts_registry
     - kb_pgvector
     - bullmq_workers
     - studio_react_flow
     - marketplace_skeleton
     - 3_seed_templates

   out_of_scope:
     - visual_builder        # Phase 2
     - sandbox               # Phase 2
     - sdk
     - cli
     - saml_sso              # Phase 4
     - multi_region          # Phase 5
     - on_prem               # Phase 6

   exit_criteria:
     - id: e2e-workflow
       text: |
         A user can author a 3-node workflow in Studio, run it on
         BullMQ, and see the result + Langfuse trace.
       verified: false
     - id: ai-gateway-prod
       text: |
         AI gateway routes via OpenRouter, with primary + fallback,
         budget enforcement, and PII redaction. 95% of slots have an
         eval suite >= 90% pass.
       verified: false
     - id: kb-pgvector
       text: |
         pgvector index serves <50ms p95 retrieval over 1M rows in
         staging.
       verified: false
     - id: marketplace-skeleton
       text: |
         /marketplace lists 3 seed templates; install flow installs a
         template into a fresh org end-to-end.
       verified: false
     - id: tests
       text: |
         Unit ≥80% on AI + workflow packages; Playwright covers the
         workflow author-and-run golden flow.
       verified: false
     - id: adr
       text: |
         ADRs 0011..0014 written and merged (workflow runtime, AI
         gateway, KB, prompt registry).
       verified: false
   ```

   Update **only** the fields that change. Don't drop history — keep a
   `phases_history` block at the bottom:

   ```yaml
   phases_history:
     - phase: 0
       name: Foundation
       started_at: 2026-05-08
       finished_at: 2026-06-13
       release: v0.1.0
   ```

3. **Write the phase brief.** New file:
   `docs/phases/phase-<N>-brief.md`. Sections:

   - Goal (1 sentence).
   - Why now.
   - In scope (bullet list, mirrors `allowed_scopes`).
   - Out of scope (bullet list, mirrors `out_of_scope`).
   - Top 3 risks + mitigations.
   - Architecture deltas (link to ADRs being written this phase).
   - Hiring / capacity needed.
   - Demo (what we'll demo at the exit gate).

4. **Derive stories from the plan.** Open the matching plan section
   (e.g., §89 for Phase 1) and turn each deliverable into a
   Linear/issue. Each story uses the [story breakdown template](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#67-story-breakdown-template):
   - Title, why, acceptance criteria, tech notes, telemetry, risks,
     test plan, owner, ETA.
   - Tag with `phase:N`, `epic:<area>`.
   - Link to plan section + relevant ADR/RFC.

5. **Register exit criteria as machine-checkable.** Each criterion
   should be tied to a check:
   - A passing CI job.
   - A Grafana dashboard panel reaching a threshold for ≥7 days.
   - A Linear story marked Done with a screenshot.
   - A specific artifact in the repo (ADR file, eval result).

   Track in `docs/phases/phase-<N>-exit.md` with the same shape as
   `current-phase.md exit_criteria` plus evidence URLs as they fill
   in.

6. **Update the README roadmap section.** Reflect that Phase N-1 is
   complete and Phase N is active. Update `Status` + `Phase` badges.

7. **Open ADRs as drafts.** Any decisions the phase will lock in
   should have a draft ADR file in `docs/adr/00xx-...-draft.md` so
   the team starts thinking about them early. Promote draft → ADR
   when decision is made.

8. **Update `.agents/state/owners.md`** if owners change for the new
   phase.

9. **Set up tracking.**
   - Linear: phase view filtered by `phase:N` label.
   - GitHub project: column per epic.
   - PostHog dashboard: `Phase N · Adoption` if user-visible features
     ship.
   - Grafana dashboard: `Phase N · Health` linking SLOs that this
     phase introduces.

10. **Announce.** Post in the team channel (or the project's chosen
    venue) with:
    - Link to the phase brief.
    - Link to the kickoff PR.
    - Stories owned by each person.
    - First standup date.

## Closing a phase

When all `exit_criteria.verified == true`:

1. Cut the release branch and tag `vX.Y.0` per plan §95.
2. Run the cutover playbook from plan §95.
3. Publish the demo video / screenshots in
   `docs/phases/phase-<N>-demo.md`.
4. Move `current-phase.md` → next phase via this skill again.
5. Move all open `phase:N` stories to either Done or backlog (with
   note explaining why deferred).

## Pitfalls

- Kicking off the new phase while the old one's release is still in
  the post-launch watch window — risk of context-switch mid-fire.
- Updating only `current-phase.md` without writing the brief — no
  shared understanding for new contributors.
- Writing soft, untestable exit criteria ("workflow feels good") —
  every criterion must be checkable.
- Over-scoping the phase. If the exit-due date slips by >2 weeks, cut
  scope, don't extend.
- Skipping the ADR drafts — phase ends without canonical decisions
  recorded.

## Done when

- `.agents/state/current-phase.md` reflects the new phase + history.
- `docs/phases/phase-<N>-brief.md` written.
- Stories derived from plan §<N> and tracked.
- Draft ADRs filed.
- README + dashboards updated.
- Kickoff PR merged.
- Team announced and aligned.
