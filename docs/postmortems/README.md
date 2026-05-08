# Postmortems

> **Status:** Living index. One entry per postmortem.
> **How to author:** see [`write-postmortem` SKILL](../../.agents/skills/write-postmortem/SKILL.md).

This directory holds blameless postmortems for every Sev1 incident
and any Sev2 / near-miss the on-call team chose to write up.

A Sev1 with no postmortem within **10 business days** is itself a
Sev2 process incident.

## Conventions

- File name: `<YYYY-MM-DD>-<slug>.md` (UTC date the incident closed).
- Slug: short, kebab-case, descriptive
  (e.g. `stripe-webhook-replay`, `pgvector-index-rebuild-stall`).
- Newest entries go at the top of the index below.
- The author opens a PR with the doc; the index entry is added in
  the same PR.
- Once an action item closes, mark it done **in the doc** and link
  the closing PR.
- Customer-identifying details are redacted.

## Template

A starter template lives at
[`0000-template.md`](./0000-template.md) (created with the first
postmortem). Copy → rename → fill in.

## Index

<!-- Newest first. One line per postmortem.
     Format: `- YYYY-MM-DD — Sev<N> — <surface> — [<title>](<file>.md) — owner: @x`
-->

_No postmortems yet. The first incident closes here._

## Quarterly review

Once a quarter, security + platform leads tally root-cause
categories across this index, look for clusters, and publish a
one-page **Reliability Review** that feeds back into:

- [`docs/observability/slo-sli.md`](../observability/slo-sli.md) — SLO updates.
- `gameday-drill` skill — new scenarios.
- ADRs / RFCs — structural fixes.

See `write-postmortem` SKILL §12 for the cadence.
