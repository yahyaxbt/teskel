# SLI / SLO matrix

> **Status:** Stable. Reviewed quarterly + after every Sev1 / Sev2.
>
> **Governed by:** Plan §39 (SLI/SLO/SLA), §40 (error budget),
> §41 (on-call & runbooks).

This is the source-of-truth for **what counts as good** at TESKEL.
If an SLO is not listed here, it is **not** an SLO — it is just
a metric.

A new SLO requires a PR to this file, an alert rule, a runbook, and
a CODEOWNER review. Removing or relaxing an SLO requires an ADR.

---

## 1. Vocabulary

- **SLI** (Service Level Indicator) — a number you can compute, e.g.
  `(successful HTTP responses with status < 500) / (total HTTP responses)`.
- **SLO** (Service Level Objective) — a target for an SLI over a time
  window, e.g. `99.9% over 30 days`.
- **Error budget** — `(1 - SLO) × window_traffic`. Burn rate = how
  fast the budget is consumed.
- **SLA** (Service Level Agreement) — what we **commit** to customers
  contractually. SLA ≤ SLO (with margin).

---

## 2. Surfaces

| # | Surface | Owner | SLOs in this matrix |
| --- | --- | --- | --- |
| 1 | Web app (anonymous) | Frontend | Availability, p75 LCP |
| 2 | Web app (authenticated) | Frontend | Availability, p75 INP, RSC TTFB |
| 3 | Public API | Backend | Availability, p99 latency, idempotency |
| 4 | Workers / queue | Backend | Job success rate, p95 end-to-end latency, DLQ size |
| 5 | AI gateway | AI | Availability, p95 latency (excluding LLM time), cost ceiling |
| 6 | Sandbox broker | Platform | Job admission latency, abort-on-policy fidelity |
| 7 | Marketplace | Marketplace | Listing render p95, install success rate |
| 8 | Payments (Stripe) | Backend | Webhook ingest success, payout reconciliation |
| 9 | Auth / sessions | Identity | Login success rate, MFA challenge success |
| 10 | Storage (R2) | Backend | Object availability, signed-URL latency |

---

## 3. SLI / SLO matrix

### 3.1 Web — anonymous (`apps/web`, public routes)

| SLI | Definition | SLO | Window | Alert when |
| --- | --- | --- | --- | --- |
| Availability | Fraction of HTTP responses with status `< 500` | **99.95%** | 30 d | 1h burn ≥ 14×, 6h burn ≥ 6× |
| Page load (LCP) | p75 LCP measured by PostHog (real-user) | **≤ 2.5 s** | 7 d rolling | p75 > 3.0 s for 30 min |
| First byte | p95 RSC TTFB | ≤ 600 ms | 7 d rolling | p95 > 1.0 s for 30 min |

### 3.2 Web — authenticated (`apps/web`, app shell)

| SLI | Definition | SLO | Window | Alert when |
| --- | --- | --- | --- | --- |
| Availability | Fraction of HTTP responses with status `< 500` | **99.9%** | 30 d | burn rates as §3.1 |
| Interactivity (INP) | p75 INP per session | **≤ 200 ms** | 7 d rolling | p75 > 300 ms for 30 min |
| RSC TTFB | p95 of authenticated RSC route TTFB | ≤ 800 ms | 7 d rolling | p95 > 1.5 s for 30 min |

### 3.3 Public API (`apps/api`)

| SLI | Definition | SLO | Window | Alert when |
| --- | --- | --- | --- | --- |
| Availability | Fraction of `2xx + 3xx + 4xx` responses (5xx and timeouts are bad) | **99.95%** | 30 d | 1h burn ≥ 14× |
| Latency p99 | p99 of `request_duration_seconds` excluding 4xx | **≤ 500 ms** | 7 d rolling | p99 > 1.0 s for 15 min |
| Idempotency hit rate | `(idempotency_replays_served / total_idempotent_requests)` is *informational*; failure mode = key collision producing wrong response | **0** wrong-response replays | always | any single occurrence pages on-call |
| Auth success rate | `(login_success / login_attempts)` excluding wrong-password | **≥ 99.5%** | 7 d | rate < 99% for 30 min |

