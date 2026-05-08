---
name: release-canary
description: Promote a canary deploy to 100% in production for TESKEL. Use after merging a release-tagged commit to main. Covers pre-promotion health checks, gradual ramp, rollback triggers, and post-promotion verification.
---

# release-canary — promote canary to 100%

> Use this skill after a release-tagged commit lands on `main` and the
> Coolify auto-deploy has rolled out a canary slice (10% by default).
> Plan ref: [Sec. 36 (Deployment)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#36-deployment-infrastructure),
> [Sec. 95.5 (Launch Sequence)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#955-t-0--launch-sequence-hour-by-hour),
> [Sec. 98 (Promotion Checklist)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#98-local-dev--prod-promotion-checklist-per-story).

## Pre-promotion checklist (block promotion if any fails)

- [ ] CI green on the release commit.
- [ ] Canary has been live ≥30 minutes (or ≥10× normal release window
      traffic).
- [ ] Sentry: error rate on canary ≤ 1.2× the stable rate (no >20%
      regression).
- [ ] HTTP 5xx p95 ≤ 0.5%.
- [ ] Latency p95 ≤ 1.2× the stable p95.
- [ ] DB connection pool utilization ≤ 80%.
- [ ] Redis memory ≤ 80%.
- [ ] No new Sev1/Sev2 alerts since canary started.
- [ ] Synthetic monitor on the canary URL is green for the last 15
      minutes.
- [ ] Migrations applied cleanly (`migrations` table has the new entry,
      no errors in worker logs).
- [ ] Author of the release confirms in PR comment: "ready to promote".

If any of the above fails, **do not promote** — see "Rollback" below.

## Promotion steps

1. **Announce in `#releases` Slack/Discord:**

   ```text
   :rocket: Promoting canary to 100% — release v<x.y.z>
   PR: <link>  Author: @<handle>  IC: @<oncall>
   Watching: error rate, latency, p95 saturation
   ```

2. **Trigger the promotion in Coolify** (UI or API):
   - From the `web` service → canary slice → "Promote to 100%".
   - From the `api` service → same.
   - From any worker service that ships in the same release.

3. **Watch for 15 minutes minimum:**
   - Sentry release health page filtered to the new release.
   - Grafana "Live Health" dashboard.
   - PostHog active users / event rate (sanity check).

4. **Run synthetic smoke** on the prod URL:

   ```bash
   pnpm --filter testing smoke --env=prod
   ```

   This script (Phase 0+) covers: signup, login, create org, create
   project, run the demo workflow, view billing.

5. **Mark release "stable" in Coolify** so the next deploy cycles
   correctly.

6. **Update the changelog** (`apps/web/changelog/<x.y.z>.mdx` or the
   /changelog source) with: features, fixes, breaking changes, upgrade
   notes. Push to `main` (small follow-up PR `chore(release): publish
   changelog v<x.y.z>` is fine).

7. **Tag the release** if not already tagged:

   ```bash
   git tag v<x.y.z>
   git push origin v<x.y.z>
   ```

8. **Post-promotion announcement** in `#releases`:

   ```text
   :white_check_mark: v<x.y.z> live at 100%. Watching for next 1h.
   Changelog: <link>
   ```

## Rollback triggers

Roll back **immediately** (do not "give it a few more minutes") if any
of these fire on canary or post-promotion:

- 5xx rate > 1% sustained for 5 minutes.
- Auth failure rate > 5% sustained for 3 minutes.
- Stripe webhook handler errors > 5 in 5 minutes.
- DB primary error log spike (>10 errors/min).
- Any data-corruption incident report.
- Sev1 customer report.

## Rollback steps

1. **Trigger rollback in Coolify** to the previous stable slice.
2. **Announce in `#releases`** (template):

   ```text
   :rotating_light: Rolling back v<x.y.z> due to <symptom>.
   IC: @<oncall>  Owner: @<handle>
   Stable target: v<x.y-1.z>
   ```

3. **Open an incident** if Sev1/Sev2: see
   [`incident-sev1`](../incident-sev1/SKILL.md) skill.
4. **Roll back migrations** *only* if the new migration is the cause
   and a `down` migration exists. Otherwise, **forward-fix** with a
   patch release.
5. **Postmortem** within 5 business days, action items in Linear.

## After promotion (T+1h, T+24h)

- T+1h: review Sentry, Grafana, PostHog dashboards. Compare with
  baseline (last week same time).
- T+24h: cost dashboard (AI burn, Stripe receipts, infra). Anything
  unexpected?
- T+24h: support volume — any spike correlated with the release?

## Pitfalls

- Promoting before migrations finish on the canary worker — wait for
  the worker to log "migrations applied".
- Not watching after promotion — most regressions surface in the
  first 30 minutes.
- Rolling back forward (i.e., shipping a hotfix on top of broken
  release) without first stabilizing — restore stable first, fix
  separately.
- Forgetting to publish the changelog — looks unprofessional and
  confuses customers.

## Done when

- Coolify shows 100% on the new release for all services.
- Sentry release health page shows ≥99.5% sessions clean for 1 hour.
- Changelog merged and visible at `/changelog`.
- Tag pushed.
- Announcement posted in `#releases`.
