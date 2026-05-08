# Observability

> **Status:** Living. Updated whenever a new SLO is introduced or a
> dashboard is added.

This directory holds the **operational observability contract** for
TESKEL. The goal is that any on-call engineer can answer three
questions in under five minutes:

1. Are we serving traffic correctly **right now**? (golden signals)
2. Is any tenant being affected disproportionately? (per-tenant)
3. What is the **error budget** consumed this window?

If a question takes longer than five minutes, the dashboard is
broken — file a story to fix it.

---

## What lives here

| Document | What it covers |
| --- | --- |
| [`slo-sli.md`](./slo-sli.md) | SLI/SLO matrix per surface (web, API, workers, AI gateway, sandbox, marketplace, payments). Error budget policy. Burn-rate alerts. |
| [`dashboards.md`](./dashboards.md) | Grafana dashboard inventory + golden signals + per-tenant drill-downs + RACI. |
| (planned) `tracing.md` | OpenTelemetry conventions, span naming, sampling. |
| (planned) `cost.md` | FinOps observability (LLM, DB, queue, storage). |

For runbooks (the *response* side), see [`docs/runbooks/`](../runbooks/).
For the threat-model side, see
[`docs/architecture/threat-model.md`](../architecture/threat-model.md).

---

## Tooling stack

| Tool | Role | Source of truth for |
| --- | --- | --- |
| **Sentry** | Error tracking | exceptions, performance issues, release health |
| **Loki + Grafana** | Logs + metrics + dashboards | structured logs, RED metrics, SLO panels |
| **Better Stack / incident.io** | Alert routing + incident timeline | alert → on-call paging, status page, incident retro |
| **PostHog** | Product analytics + feature flags | user funnels, retention, flag rollouts |
| **Langfuse** | LLM observability | per-prompt-slot trace, cost, eval results |
| **OpenTelemetry collector** | Trace shipping | spans (HTTP, DB, queue, LLM) |

Every span / log line carries: `tenant_id`, `request_id`, `user_id`
(if present), `release`, `region`. See
[`docs/architecture/multi-tenancy.md`](../architecture/multi-tenancy.md).

---

## Golden signals

For *every* externally-observable surface (web pages, public API,
worker queues, AI gateway, sandbox broker), we publish four
metrics:

1. **Latency** — p50, p95, p99 (histogram).
2. **Traffic** — rps / jobs-per-min.
3. **Errors** — rate of 5xx / failed jobs / failed LLM calls.
4. **Saturation** — queue depth, DB pool usage, sandbox slots,
   token budget consumed.

These are the *only* metrics that get woken up for at 3am. Everything
else is dashboard fodder.

---

## Naming conventions

| Item | Convention | Example |
| --- | --- | --- |
| Metric | `<domain>_<unit>_<aggregation>` | `api_request_seconds_p99`, `worker_jobs_failed_total` |
| Span | `<domain>.<verb>` | `api.handle`, `db.tx`, `llm.call`, `worker.job.run` |
| Log key | snake_case | `tenant_id`, `request_id`, `prompt_slot_id` |
| Dashboard URL slug | kebab-case | `/d/api-overview`, `/d/llm-cost` |
| Alert name | `<surface>: <signal> <severity>` | "API: error rate Sev2" |

---

## Owners

- **Platform/SRE** — owns the contract: tooling choice, ADR, retention.
- **Per-team** — owns the *content*: each domain team writes its
  dashboards + alerts + runbooks. CODEOWNERS routes review.
