# Plans, entitlements, & metering

> **Status:** Stable. Reviewed quarterly + when pricing changes.
>
> **Governed by:** Plan §10 (plan tiers + revenue share),
> §62–§64 (billing, metering, growth). Operationalized by
> `add-feature-flag`, `add-rbac-role`, `add-integration` skills.

This document is the **source of truth** for what each plan grants,
how usage is metered, and how overages and downgrades behave. Sales,
support, and engineering all derive from this file. Do not encode
plan-specific values in product code — read them from the
`packages/billing/plans` registry which is generated from this doc.

---

## 1. Plans

| Plan | Audience | Monthly | Annual | Trial |
| --- | --- | --- | --- | --- |
| **Free** | Tire-kickers, students, hobby | $0 | n/a | always |
| **Starter** | Solo founder, indie hacker | $29 | $290 | 14 d |
| **Pro** | Small team, paying creator | $99 | $990 | 14 d |
| **Business** | Growing team, multi-org needs | $299 | $2,990 | 14 d |
| **Enterprise** | Compliance, SAML, region pin, BAA | custom | custom | custom |

Currency: USD primary; IDR + EUR + GBP secondary (Phase 1+).
Non-USD pricing is set in Stripe; checkout shows local currency.

---

## 2. Entitlement matrix

| Capability | Free | Starter | Pro | Business | Enterprise |
| --- | --- | --- | --- | --- | --- |
| **Workspaces** | 1 | 1 | 3 | 10 | unlimited |
| **Members per workspace** | 2 | 5 | 15 | 50 | per contract |
| **Active workflows** | 3 | 25 | 100 | unlimited | unlimited |
| **Active pages** | 5 | 50 | 200 | unlimited | unlimited |
| **KB documents** | 50 | 500 | 5,000 | 50,000 | per contract |
| **Knowledge base size (MB)** | 50 | 500 | 5,000 | 50,000 | per contract |
| **LLM tokens / month** (input + output combined) | 100k | 5M | 50M | 250M | per contract |
| **LLM model access** | basic only | basic + mid | all + frontier | all + frontier | all + frontier (per region) |
| **Workflow runs / month** | 1,000 | 25,000 | 250,000 | 2,500,000 | unlimited |
| **Sandbox seconds / month** | 0 | 1,800 (30 min) | 18,000 | 180,000 | per contract |
| **Custom domain** | ✗ | 1 | 5 | 25 | per contract |
| **Subdomain `*.teskel.app`** | watermarked | clean | clean | clean | clean |
| **Marketplace publishing** | ✗ | 1 listing | 10 listings | unlimited | unlimited |
| **Stripe Connect (creator payout)** | ✗ | ✓ | ✓ | ✓ | ✓ |
| **Revenue share on marketplace sales** | n/a | 80% | 85% | 85% | 85% |
| **Audit log retention** | 7 d | 30 d | 90 d | 1 y | 7 y |
| **DSAR self-service** | ✗ | ✓ | ✓ | ✓ | ✓ |
| **API rate limit** | 50 rpm | 100 rps | 500 rps | 2,000 rps | per contract |
| **Webhook signing rotation** | manual | manual | quarterly auto | monthly auto | per contract |
| **Outbound email / month (transactional)** | 100 | 5,000 | 50,000 | 250,000 | per contract |
| **Custom prompt slots** | 5 | 50 | 500 | unlimited | unlimited |
| **Eval suite (Promptfoo)** | ✗ | basic | full | full | full |
| **SSO** | Google / GitHub OAuth | + Magic link | + Microsoft | SAML / OIDC | SAML / OIDC + custom IdP |
| **SCIM** | ✗ | ✗ | ✗ | ✓ | ✓ |
| **2FA enforcement** | optional | optional | required for owners | required for all | required for all |
| **HIPAA / BAA** | ✗ | ✗ | ✗ | ✗ | ✓ (Phase 6) |
| **EU AI Act mapping** | ✗ | ✗ | ✓ | ✓ | ✓ |
| **Region pinning** | ✗ | ✗ | ✗ | ✗ | ✓ |
| **Self-host Helm chart** | ✗ | ✗ | ✗ | ✗ | ✓ |
| **Support SLA** | community | best-effort | 24 h | 8 h biz | 1 h |
| **Designated CSM** | ✗ | ✗ | ✗ | ✗ | ✓ |

A capability marked ✗ at a tier means it is hard-gated — the API
returns `PLAN_NOT_ENTITLED` if attempted.

