# ADR-0000: <Decision Title>

> Copy this file to `docs/adr/NNNN-<short-slug>.md` (zero-padded, next
> available number) and fill it in. Keep it terse — ADRs are read
> repeatedly; brevity is a feature.
>
> See `AGENTS.md` §15 for when an ADR is required vs an RFC, and the
> mandatory companion plan-update PR.

| | |
| --- | --- |
| Status | `proposed` / `accepted` / `superseded by ADR-NNNN` / `deprecated` |
| Date | YYYY-MM-DD |
| Deciders | @handle1, @handle2 |
| Plan section affected | Sec. NN (e.g., Sec. 28 Queues) |
| Related ADRs | ADR-NNNN, ADR-NNNN |
| Related RFC | docs/rfc/NNNN-... (if applicable) |

## Context

What is the problem we are deciding about? Why now? Include any
constraints (regulatory, performance, cost, vendor) that bound the
solution space. Link to the relevant plan section so the reader can
read the broader context.

Keep this section to ≤300 words. If it grows beyond that, the
decision is probably an RFC, not an ADR.

## Options Considered

For each option, capture: name, summary, pros, cons, rough cost.

### Option A — <name>

- Summary: …
- Pros:
  - …
- Cons:
  - …
- Cost / risk: …

### Option B — <name>

- Summary: …
- Pros:
  - …
- Cons:
  - …
- Cost / risk: …

### Option C — Status quo (do nothing)

- What breaks if we don't decide? What's the cost of inaction?

## Decision

We will choose **Option <X>** because …

State the decision in one or two declarative sentences. Avoid hedging.

## Consequences

### Positive

- …

### Negative / Tradeoffs

- …

### Risks & Mitigations

| Risk | Likelihood | Mitigation |
| --- | --- | --- |
| … | low/med/high | … |

### Revisit when

- A specific event triggers re-evaluation (e.g., "if monthly active
  workflows exceed 10M", "if Better Auth ends MIT licensing", "after
  Q2 2026 cost review").

## Implementation Notes

- Migration plan (if any).
- Affected packages / files.
- Telemetry / dashboards to add.
- Documentation updates required (link to PR).

## References

- Plan section.
- Vendor docs.
- Comparable ADRs from other projects (cite if you cribbed).
- Benchmarks, blog posts, case studies.
