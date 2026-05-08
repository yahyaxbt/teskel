---
name: gameday-drill
description: Plan and execute a controlled chaos / disaster-recovery drill — pick a scenario, communicate to staff (not customers), inject the failure, observe, recover, debrief, and capture learnings. Use quarterly minimum and before any phase that depends on operational maturity (Sec. 88+).
---

# gameday-drill — controlled chaos drill

> Plan ref: [Sec. 41 (Backup & DR)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#41-backup--disaster-recovery),
> [Sec. 75 (Failure modes)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#75-failure-modes--rollback-strategy),
> [Sec. 96 (Day-2 ops)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#96-day-2-operations-cadence).

> **Hard rule:** drills are **planned and announced internally**.
> Surprise tests of on-call run the risk of real harm to production
> if a real incident occurs simultaneously. You can vary the
> scenario; you cannot vary that the team knows a drill window is
> live.

## Why GameDays

- Verify runbooks work as written (they almost never do, first time).
- Surface tooling gaps under stress, not while shipping features.
- Train new on-calls before they get paged in anger.
- Validate RTO / RPO claims with measured numbers.
- Find dependencies you didn't know you had.

## Cadence

- **Phase 0 / 1:** none required (we don't have enough surface area).
- **Phase 2 onward:** quarterly minimum.
- **Pre-launch (Phase 4):** monthly during the 90-day pre-GA window.
- **Phase 5 (compliance):** every release-watch window includes a
  scoped drill.

## Scenario menu

Pick **one** scenario per drill. Keep blast radius contained.

| # | Scenario | Trains | Required infra |
| --- | --- | --- | --- |
| 1 | **Postgres primary failover** | DBA, SRE, on-call, app retry logic | Standby cluster |
| 2 | **WAL-G PITR shadow restore** | DBA, runbook accuracy | R2 backups |
| 3 | **Region failure** | SRE, multi-region routing | EU + US regions |
| 4 | **Vendor outage — Stripe** | Billing team, kill-switch flag | Sandbox |
| 5 | **Vendor outage — OpenRouter** | AI gateway fallback | Multiple model vendors |
| 6 | **Cache wipe (Redis flush)** | App resilience, cold-start | Staging |
| 7 | **Storage outage — R2** | Upload paths, fallback | S3 alt or staging |
| 8 | **DDoS / traffic spike 10×** | Cloudflare WAF, autoscaler | k6 + load gen |
| 9 | **Sev1 incident-response** | Comms, status page, exec call | Just process |
| 10 | **Secrets rotation in 30 min** | Vault tooling, dual-window | Doppler / 1Password |
| 11 | **Worker queue backlog 100k jobs** | BullMQ, autoscale, drain | Stage |
| 12 | **Auth provider OIDC outage** | Login fallback, SAML test | SSO partner staging |
| 13 | **Accidental DROP TABLE on staging** | Restore, scope discovery | WAL-G |
| 14 | **Loss of one AZ** | Failover topology | Coolify multi-AZ |

For new scenarios, propose via [`write-rfc`](../write-rfc/SKILL.md)
to socialize before scheduling.

## Steps

1. **Pick the scenario, owner, and date.**
   - Owner is **not** the on-call rotation — they're observers.
   - Pick a low-traffic window (e.g., Wed 14:00–17:00 UTC, off-launch).
   - Avoid release-watch windows (post-deploy 24h periods).

2. **Write the drill plan.** File:
   `docs/drills/<YYYYMMDD>-<scenario-slug>.md`. Sections:

   ```
   # GameDay <YYYY-MM-DD> — <scenario>

   - Owner: @drill-lead
   - Observers: @on-call, @sre, @cto (silent)
   - Date / window: 2026-05-14 14:00–17:00 UTC
   - Affected env: staging (default) | prod-canary | prod-full
   - Customer impact expected: none | low | scoped | high
   - Status page entry needed: yes/no
   - Rollback plan: …

   ## Hypothesis
   What we believe will happen.

   ## Failure injection
   - Method: <tool / command>
   - Command (rehearsed):
     ```
     pnpm teskel drill inject --scenario=primary-failover --env=staging
     ```

   ## Observation plan
   What we'll watch:
   - Dashboard A (link)
   - Dashboard B (link)
   - Synthetic checks (link)
   - Customer-impact proxy: ___

   ## Success criteria
   - RTO measured, ≤ <target>
   - No data loss
   - Runbook X executed end-to-end
   - <named gap> closes

   ## Abort criteria
   Stop the drill if:
   - Real customer incident overlaps.
   - Failure injection exceeds intended blast radius.
   - Observability stack itself goes down.

   ## Roles
   - Driver (executes commands): @…
   - Scribe (timestamps): @…
   - Comms (informs ops channel): @…
   ```

3. **Internal announcement.**
   - Slack `#ops-announcements`: 7 days, 1 day, and 1 hour before.
   - Calendar block on engineering shared cal.
   - On-call rotation: explicit handoff "drill window 14:00–17:00,
     real pages still take precedence; drill driver is @x".

4. **Customer announcement (if any).**
   - Default: drills run in staging or prod-canary with no customer
     impact. **No customer comms.**
   - For prod-full drills with measurable impact (rare), schedule a
     maintenance window via status page ≥7 days ahead.

5. **Pre-flight checklist (T−15 min).**
   - Observability dashboards open and pinned.
   - Drill window posted in incident channel
     `#drill-<YYYYMMDD>-<slug>`.
   - Real pager rotation acknowledged in channel.
   - Scribe has the drill plan open.

6. **Inject the failure.** Driver runs the rehearsed command. Scribe
   captures timestamp. Observers watch dashboards in silence — they
   are NOT helping, they are measuring.

7. **Observe + run the runbook.**
   - On-call (or whoever pretends to be) executes the runbook
     verbatim.
   - Scribe captures: every step, every command, every divergence,
     every "this command in the runbook doesn't work".
   - Driver does NOT prompt or coach. If on-call gets stuck for >10
     min, that's a finding — not something to rescue.

8. **Recover.** Driver performs the rollback / cleanup. Verify:
   - Production / staging back to baseline metrics.
   - No leftover state (shadow clusters torn down, flags reset,
     test rows cleaned).

9. **Hot debrief (within 30 min of recovery).** 30-minute synchronous
   meeting:
   - Walk the timeline from the scribe's notes.
   - Each observer shares one observation.
   - Capture findings in the drill plan doc under `## Findings`:
     - Runbook accuracy.
     - Tooling gaps.
     - Detection lag.
     - Communication gaps.
     - Surprise dependencies.

10. **Action items.** Each finding gets:
    - Owner.
    - Due date (default: ≤30 days).
    - Tracker link.
    - Severity (blocker / important / nice-to-have).

11. **Drill report.** Within 7 business days, the report file
    becomes the permanent record:

    ```
    ## Results
    - RTO measured: 47 min (target ≤2h) ✓
    - RPO measured: 3 min (target ≤5min) ✓
    - Runbook accuracy: 70% (3/10 commands needed correction)
    - Detection: alert fired 4 min after injection (target <2min) ✗

    ## Findings
    1. Runbook command `pg_isready -h $PRIMARY` missing -p flag …
    2. Grafana alert threshold too lax (12 min vs target <2 min) …
    3. PagerDuty escalation policy paged Tier 2 simultaneously …

    ## Action items
    | # | Action | Owner | Due | Severity |
    | 1 | Fix runbook step 4 | … | YYYY-MM-DD | blocker |
    | 2 | Tune p95 alert | … | YYYY-MM-DD | important |
    ```

12. **Follow-through.** Engineering lead sweeps action items
    weekly until closed. The next drill of the same scenario MUST
    verify the prior items are fixed.

13. **Aggregate metrics.**
    - Drill cadence (drills/quarter).
    - Coverage (% of scenarios in menu drilled in last 12 months).
    - RTO / RPO trend lines.
    - Action-item burn-down.

    Dashboard `Reliability · Drills` keeps these visible.

## Drill scenarios — quick recipes

### Postgres primary failover (Scenario 1)

```bash
# T+0: trigger failover
pnpm teskel drill inject --scenario=postgres-failover --env=staging

# Expected: standby promotes; app reconnects via DNS or PgBouncer
# update; write traffic resumes <2 min.

# T+30: rejoin original primary as standby
pnpm teskel drill recover --scenario=postgres-failover --env=staging
```

### WAL-G PITR drill (Scenario 2)

Follow [`db-restore-pitr`](../db-restore-pitr/SKILL.md) with mode
"restore drill". RTT = "yesterday at noon". Restore to shadow
cluster, verify, tear down. **Do not promote to prod.**

### Vendor outage — OpenRouter (Scenario 5)

```bash
# T+0: enable kill-switch
pnpm teskel drill inject --scenario=openrouter-outage --env=staging
# Observe: AI gateway routes to fallback model; degraded but functional.
# Customer-facing copy: "experiencing slower responses".
```

## Pitfalls

- Surprise drill — high risk of double incident; ban in production
  forever.
- Skipping the runbook and "just fixing it" — defeats the purpose.
- No abort criteria — drill spirals into a real incident.
- Action items with no owner — they rot; same gaps next drill.
- Drilling the same scenario every quarter — coverage stagnates.
  Rotate.
- Drilling only in staging — staging differs from prod; you'll be
  surprised. Plan a prod-canary drill at least once per phase.
- Treating drill as theatre — observers must be silent, drivers must
  not coach, on-call must improvise.
- No follow-through — drill becomes a calendar event, not a
  reliability practice.

## Done when

- Drill plan + report file in `docs/drills/`.
- Scribe timeline captured in real time.
- Action items in tracker with owners + due dates.
- Aggregate dashboard updated.
- Next drill scheduled (rotating scenario coverage).
