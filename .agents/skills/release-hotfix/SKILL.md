---
name: release-hotfix
description: Ship an emergency fix to production outside the normal release train. Skips canary if and only if the alternative is worse than the bug. Documents what was bypassed, ensures rollback ready, communicates clearly, and reconciles main afterward.
---

# release-hotfix — emergency production fix

> Plan ref: [Sec. 95 (Final cutover)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#95-final-cutover-playbook),
> [Sec. 96 (Day-2 ops)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#96-day-2-operations-cadence),
> Companion: [`release-canary`](../release-canary/SKILL.md),
> [`incident-sev1`](../incident-sev1/SKILL.md), [`incident-sev2`](../incident-sev2/SKILL.md).

> **Hard rule:** a hotfix is a **declared exception**, not a habit.
> If you've shipped >2 hotfixes in 7 days, you have a release-process
> bug — open an incident on the **process**, not just the symptoms.

## When a hotfix is justified

| Situation | Hotfix? |
| --- | --- |
| P0/P1 outage with active customer impact | **Yes** |
| Data corruption ongoing (write path) | **Yes** |
| Security vuln being actively exploited | **Yes** |
| CVE in dependency, exploit public, blast radius high | **Yes** |
| Feature that 5% of users complain about | **No** — next train |
| Cosmetic bug | **No** |
| Config change | **Use feature flag**, not code hotfix |
| Vendor outage we depend on | **No** — circuit breaker / kill-switch |

When unsure, treat as Sev2 first (`incident-sev2` skill); a hotfix
follows from incident triage, not vice versa.

## Pre-flight

1. **Designate the Hotfix Commander.** The same person who is running
   the incident if there is one. They:
   - Approve the diff size (target ≤200 LoC).
   - Decide whether canary is bypassed.
   - Own communication.
   - Accept rollback responsibility.

2. **Confirm scope.** A hotfix touches **only** what's required to
   stop the bleeding. Do not refactor adjacent code, do not add
   cleanups, do not bump deps. If you find adjacent issues, file
   tickets — don't include them in the hotfix.

3. **Write the rollback plan first.** Before writing the fix:
   - Can the previous tag be re-deployed cleanly? (Migration-aware
     check; see step 7.)
   - Or is rollback a feature flag flip?
   - Or is rollback a forward-fix only? (Worst case; document why.)

   Hotfix MUST NOT ship if rollback isn't faster than the fix's blast
   radius would justify.

## Steps

1. **Cut the branch off prod tag.**

   ```bash
   git fetch --tags origin
   git checkout -b hotfix/<short-slug> v0.X.Y      # current prod tag
   ```

   Hotfixes branch off the **current prod tag**, not `main`. This
   keeps the diff minimal and avoids accidentally shipping merged-but-
   unreleased work.

2. **Implement the fix.** Smallest change that resolves the issue.
   - No formatting churn.
   - No dependency bumps.
   - One concept per commit.

3. **Add a regression test.** Always. Even under pressure. The test
   reproduces the bug against `main` (would fail) and passes after
   the fix. Without this, the next deploy can re-introduce it.

4. **Verify locally + in staging.**
   - Unit + integration suite green.
   - Manual repro of the bug on staging shows it's fixed.
   - Smoke pack passes.

5. **Open the PR with `[hotfix]` prefix.**
   - Title: `[hotfix] <short-slug>` (matches branch).
   - Description must include:
     - Incident link / Sentry / Grafana evidence of impact.
     - Root cause (1-paragraph).
     - Why the normal release train is too slow (concrete impact $/min).
     - Rollback plan.
     - Whether canary is bypassed and why.
     - Sign-offs needed: Commander + 1 Owner per affected area + Security
       if security-related.

6. **Approval (compressed).**
   - For P0/P1: minimum **2** approvals (Commander + 1 Owner).
   - Skip-level review (CTO / Head of Eng) informed but not blocking.
   - Time budget: ≤30 min from PR open to merge for Sev1.

7. **Migration check.**
   - **No DB migrations** in a hotfix. Period. Migrations are not
     hotfix-grade. If a migration is required, it's a different
     class of incident — escalate to data team and treat as Sev1
     change-management.
   - If this hotfix follows a release that included a migration, you
     may need to also reverse the migration if it's the proximate cause —
     but that is a separate, planned operation, not a hotfix.

8. **Tag the release.**

   ```bash
   git tag -a v0.X.Y+1 -m "hotfix: <short-slug>"
   git push origin v0.X.Y+1
   ```

   Versioning: `v<major>.<minor>.<patch>+1`, never reuse a number,
   never overwrite a tag.

9. **Deploy: canary or skip.**
   - **Default: still canary.** Even hotfixes flow through canary,
     ~10 min observation window (vs the normal hour). The active bug
     is already 100% impact; a 5-minute canary detection window is
     cheap insurance.
   - **Skip canary** only if:
     - The bug is on the canary path itself, OR
     - Canary infra is the broken thing, OR
     - You can prove the fix is provably correct (e.g., revert).

   Deploy via Coolify with `--strategy=canary` or `--strategy=full`
   per above.

10. **Watch.** Hotfix deploys are observed in real time:
    - Error rate dashboards on the affected surface.
    - Synthetic checks.
    - The original alarm clearing.

    If error rate increases for ≥3 min — immediate rollback (step 11).

11. **Rollback path.**
    - **Tag-based:** redeploy `v0.X.Y` via Coolify; takes <60s.
    - **Flag-based:** flip the relevant flag.
    - **Forward-fix:** open another hotfix on top.

    Whoever has Commander role decides at the 3-min mark.

12. **Communicate.**
    - Status page: "investigating → identified → monitoring →
      resolved" cadence.
    - Internal channel ping at deploy start, deploy end, all-clear.
    - Post-incident note in `#incidents` with timestamps.

13. **Reconcile with `main`.**

    ```bash
    git checkout main
    git fetch origin
    git merge --no-ff hotfix/<slug>     # or cherry-pick if main has diverged
    git push origin main
    ```

    The hotfix branch and tag stay in history; do not delete.

14. **Postmortem.**
    - **Sev1 hotfix:** blameless postmortem within 5 business days
      (`incident-sev1` skill).
    - **Sev2 hotfix:** lightweight retro within 7 days, captured in
      `docs/incidents/<date>-<slug>.md`.

    Action items: a regression test (you wrote one already, right?),
    a process fix, and — if the bug was config — a config-validation
    test added to CI.

15. **Update the runbook.** If the path that detected the bug is
    flaky or unclear, update the relevant runbook (`add-runbook`
    skill).

## What you DON'T do during a hotfix

- Don't refactor.
- Don't bump dependencies.
- Don't run a DB migration.
- Don't change feature flags as part of the diff (do it separately
  via flag service).
- Don't merge unrelated PRs along with the hotfix to "save a deploy".
- Don't push to `main` then to prod tag — go through PR + approval.
- Don't `git push --force` anything.

## Pitfalls

- Hotfix that grows from "tiny revert" to "while we're here" — kill
  the scope creep at PR review.
- Skipping the regression test — bug recurs in a week.
- Skipping postmortem — same incident repeats.
- Forgetting to merge back to `main` — the next train re-introduces
  the bug.
- Bypassing approval to "save time" — a wrong hotfix is more
  expensive than the bug.
- Hotfix during release-watch window of a phase kickoff — defer to
  phase team if at all possible.

## Done when

- PR merged with `[hotfix]` prefix and required approvals.
- Tag `v0.X.Y+1` pushed; deployed via canary or documented bypass.
- Original alarm cleared; synthetic + error rate stable for ≥30 min.
- Branch merged back to `main`.
- Status page green.
- Postmortem scheduled (Sev1) or retro filed (Sev2).
- Action items captured in tracker.