### 3.4 Workers / queue

| SLI | Definition | SLO | Window | Alert when |
| --- | --- | --- | --- | --- |
| Job success rate | `(jobs_succeeded / jobs_attempted)` excluding deliberate aborts | **≥ 99.5%** | 30 d | rate < 99% for 1 h |
| End-to-end p95 | enqueue→complete p95 by job kind | per-job (see below) | 7 d | p95 > 1.5× SLO for 30 min |
| DLQ size | dead-letter queue depth | **≤ 100** items always | live | depth ≥ 100, growing |
| Backlog age | oldest job in main queue | **≤ 5 min** | live | age ≥ 5 min for 5 min |

Per-job p95 SLOs:

| Job | SLO p95 |
| --- | --- |
| `email.send` | 30 s |
| `workflow.step.run` | 60 s (excluding sleeps) |
| `template.install` | 90 s |
| `export.bundle` | 5 min |
| `data.backfill.batch` | 5 min |
| `webhook.process` | 30 s |

### 3.5 AI gateway

| SLI | Definition | SLO | Window | Alert when |
| --- | --- | --- | --- | --- |
| Availability | non-5xx responses from gateway service (excluding upstream LLM 5xx) | **99.9%** | 30 d | 1h burn ≥ 14× |
| Gateway overhead p95 | p95 added latency by gateway over raw provider call | **≤ 50 ms** | 7 d | p95 > 100 ms for 30 min |
| Cost ceiling | per-tenant LLM spend / month | **≤ 100% of quota** | live | tenant spend > 80% of quota; auto-throttle at 100% |
| Eval pass rate | per-slot Promptfoo eval `pass_rate` post-deploy | **≥ slot threshold (default 90%)** | per release | drop > 5% vs prior version |

### 3.6 Sandbox broker

| SLI | Definition | SLO | Window | Alert when |
| --- | --- | --- | --- | --- |
| Admission latency | p95 time from "request sandbox" to "ready" | **≤ 3 s** | 7 d | p95 > 5 s for 15 min |
| Policy fidelity | fraction of sandbox jobs that hit a policy violation event we *failed* to abort | **0** | always | any single occurrence pages on-call |
| Concurrency utilization | active sandboxes / pool capacity | **≤ 80%** sustained | live | utilization > 90% for 10 min |

### 3.7 Marketplace (Phase 3+)

| SLI | Definition | SLO | Window | Alert when |
| --- | --- | --- | --- | --- |
| Listing render p95 | server render p95 of listing detail | ≤ 800 ms | 7 d | p95 > 1.5 s for 30 min |
| Install success rate | `(installs_completed / installs_started)` | **≥ 99%** | 7 d | rate < 97% for 1 h |
| Search availability | search returns within 2 s | ≥ 99.5% | 7 d | rate < 99% for 1 h |

### 3.8 Payments (Stripe)

| SLI | Definition | SLO | Window | Alert when |
| --- | --- | --- | --- | --- |
| Webhook ingest success | non-error response to Stripe within 5 s | **≥ 99.9%** | 30 d | rate < 99.5% for 30 min |
| Reconciliation lag | time between Stripe event and DB invoice mirror | ≤ 5 min p95 | 7 d | p95 > 15 min for 30 min |
| Payout reconciliation drift | sum(payouts) − sum(creator_balances) | **0** at 24h | daily | any non-zero drift at daily close pages on-call |

### 3.9 Auth / sessions

| SLI | Definition | SLO | Window | Alert when |
| --- | --- | --- | --- | --- |
| Login success rate (legitimate) | `(login_success / (login_success + login_5xx))` | **≥ 99.95%** | 7 d | rate < 99.9% for 30 min |
| 2FA challenge success | `(2fa_success / 2fa_attempts)` | ≥ 99.5% | 7 d | rate < 99% for 1 h |
| Session refresh availability | non-5xx on session refresh | **≥ 99.99%** | 30 d | 1h burn ≥ 14× |