---

## 3. Metering

### 3.1 What is metered

| Meter | Unit | Reset window | Source |
| --- | --- | --- | --- |
| `llm.tokens.input` | tokens | calendar month | AI gateway |
| `llm.tokens.output` | tokens | calendar month | AI gateway |
| `workflow.runs` | runs | calendar month | workflow runtime |
| `sandbox.seconds` | seconds | calendar month | sandbox broker |
| `email.outbound` | emails | calendar month | email worker |
| `kb.size_mb` | MB stored | live (continuous reconciliation) | object storage + DB |
| `members.active` | members with login in last 30 d | live | identity service |
| `webhooks.outbound` | deliveries | calendar month | outbox sender |
| `api.requests` | rate-limited; not metered for billing on bundled plans | per second / minute | API gateway |

### 3.2 How it's collected

- Each surface emits a metering event to the `metering_events` table
  + Loki stream. Events are append-only.
- A nightly aggregator collapses events into `metering_aggregates`
  (per `(tenant, meter, day)`) for fast dashboards.
- A monthly close-out (3rd of next month) freezes the prior month's
  totals into `metering_invoices_input` for billing.

### 3.3 Quotas vs limits

- **Quota**: monthly entitlement. When 80% consumed, in-app banner;
  100% consumed → soft-stop with typed error
  `QUOTA_EXCEEDED`. Owner can buy overage (Pro+) or wait for reset.
- **Limit**: hard ceiling that protects platform safety regardless
  of plan (e.g. max 10k workflows in one workspace). Hitting a
  limit returns `LIMIT_EXCEEDED` and is documented per-feature.

---

## 4. Overages

Available on **Pro+**. Free / Starter cap at quota, no overage.

| Meter | Overage rate (Pro/Business) | Cap |
| --- | --- | --- |
| `llm.tokens.input` | $0.50 per 100k | 5× plan quota |
| `llm.tokens.output` | $1.50 per 100k | 5× plan quota |
| `workflow.runs` | $1.00 per 1k | 5× plan quota |
| `sandbox.seconds` | $0.10 per 60 s | 5× plan quota |
| `email.outbound` | $0.50 per 1k | 5× plan quota |
| `kb.size_mb` | $0.05 per GB-month | 10× plan quota |

Overages are toggled **off by default**. Owner explicitly enables in
billing settings; auto-off if monthly invoice exceeds 2× plan
subscription (safety brake).

Enterprise overage is per-contract; billed on the same invoice line.

---

## 5. Trials, downgrades, cancellations

### 5.1 Trial

- 14 days, full Pro entitlements (default).
- No credit card required for **Starter** trial; required for
  **Pro** and **Business** trial.
- Trial ends → tenant moves to **Free**; data retained per
  retention class B floors.

### 5.2 Downgrade

- Allowed at the end of current billing period (no proration).
- If downgraded plan's quotas are exceeded:
  - Workflows over limit are **paused**, not deleted.
  - KB over limit triggers banner; reads work, writes blocked.
  - Members over limit prompts owner to remove or upgrade; until
    resolved, new logins blocked for over-limit members.
- Audit log retention shortens immediately to the new plan's
  window; pruning runs at next nightly sweep (per
  [`docs/data/retention.md`](../data/retention.md)).

### 5.3 Cancellation

- "Pause" (Starter+): workspace marked `paused`; data retained 90 d
  beyond retention windows; reactivation restores access.
- "Cancel": workspace enters **30-day grace** at Free entitlements.
  After 30 d, tenant data is purged per retention class A,B floors
  unless legal hold.
- DSAR-driven erasure overrides grace period (per
  `gdpr-data-request` skill).

---

## 6. Discounts & credits

| Mechanism | When | Implementation |
| --- | --- | --- |
| Annual discount | 17% off (2 months free) | Stripe Price object. |
| Education / nonprofit | 50% off Pro | Coupon, manual review. |
| Open source maintainers | Pro free up to 10 seats | Coupon, manual review. |
| Goodwill credits (incident, support compensation) | as issued | `billing_credits` table; applied to next invoice; never refunded as cash. |
| Marketplace creator earned credits | from referral / promo | same table; capped at 100% of next invoice. |

All issuance audit-logged with operator + reason.

---

## 7. Marketplace economics

See [`docs/marketplace/manifest-spec.md`](../marketplace/manifest-spec.md)
for the lifecycle. The platform fee:

