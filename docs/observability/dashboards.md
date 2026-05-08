# Dashboards inventory

> **Status:** Living. Updated whenever a dashboard is added or
> retired.
>
> **Governed by:** Plan §39 (SLO), §41 (on-call & runbooks).

This file lists every Grafana dashboard the org maintains. The goal
is that an on-call engineer can find the right dashboard by name in
under 30 seconds.

If you create a dashboard that isn't here, it doesn't exist. Either
add it here (in the same PR that adds the JSON) or delete it.

---

## 1. Conventions

- **URL slug:** kebab-case, prefixed by surface — `/d/api-overview`,
  `/d/llm-cost`, `/d/db-postgres`.
- **Title:** `Surface — Aspect`, e.g. `API — Overview`,
  `Workers — Job latency`, `LLM — Cost & burn`.
- **Owner:** name of the team in the title bar (Frontend / Backend /
  AI / Platform / Marketplace / Identity / Finance).
- **Time range:** dashboards default to **last 24h** for ops, **last 7d**
  for reviews; never a hardcoded absolute range.
- **Variables:** every dashboard with per-tenant data has a `tenant`
  variable (default: all). Region and release are also variables
  where relevant.
- **Source:** dashboard JSON lives at
  `infra/grafana/dashboards/<surface>/<slug>.json` and is provisioned
  by the Grafana operator. **No hand-editing in production.**

---

## 2. The five every on-call must know

These five exist on day one. If any of these is broken, fix before
shipping anything else.

| Dashboard | URL slug | What it answers |
| --- | --- | --- |
| **Platform — Overview** | `/d/platform-overview` | Are we up? Single pane of glass: availability, error rate, p99 across all surfaces, error budget burn. |
| **API — Golden signals** | `/d/api-golden` | Latency / traffic / errors / saturation per route. |
| **Workers — Queue health** | `/d/workers-queue` | Queue depth, oldest job, retry rate, DLQ size by job kind. |
| **DB — Postgres** | `/d/db-postgres` | CPU, IOPS, replication lag, slow queries, index usage, locks. |
| **LLM — Cost & burn** | `/d/llm-cost` | Spend per provider, per slot, per tenant; budget consumed; eval pass rate. |

Each links to the matching runbook in [`docs/runbooks/`](../runbooks/).

---

## 3. Per-surface dashboards

### Web

| Dashboard | URL slug | Owner | Notes |
| --- | --- | --- | --- |
| Web — RUM | `/d/web-rum` | Frontend | LCP, INP, CLS, TTFB by route + device. PostHog real-user data. |
| Web — Server-render | `/d/web-rsc` | Frontend | RSC TTFB, cache hit rate, hydration errors. |
| Web — Funnel (signup) | `/d/web-funnel-signup` | Growth | PostHog funnel: landing → signup → first workflow. |

### API

| Dashboard | URL slug | Owner | Notes |
| --- | --- | --- | --- |
| API — Golden signals | `/d/api-golden` | Backend | per-route p50/p95/p99, errors, traffic. |
| API — Auth | `/d/api-auth` | Identity | login success / 2FA / session refresh / token revocation. |
| API — Idempotency | `/d/api-idempotency` | Backend | replay rate, key collisions (must be 0), key TTL. |
| API — Rate limits | `/d/api-ratelimit` | Backend | per-tenant + per-IP buckets, throttle events. |

### Workers / queue

| Dashboard | URL slug | Owner | Notes |
| --- | --- | --- | --- |
| Workers — Queue health | `/d/workers-queue` | Backend | depth / age / retry / DLQ per job kind. |
| Workers — Outbox | `/d/workers-outbox` | Backend | outbox lag, sender retry, dead-letter outbox. |
| Workers — Backfills | `/d/workers-backfills` | Backend | per-backfill progress, rate, errors (per `data-backfill-job`). |

### AI

| Dashboard | URL slug | Owner | Notes |
| --- | --- | --- | --- |
| LLM — Cost & burn | `/d/llm-cost` | AI | $/min, $/tenant, $/slot, budget consumed, projected EOM. |
| LLM — Latency | `/d/llm-latency` | AI | provider p50/p95/p99 (Claude, GPT, Llama, Gemini), gateway overhead. |
| LLM — Eval pass rate | `/d/llm-eval` | AI | per-slot Promptfoo pass rate over time + per release. |
| LLM — Errors | `/d/llm-errors` | AI | error rate per provider; circuit-breaker state; fallback hits. |

### Sandbox