### 3.10 Storage (R2)

| SLI | Definition | SLO | Window | Alert when |
| --- | --- | --- | --- | --- |
| Object availability | non-5xx on `GET` of an existing key | **≥ 99.9%** | 30 d | rate < 99.5% for 30 min |
| Signed-URL latency | p99 of helper to mint a signed URL | ≤ 50 ms | 7 d | p99 > 200 ms for 30 min |

---

## 4. SLAs (customer-facing) vs SLOs (internal)

We publish SLAs only for paid plans. Internal SLOs are tighter to
preserve a buffer.

| Surface | Internal SLO | Public SLA (Business+) | Public SLA (Enterprise) |
| --- | --- | --- | --- |
| Web auth + API | 99.9% / 99.95% | **99.5%** | **99.9%** |
| Workers (workflow) | 99.5% (success rate) | none | **99.5%** |
| AI gateway | 99.9% | none | **99.5%** |
| Marketplace install | 99% | n/a | n/a |
| Payments webhook | 99.9% | none (best effort) | **99.9%** |

SLA breach → **service credit** per public ToS. SLO breach → internal
incident retro and possible feature freeze (see §6).

---

## 5. Burn-rate alerts (Google SRE-style)

Two-window alert: a fast burn for symptom detection, a slow burn for
budget protection.

| Window | Threshold (multi-burn) | Page who | Severity |
| --- | --- | --- | --- |
| 1h | burn ≥ 14× over 5m **AND** 1h | on-call | Sev2 |
| 6h | burn ≥ 6× over 30m **AND** 6h | on-call | Sev2 (escalates to Sev1 if 1h also fires) |
| 3d | burn ≥ 1× over 6h **AND** 3d | team channel only | informational |

Pages route via Better Stack → primary on-call → secondary if
unacked in 5 min.

---

## 6. Error budget policy

For each 30-day window, each tier has an error budget = `1 − SLO`
applied to actual traffic.

| Budget remaining | Posture |
| --- | --- |
| **> 50%** | Normal velocity; ship features. |
| **20–50%** | Yellow; review change-risk in standup; bias toward smaller PRs. |
| **5–20%** | Orange; release-train cadence frozen for non-critical changes. Hotfixes only. |
| **< 5%** | Red; **feature freeze** for the affected surface. All work redirected to reliability until the next window resets. ADR required to override. |
| **Exhausted** | Auto-rollback latest risky deploy if any. SRE owns the next 7 days. |

Budget consumption dashboard: `/d/error-budget` (planned).

---

## 7. Adding / changing an SLO

1. Open a story per [`docs/stories/`](../stories/).
2. Open a PR to this file with the new row.
3. Add the alert rule to the canonical alerts location for this
   repo. Until the observability bootstrap PR (Phase 0 W2) lands,
   that location is **TBD**; once it lands, this section will pin
   the exact path (proposed: `infra/grafana/alerts/<surface>.yml`,
   confirmed by ADR-0014). Cross-reference the bootstrap PR in your
   own PR description so reviewers can sanity-check the path.
4. Add or update the runbook under `docs/runbooks/<surface>/`
   (per-surface runbook trees land with the same observability
   bootstrap PR; until then, file the runbook content inline in the
   PR and the bootstrap PR will move it to its final home).
5. Bump the `Last reviewed` date below.
6. CODEOWNER review (per surface owner) + Platform/SRE.

Removing or *relaxing* an SLO is an **ADR**, not a PR. Tightening an
SLO is a PR.

---

## 8. Update protocol

| When | Action |
| --- | --- |
| Quarterly | Full review per Plan §39. |
| Post-Sev1 / Sev2 | Inspect whether the SLO matched lived experience; adjust if not. |
| New surface ships | Add SLOs **before** the surface is GA — at minimum availability + latency + saturation. |
| New paid tier | Reconcile public SLA wording with internal SLO. |

---

```text
Last reviewed: 2026-05-08
```
