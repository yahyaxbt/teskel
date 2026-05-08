---
name: write-postmortem
description: Author a blameless postmortem after a Sev1 / Sev2 incident — captures timeline, contributing factors, action items, and learnings without scapegoating. Required for every Sev1; optional but recommended for Sev2.
---

# Skill — Write a postmortem

> **Plan refs:** §41 (on-call & runbooks), §44 (continuous improvement).
> **Related skills:** `incident-sev1`, `incident-sev2`,
> `gameday-drill`, `add-runbook`, `release-hotfix`.
> **Hard rules:** [`AGENTS.md` §17](../../../AGENTS.md#17-escalation-triggers).

A postmortem is the **artifact** that converts an incident into a
durable improvement to the platform. It is **blameless** —
people made the best decisions they could with the information they
had — and **specific** — every action item is owned and dated.

If a postmortem reads as "we'll be more careful next time", it
failed. The point is **to remove a class of failure**, not to scold.

---

## When to write one

| Trigger | Required? | Window |
| --- | --- | --- |
| Sev1 closed | **Required** | Draft within 5 business days, finalized within 10. |
| Sev2 closed | Recommended; required if same class within 30 d | Draft within 7 business days. |
| Near-miss caught by automation (auto-rollback, budget brake) | Optional | Lightweight one-pager. |
| GameDay finding | Optional | Folded into drill report (see `gameday-drill`). |
| Customer-reported issue that *almost* breached SLA | Recommended | One-pager. |

A Sev1 with no postmortem within 10 business days is itself a Sev2
process incident.

---

## Pre-conditions

- The incident is **closed** (status page green; on-call stood
  down).
- The incident-channel transcript and the incident timeline are
  preserved (Better Stack / incident.io export, or chat export).
- Linked artifacts collected: PRs, dashboards, runbooks invoked,
  error budgets affected, audit-log entries.
- An **author** is assigned (typically the incident commander, but
  may be delegated). The author owns the document until it lands.

---

## Steps

### 1. Open the doc

Create `docs/postmortems/<YYYY-MM-DD>-<slug>.md` from the template
(see §3 below). Slug is one short phrase, kebab-case
(`stripe-webhook-replay`, `pgvector-index-rebuild-stall`).

The `docs/postmortems/` directory and its `README.md` index are
**pre-created** in the repo (see [`docs/postmortems/README.md`](../../../docs/postmortems/README.md)).
After landing the postmortem PR, add a one-line entry to that
index — newest first — pointing to the new doc.

### 2. Reconstruct the timeline

Pull every dated event into chronological order with timestamps in
**UTC**. Sources:

- Better Stack / incident.io.
- `#incidents` channel scrollback.
- Sentry release health.
- Loki + Grafana panels at the incident's time range.
- Audit log entries.
- Customer-facing status page transitions.

A timeline entry is one line: `2026-05-08 09:14 UTC — <event>`.
Skip noise ("@x acknowledged" without effect); keep decisions and
state transitions.

### 3. Diagnose contributing factors

Use the **5 Whys** but stop when "why" turns into "who". The aim is
*system* causes, not *human* causes.

For each contributing factor, classify:

- **Trigger** — the proximate event that started the incident.
- **Latent fault** — a pre-existing weakness that was waiting.
- **Detection gap** — why we didn't notice sooner.
- **Mitigation gap** — why the response wasn't faster.
- **Recovery gap** — why fixing was hard.

Most incidents have at least one of each. Don't force-fit; if a
category doesn't apply, write "n/a" and move on.

### 4. Quantify customer impact

| Field | Notes |
| --- | --- |
| Affected tenants | Count + list of major ones (anonymized if public). |
| Surface | API / web / workers / specific feature. |
| Duration of degraded service | From first measurable symptom to full recovery. |
| SLO impact | Error budget consumed (per [`docs/observability/slo-sli.md`](../../docs/observability/slo-sli.md)). |
| Data loss / corruption | None / per-tenant / region-wide. Required field. |
| Customer comms | Status page + emails sent. |
| Compensation issued | Any service credits or refunds. |

If data loss is non-zero, the postmortem **must** include a recovery
audit trail (per `db-restore-pitr` if applicable).

### 5. Decide what would have prevented this

Two passes:

1. **Make this exact incident impossible.** What invariant, if it
   had held, would have averted this?
2. **Make this *class* impossible.** What broader pattern was the
   incident an example of?

The second pass is where leverage lives. A postmortem that ships
only first-pass action items keeps re-running the same incident
with new triggers.

### 6. Action items — concrete, owned, dated

Every action item has all four:

| Field | Example |
| --- | --- |
| Owner | `@backend-lead` |
| Description | "Add `Idempotency-Key` enforcement to `/v1/payouts`." |
| Due date | "2026-05-22" (≤ 30 days unless explicitly justified). |
| Tracking link | Linear / GitHub story (`P3-091`). |

Categories:

- **Prevent** — code/policy change that removes the latent fault.
- **Detect** — alert / dashboard / SLO that would have surfaced it
  sooner.
- **Mitigate** — automation that limits blast radius next time.
- **Recover** — runbook / tooling change that speeds resolution.
- **Drill** — gameday scenario added (see `gameday-drill`).

Avoid:

- "Investigate." (Not actionable; either schedule the investigation
  or omit.)
- "Be more careful." (Not an action item.)
- Items longer than 30 days without a justification block.

### 7. Identify what went **right**

Most teams skip this and lose institutional knowledge. List 3–5
things that worked: alert fired in time, runbook X said exactly
the right thing, customer comms went out within 5 min, on-call
secondary picked up cleanly. **Reinforce** these by name —
investments in those areas paid off.

### 8. Open the PR

Branch: `postmortem/<YYYY-MM-DD>-<slug>`. PR title:
`postmortem: <title>` (no leading word like `Add` — the prefix
*is* the type).

The PR description should include:

- Severity, dates, surface.
- Customer impact one-liner.
- Link to the doc in this PR.
- Tagged reviewers: incident commander, surface owner,
  Platform/SRE, Identity/Security if security-relevant, plus the
  `#leadership` review line for Sev1.

### 9. Review

Reviewers verify:

- Timeline is coherent (no gaps > 15 min unaccounted for).
- Contributing factors are *system* not *human*.
- Action items meet the four-field rule.
- "What went right" section is non-empty.
- No customer PII in the doc.

Comments are **on the document**, not on the people. Maintainers
moderate to keep tone blameless.

### 10. Land + announce

When merged:

- Post a summary in `#engineering` and `#incidents`.
- Cross-link from the originating Linear / Jira incident ticket.
- Subscribe action-item owners to a follow-up thread; status
  reviewed weekly until all are closed.
- Add the postmortem to the **next monthly retro** agenda.

### 11. Track action items to closure

Action items live as stories per [`docs/stories/`](../../docs/stories/).
Postmortem doc remains the "case file" — when an action item is
done, mark it done in the doc and link the closing PR.

A postmortem with **action items overdue by > 30 days** triggers a
second-look meeting; the system is telling you the work was
mis-estimated or de-prioritized.

### 12. Aggregate quarterly

Once a quarter, security + platform leads:

- Tally root-cause categories (auth, RLS, queue, LLM, vendor,
  config drift, etc.).
- Look for clusters; one cluster = one ADR-or-RFC level fix.
- Publish a one-page **Reliability Review** summarizing trends.
- Adjust `docs/observability/slo-sli.md` and gameday scenarios.

---

## Template (copy into `docs/postmortems/0000-template.md` once)

```markdown
# Postmortem — <title>

- **Status:** Draft | Final
- **Severity:** Sev1 | Sev2
- **Surface:** API | Web | Workers | AI gateway | …
- **Date(s):** YYYY-MM-DD (UTC)
- **Authors:** @x, @y
- **Incident commander:** @z
- **Reviewers:** @platform, @security, @<surface>

## Summary
One paragraph: what happened, who was affected, for how long, what
the impact was.

## Customer impact
| Field | Value |
| --- | --- |
| Affected tenants | … |
| Surface | … |
| Duration | … |
| SLO impact | … (error budget consumed) |
| Data loss | None / per-tenant / region-wide |
| Comms sent | status page, emails |
| Compensation | service credits, refunds |

## Timeline (UTC)
- HH:MM — …
- HH:MM — …

## Contributing factors
### Trigger
…
### Latent fault
…
### Detection gap
…
### Mitigation gap
…
### Recovery gap
…

## What went right
- …

## What went wrong
- …

## Action items
| Owner | Description | Category | Due | Tracking |
| --- | --- | --- | --- | --- |
| @x | … | Prevent | YYYY-MM-DD | P0-001 |

## Lessons
- …

## References
- Incident ticket
- Dashboards (links)
- PRs (the bug, the fix, the canary, the rollback)
- Runbooks invoked
- Related ADRs / RFCs
```

---

## Pitfalls

- **"Tone-policing" the engineer.** A postmortem describes the
  system; if a reviewer reads "person X did Y", treat it as a doc
  bug and re-write.
- **Ten action items, one date.** If everything is due by the same
  date, none are.
- **Postmortem of a postmortem.** Don't write a meta-postmortem
  about the postmortem process; the right artifact is an ADR.
- **Burying the customer impact** in technical detail. Customer
  impact is the headline.
- **Linking to chat scrollback only.** Chat is volatile. Quote the
  decisive lines into the doc.
- **Open-ended "investigate" items.** Convert into bounded stories
  ("re-test PITR scenario X by date") or drop them.
- **Skipping "what went right".** You'll miss reinforcement
  opportunities and demoralize the team.

---

## Done when

- Doc lives at `docs/postmortems/<date>-<slug>.md`.
- All action items have owner + due date + tracking link.
- "What went right" has ≥ 3 items.
- Reviewers (incident commander + surface owner + Platform/SRE)
  approved.
- Linked from the originating incident ticket.
- Posted to `#engineering`.
- For Sev1: discussed in next monthly retro.

---

## References

- [`AGENTS.md` §17](../../../AGENTS.md#17-escalation-triggers).
- [`docs/observability/slo-sli.md`](../../docs/observability/slo-sli.md).
- `incident-sev1`, `incident-sev2`, `gameday-drill` skills.
- Plan §41, §44.
