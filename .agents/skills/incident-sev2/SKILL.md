---
name: incident-sev2
description: Respond to a Sev2 incident — degraded experience but not down. Lighter than Sev1, no all-hands page, but still triaged, comms'd, and post-mortemed. Use for partial/limited customer impact, single tenant outages, single feature degradation, performance regression beyond budget.
---

# incident-sev2 — partial impact incident

> Plan ref: [Sec. 40 (On-call & runbooks)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#40-on-call--runbooks),
> [Sec. 75 (Failure modes)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#75-failure-modes--rollback-strategy),
> Companion: [`incident-sev1`](../incident-sev1/SKILL.md), [`release-hotfix`](../release-hotfix/SKILL.md).

> **Hard rule:** if customer impact escalates (more tenants, write-
> path corruption, security boundary), **upgrade to Sev1
> immediately** — don't ride Sev2 down a slope.

## Severity matrix

| Sev | Indicators | Response |
| --- | --- | --- |
| **Sev1** | Multi-tenant outage, data loss, security breach, payment broken | Page on-call + leads; war room |
| **Sev2** | One feature down, single big tenant down, perf regression breaching SLO, third-party degraded but mitigatable | Page on-call only; async triage |
| **Sev3** | Bug affecting some users, no SLO breach | Ticket; next train |
| **Sev4** | Cosmetic / single user | Ticket; backlog |

Sev2 examples:
- Search feature down for everyone but core flows work.
- Billing reconciliation cron failed for 24h (will be retried).
- p95 latency 2× over budget on one endpoint.
- One enterprise tenant unable to log in (their SSO IDP issue).
- Email deliverability bounce rate >5% but not >25%.

## Pre-conditions

- Detection: alert fired, customer report verified, or ops noticed
  metric anomaly.
- On-call has been notified.
- Severity classified (start as Sev2; upgrade to Sev1 if scope
  grows).

## Steps

1. **Open the incident channel.** Slack `#inc-<YYYYMMDD-slug>` (e.g.,
   `#inc-20260508-search-down`). Add bookmarks:
   - Status page entry (draft).
   - Grafana dashboard.
   - Sentry issue.
   - Runbook URL.

2. **Assign roles (compressed for Sev2).**
   - **Commander:** primary on-call. Drives triage and decision-
     making. Time-boxes investigation.
   - **Comms:** secondary on-call OR Commander if no traffic.
     Updates status page and customer touchpoints.
   - **Scribe:** anyone — writes timestamps in the channel.

   Sev2 does **not** require leadership, exec comms, or a war room.

3. **Initial triage (≤15 min).**
   - **Confirm impact:** how many tenants? Which features? Read
     vs write? Mitigatable by user (refresh, retry)?
   - **Recent changes:** last deploy, last flag flip, last config
     change, last vendor incident.
   - **Form a hypothesis.** Don't guess; check evidence.
   - **Post the triage summary** in the channel:
     ```
     [TRIAGE 04:55Z] Sev2 — Search returning 500s for ~3% of queries.
     Started ~04:30Z. Coincides with Elasticsearch index rebuild.
     Hypothesis: rebuild not idempotent; some shards missing aliases.
     Action: rolling back rebuild; investigating.
     ```

4. **Update status page.** Within 30 min of detection (faster for
   high-traffic features). Components:
   - **Investigating** → **Identified** → **Monitoring** → **Resolved**.
   - For Sev2, prefer the "Degraded performance" or "Partial outage"
     state, not "Major outage".

5. **Customer comms (if needed).**
   - Affected tenant(s) with named CSM/AE → CSM contacts directly,
     using template in `docs/comms/templates/sev2-affected-tenant.md`.
   - Status page handles non-named customers.
   - **Don't** send a global email blast for Sev2.

6. **Mitigate first; root-cause second.** Common Sev2 mitigations:
   - Flip kill-switch to disable broken feature.
   - Revert last deploy via canary tag.
   - Throttle the failing dependency.
   - Increase replicas if it's load-shaped.
   - Drain & retry stuck queue.

   Document each mitigation as it's applied (keeps the channel a
   timeline).

7. **Detect escalation triggers.** Upgrade to Sev1 immediately if:
   - Impact spreads to multiple tenants OR core flow.
   - Data integrity at risk (writes corrupting).
   - Security boundary breached.
   - Mitigation not landing within 60 min.
   - Escalation requested by exec / customer leadership.

   Upgrade is announced with `!sev1` in the channel, paging chain
   triggered.

8. **Resolve.** Criteria for "resolved" Sev2:
   - Error metric back within budget for ≥30 min.
   - Synthetic checks green.
   - Affected tenants confirm (if specific tenants).
   - No related runbook is still firing.

   Update status page → "Resolved". Close incident channel
   announcement (channel stays open for postmortem).

9. **Lightweight retro within 7 business days.**

   File: `docs/incidents/<YYYYMMDD>-<slug>.md`. Template:

   ```
   # Incident <YYYY-MM-DD> — <title>

   - Severity: Sev2
   - Detected: HH:MMZ via <alert / customer / dashboard>
   - Mitigated: HH:MMZ
   - Resolved: HH:MMZ
   - Duration of impact: HH:MM
   - Tenants affected: <count or "subset of users">
   - SLO impact: <yes/no>; <which SLI breached>

   ## Timeline
   HH:MM — event
   HH:MM — event

   ## Root cause
   2–4 paragraphs. Be specific about the trigger (PR, deploy, config
   change, vendor change) and the latent condition (the bug that was
   waiting to be triggered).

   ## What went well
   - Detected in N min.
   - Mitigation applied in M min.

   ## What didn't
   - Alert was misleading.
   - Runbook was wrong / missing.
   - Impact estimation was off.

   ## Action items
   | # | Action | Owner | Due | Tracker |
   |---|--------|-------|-----|---------|
   | 1 | Add regression test for X | … | YYYY-MM-DD | … |
   | 2 | Update runbook | … | YYYY-MM-DD | … |
   ```

   No "five whys" required (that's Sev1). Focus on:
   - One regression test.
   - One process / runbook improvement.
   - One detection improvement (so a similar issue is caught
     earlier next time).

10. **Action item follow-through.** Each item assigned an owner +
    due date + tracker link. Engineering lead sweeps once a week to
    ensure nothing rots.

11. **Recurrence rule.** If the same incident class recurs 2× within
    30 days, treat the next occurrence as Sev1 *regardless of
    impact size* — the recurrence itself is the elevated risk
    signal.

## What you DON'T do during a Sev2

- Don't page execs, board members, or leadership.
- Don't write postmortem during the incident; write triage notes
  only.
- Don't ship un-related changes "while we're at it".
- Don't merge a hotfix without `release-hotfix` skill flow.
- Don't close status page before mitigation lasts ≥30 min.

## Pitfalls

- Treating Sev2 as Sev3 (no comms) — affected customers feel
  ignored.
- Treating Sev2 as Sev1 (full war room) — alert fatigue, bad
  signal-to-noise for real Sev1.
- Skipping the retro because "it was a quick fix" — same incident
  recurs.
- Not capturing timestamps in real time — postmortem becomes guesswork.
- Leaving the incident channel open forever — archive after retro
  is filed.

## Done when

- Incident channel has a clear timeline.
- Status page resolved; affected tenants notified individually
  if applicable.
- Mitigation landed; metrics stable ≥30 min.
- Retro filed in `docs/incidents/`.
- Action items in tracker with owners + due dates.
- If a recurrence happens within 30 days, it's auto-elevated.
