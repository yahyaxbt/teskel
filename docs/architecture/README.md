# Architecture

This directory holds **system-level architecture documentation**.
It is a *reading-room*, not a workshop — every document here describes
how TESKEL is *meant* to be built.

If you want to **change** an architectural decision, you do **not**
edit these docs first. You write a [`docs/rfc/`](../rfc/) →
[`docs/adr/`](../adr/) → only then update the architecture doc in the
**same PR** that lands the ADR.

If you want to **explain** a decision more deeply, write a new
document under this directory and link it from both the relevant ADR
and the master plan.

---

## What lives here

| Document | What it covers | Owner |
| --- | --- | --- |
| [`c4.md`](./c4.md) | C4-Light diagrams: System Context → Container → Component for the four bounded contexts (Web, Workflow, Marketplace, Identity). | Architecture |
| [`multi-tenancy.md`](./multi-tenancy.md) | Tenant isolation strategy. RLS, `db.withTenant`, signed URLs, queue routing, cross-tenant test matrix. **Required reading** for anyone adding a table or API route. | Architecture |
| [`threat-model.md`](./threat-model.md) | STRIDE-based threat model per surface (web, API, workers, AI gateway, marketplace, sandbox). Required reading for anyone touching auth, RLS, or sandbox. | Security |
| (planned) `data-flow.md` | High-level data flow + PII boundaries + cross-region replication. | Architecture |
| (planned) `ai-runtime.md` | Prompt registry, model routing, eval harness, Langfuse tracing wiring. | AI |
| (planned) `event-bus.md` | Outbox, DLQ, replay, idempotency keys end-to-end. | Backend |

---

## Source of truth ranking

When in doubt, read in this order — earlier wins:

1. [`AGENTS.md`](../../AGENTS.md) — hard rules, settled stack.
2. [`TESKEL_FULLSTACK_BUILD_BREAKDOWN.md`](../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md)
   — master plan; treat as immutable except via ADR.
3. [`docs/adr/`](../adr/) — accepted architecture decisions.
4. This directory (`docs/architecture/`) — explanatory how things fit
   together. Loses to ADRs if there is a contradiction (open a PR to
   reconcile).
5. Code comments and READMEs in `packages/` — implementation details.

---

## Conventions

- One concept per file.
- 200–800 lines, prose + diagrams + tables.
- Diagrams are **Mermaid** (renders on GitHub) or ASCII boxes — no
  proprietary formats checked in. PNG/SVG are tolerated only when
  Mermaid cannot express the diagram.
- Every doc starts with a *Status* block (`Stable | Draft | Living`).
- Every doc lists the **ADRs** that govern it.
- Cross-link liberally to plan sections, skills, runbooks.
- When you change a hard rule, update the doc *and* AGENTS.md *and*
  the relevant ADR in the same PR.

---

## Don't

- Don't add API references / SDK references here — those go in
  [`docs/api/`](../api/) and `apps/docs-site/` (when it exists).
- Don't add operational runbooks here — those go in
  [`docs/runbooks/`](../runbooks/).
- Don't add UX or product copy here — that's not architecture.
- Don't paste large code blocks; reference the file path instead.