| Dashboard | URL slug | Owner | Notes |
| --- | --- | --- | --- |
| Sandbox — Pool | `/d/sandbox-pool` | Platform | active sandboxes, pool capacity, admission p95, abandoned. |
| Sandbox — Policy | `/d/sandbox-policy` | Platform | egress denials, syscall denials, time-cap kills, OOM kills. |

### Database & cache

| Dashboard | URL slug | Owner | Notes |
| --- | --- | --- | --- |
| DB — Postgres | `/d/db-postgres` | Platform | CPU/IOPS/replication, top queries, locks, index usage. |
| DB — RLS sanity | `/d/db-rls` | Platform | count of `withTenant` calls, missing-context guard hits (should be 0 in prod). |
| DB — pgvector | `/d/db-pgvector` | AI | embedding count, kNN p95, index size, recall. |
| Cache — Redis | `/d/cache-redis` | Backend | memory, evictions, hit rate, slow log. |

### Marketplace

| Dashboard | URL slug | Owner | Notes |
| --- | --- | --- | --- |
| Marketplace — Listings | `/d/marketplace-listings` | Marketplace | impressions, clicks, install conversion, refunds. |
| Marketplace — Revenue | `/d/marketplace-revenue` | Finance | gross / net / payout, takedowns, chargebacks. |
| Marketplace — Moderation | `/d/marketplace-moderation` | Marketplace | queue depth, p95 review time, false-positive rate. |

### Payments

| Dashboard | URL slug | Owner | Notes |
| --- | --- | --- | --- |
| Payments — Webhook | `/d/payments-webhook` | Backend | ingest success, age, dedup hits, replay. |
| Payments — Reconciliation | `/d/payments-reconcile` | Finance | invoice mirror lag, payout drift, mismatch report. |

### Storage

| Dashboard | URL slug | Owner | Notes |
| --- | --- | --- | --- |
| Storage — R2 | `/d/storage-r2` | Backend | bytes, object count, error rate, signed-URL p99. |

### Identity

| Dashboard | URL slug | Owner | Notes |
| --- | --- | --- | --- |
| Identity — Auth events | `/d/identity-auth` | Identity | login / signup / passkey / SAML rate, errors. |
| Identity — RBAC | `/d/identity-rbac` | Identity | permission denies, role assignment events. |
| Identity — Audit log | `/d/identity-audit` | Identity | volume, hash-chain validation, export status. |

### Cost / FinOps

| Dashboard | URL slug | Owner | Notes |
| --- | --- | --- | --- |
| Cost — Compute | `/d/cost-compute` | Finance | $/region/service; YoY; budget. |
| Cost — Egress | `/d/cost-egress` | Finance | egress GB by destination, top tenants, anomaly. |
| Cost — Per tenant | `/d/cost-per-tenant` | Finance | gross-margin contribution per tenant; outliers. |

---

## 4. Alert linkage

Every alert defined in `infra/grafana/alerts/` MUST link to:

1. The runbook at `docs/runbooks/<surface>/<slug>.md`.
2. The dashboard URL (one of the slugs above).
3. The relevant SLO row in [`slo-sli.md`](./slo-sli.md).

If any of those three links is missing, the alert is rejected at PR
review.

---

## 5. Dashboard quality checklist

When you author a new dashboard, verify:

- [ ] Title matches `Surface — Aspect`.
- [ ] Owner team in the description.
- [ ] Time range set to a relative window, not absolute.
- [ ] `tenant`, `region`, `release` variables wired.
- [ ] Each panel has a unit (`s`, `req/s`, `bytes`, `%`).
- [ ] Each panel has a description with the metric name.
- [ ] Color thresholds match SLO (green ≤ SLO, yellow → red).
- [ ] At least one annotation per release (overlay deployment events).
- [ ] No hard-coded tenant ID, no hard-coded user ID.
- [ ] JSON committed at `infra/grafana/dashboards/<surface>/<slug>.json`.
- [ ] Linked from a runbook **and** this file.
- [ ] CODEOWNER review by Platform/SRE.

---

## 6. Retired dashboards

Retire (don't delete) by moving the JSON to `infra/grafana/retired/`
and removing the slug from this file with a one-line note.

| Slug | Retired | Reason |
| --- | --- | --- |
| (none yet) |  |  |

---

## 7. References

- [`slo-sli.md`](./slo-sli.md) — SLOs each dashboard should reflect.
- [`docs/runbooks/`](../runbooks/) — runbook surface.
- `add-runbook` skill.
- Plan §39, §41.