| Plan | Creator share | Platform share | Stripe fee |
| --- | --- | --- | --- |
| Starter | **80%** | 20% | passed to creator |
| Pro | **85%** | 15% | passed to creator |
| Business | **85%** | 15% | passed to creator |
| Enterprise | per contract (typically 85%) | balance | per contract |

Free plan cannot publish.

Payouts via Stripe Connect:

- Daily payout schedule by default; weekly option.
- Minimum payout: **$10**.
- KYC required before first payout (Stripe Identity); creator data
  held under Class A 5-y retention.
- 1099 / TIN reporting end-of-year; per US tax rules.

Refund flow:

- Buyer requests refund within `pricing.refund_window_days` from
  install.
- Refund mechanism: full reversal via Stripe; clawback from creator
  next payout.
- Refund rate per listing visible to creator; high refund rate (>
  10%) flags moderation review.

---

## 8. Tax

- Stripe Tax handles VAT / GST / state sales tax automatically once
  enabled.
- B2B reverse charge supported (EU VAT ID validation via Stripe).
- Indonesian VAT (PPN) registration target Phase 3+; until then,
  Indonesian customers are billed exclusive of PPN.
- Invoice template includes seller details, buyer details, line
  items, tax breakdown, currency, period covered.

---

## 9. Lifecycle events (Stripe → TESKEL)

The Stripe webhook is consumed per
[`add-webhook-receiver`](../../.agents/skills/add-webhook-receiver/SKILL.md).
Key events handled:

| Stripe event | TESKEL action |
| --- | --- |
| `checkout.session.completed` | Provision plan or marketplace install. |
| `customer.subscription.updated` | Update plan record; recalc entitlements. |
| `customer.subscription.deleted` | Downgrade tenant to Free; trigger 30-d grace. |
| `invoice.paid` | Mark invoice paid in mirror. |
| `invoice.payment_failed` | Notify owner; retries per Stripe; on third fail, suspend write access. |
| `payment_intent.succeeded` | One-time purchase confirmation. |
| `charge.refunded` | Process refund; clawback from creator (marketplace). |
| `payout.paid` / `payout.failed` | Update creator payout record. |
| `radar.early_fraud_warning.created` | Flag for review; freeze listing if creator. |

Stripe events older than 7 d are ignored — we treat them as already
processed unless the audit trail says otherwise.

---

## 10. Plan registry (engineering reference)

Engineering reads from
`packages/billing/plans/registry.ts` (generated from this doc):

```ts
export const PLANS = {
  free: { id: "free", price_usd: 0, ... },
  starter: { id: "starter", price_usd: 29, annual_usd: 290, ... },
  pro: { id: "pro", price_usd: 99, annual_usd: 990, ... },
  business: { id: "business", price_usd: 299, annual_usd: 2990, ... },
  enterprise: { id: "enterprise", price_usd: null, ... },
} as const;

export const ENTITLEMENTS = {
  workflows_active: { free: 3, starter: 25, pro: 100, business: Infinity, enterprise: Infinity },
  // ...
} as const;
```

A change to plans / entitlements is a PR that updates **both** this
doc and the generated TypeScript. CI fails if they drift.

---

## 11. Update protocol

| Change | Process |
| --- | --- |
| Tighten an entitlement | RFC + ADR (impacts existing customers); 30-d notice. |
| Loosen an entitlement | PR; ship in next train. |
| Add a new entitlement | PR + add to metering pipeline + add to UI banners. |
| Change a price | RFC + ADR; honor existing subs at old price for 12 months; new sign-ups at new price. |
| Add a plan tier | RFC + ADR + Stripe Price + UI rollout. |
| Add a meter | PR + emit events + add to monthly close. |

A pricing change requires legal review for Indonesian, EU, and US
disclosure rules.

---

## 12. References

- Plan §10, §62–§64.
- [`docs/api/conventions.md`](../api/conventions.md) — error codes
  (`PLAN_NOT_ENTITLED`, `QUOTA_EXCEEDED`, `LIMIT_EXCEEDED`).
- [`docs/data/retention.md`](../data/retention.md) — Class D
  (financial) data.
- [`docs/marketplace/manifest-spec.md`](../marketplace/manifest-spec.md).
- [`docs/legal/README.md`](../legal/README.md) — ToS, Privacy, DPA.
- `add-feature-flag`, `add-integration` (Stripe), `gdpr-data-request`
  skills.
