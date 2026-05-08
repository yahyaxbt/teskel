# Template manifest spec

> **Status:** Stable. Versioned independently — see §11.
>
> **Governed by:** Plan §57 (marketplace lifecycle), §58 (creator
> economy), §60 (manifest spec), §61 (moderation).
> Operationalized by `publish-template` skill.

A **template** is a packageable bundle of TESKEL primitives — workflows,
prompt slots, KB documents, blocks, pages, integrations — that a
buyer can install into their workspace.

This document defines the **manifest** that describes a template.
Every published template MUST have a manifest. The manifest is the
**source of truth** the marketplace, installer, and moderation
pipeline use.

---

## 1. File layout

```
my-template/
├── teskel.template.yaml        # the manifest, authoritative
├── README.md                   # rendered on listing page
├── CHANGELOG.md                # required from v1.1.0 onward
├── LICENSE.md                  # required
├── workflows/                  # exported workflow definitions
│   └── *.workflow.json
├── prompts/                    # prompt slot definitions
│   └── *.prompt.yaml
├── kb/                         # KB seed docs (markdown)
│   └── *.md
├── pages/                      # Puck page bundles
│   └── *.page.json
├── blocks/                     # custom block source (optional)
│   └── *
├── assets/                     # images, icons, screenshots
│   └── *
├── samples/                    # example inputs / fixtures
│   └── *.json
└── tests/                      # eval sets (Promptfoo) + e2e fixtures
    └── *
```

Every path referenced from the manifest MUST exist in the bundle.
The `publish-template` skill validates this.

---

## 2. Manifest (`teskel.template.yaml`)

The full schema, as YAML:

```yaml
# ─── identity ────────────────────────────────────────────────
manifest_version: 1                # always 1 for v1 spec; bumped on breaking changes
id: lead-scorer-v3                 # globally unique, kebab-case, immutable
name: Lead Scorer v3               # display name; can change with new version
slug: lead-scorer-v3               # SEO slug; immutable once published
version: 1.4.2                     # semver of THIS template release
canonical_id: 01J9X3K2...          # ULID assigned by TESKEL on first publish

creator:
  org_id: 01J...                   # creator org (verified)
  display_name: Acme Studio
  url: https://acme.example
  support_email: hello@acme.example
  signature: "ed25519:base64..."   # signs the bundle hash; verified at install

# ─── catalog metadata ────────────────────────────────────────
category: sales-ops                # one of the controlled vocab
tags: [b2b, lead-scoring, llm, openrouter]
description: >
  Inbound lead scorer that uses LLM + KB lookups to assign a score
  and route to the right team. Includes Slack and HubSpot examples.
icon: assets/icon.svg
screenshots:
  - path: assets/hero.png
    alt: Studio canvas with the workflow
  - path: assets/run.png
    alt: Sample run result with score breakdown
hero_video: assets/walkthrough.mp4   # optional
demo_url: https://demo.acme.example/lead-scorer  # optional, if hosted

# ─── pricing & licensing ─────────────────────────────────────
license: commercial                  # commercial | mit | apache-2 | proprietary | …
pricing:
  model: one-time                    # one-time | subscription | free
  amount_minor: 4900                 # 49.00 in stripe currency
  currency: usd
  billing_cycle: null                # required if model=subscription: month|year
  trial_days: 0
  refund_window_days: 14             # default 14 days

# ─── platform requirements ───────────────────────────────────
requires:
  teskel:
    api_version: ">=1.0.0 <2.0.0"
    plan: starter                    # min plan to install (free|starter|pro|business|enterprise)
  features:                          # platform features the template uses
    - workflow.runtime
    - ai.gateway
    - kb.pgvector
  integrations:                      # which third-parties needed
    - vendor: openrouter
      models: [claude-sonnet-4, gpt-4o]
      reason: "Primary scoring model + fallback"
    - vendor: stripe
      reason: "Optional: charge customer on score >= 80"
      optional: true
    - vendor: slack
      reason: "Notify sales channel on hot lead"
      scope_required: ["chat:write"]
  data:
    persistent_storage_mb: 50        # extra workspace storage required
    pii_processed: ["email", "name", "company"]   # transparency required

# ─── declared scopes & permissions ───────────────────────────
scopes:
  workflows: [read, write, execute]
  kb: [read, write]
  outbound_http: [allowlisted]       # uses HTTP node, allowlist enforced
  llm: [primary, fallback]
  email: [send]
  webhooks: [outbound]
  audit_log: [read-own]              # never write to audit_log
elevated_scopes_explanation: |
  This template needs the `email:send` scope to send confirmation
  emails on hot leads. Disable in workspace settings if not used.

# ─── package contents ────────────────────────────────────────
contents:
  workflows:
    - path: workflows/score-lead.workflow.json
      id: score-lead
      title: Score Lead
      trigger: webhook              # webhook | schedule | manual | form
  prompts:
    - path: prompts/score.prompt.yaml
      id: score-prompt
      slot: marketplace/lead-scorer/score
  kb:
    - path: kb/playbook.md
      id: sales-playbook
      mime: text/markdown
  pages:
    - path: pages/dashboard.page.json
      id: dashboard
      route: /dashboard
  blocks: []                         # custom blocks, if any
  cron_jobs: []                      # scheduled jobs, if any
  webhooks:
    inbound:
      - path: receivers/webhook.json
        id: incoming-lead
        url_template: /webhooks/lead-scorer/incoming

# ─── eval & quality ──────────────────────────────────────────
quality:
  evals:
    - path: tests/score.evalset.yaml
      slot: marketplace/lead-scorer/score
      threshold_pass_rate: 0.92
  e2e_fixtures:
    - path: tests/e2e/sample-run.json
  benchmark:
    expected_p95_latency_ms: 8000
    expected_avg_cost_usd: 0.012     # per run
  documentation_completeness: full   # full | partial | minimal

# ─── support ─────────────────────────────────────────────────
support:
  channels: ["email", "discord"]
  sla_response_hours: 48
  changelog_path: CHANGELOG.md
  versioning_policy: semver

# ─── compliance ──────────────────────────────────────────────
compliance:
  data_residency_supported: [us, eu]
  hipaa_ready: false
  gdpr_dpa_acknowledged: true
  exports_personal_data: false       # if the template SENDS PII to vendors
  ai_model_training_optout: true     # provider config

# ─── publication ─────────────────────────────────────────────
publication:
  status: published                  # draft | review | published | retired
  published_at: 2026-05-08T09:14:00Z
  visible_in_search: true
  featured_until: null
  retired_at: null
```

YAML keys are stable. New keys may be added (additive); removing or
renaming a key requires a manifest version bump (§11).

---

## 3. Required vs optional

| Section | Required at publish? |
| --- | --- |
| `manifest_version`, `id`, `name`, `slug`, `version`, `canonical_id` | yes |
| `creator` (full) | yes |
| `category`, `tags`, `description` (≥ 50 chars) | yes |
| `icon` + at least one `screenshots[]` | yes |
| `license` | yes |
| `pricing` | yes |
| `requires.teskel` | yes |
| `requires.integrations[]` | yes if any integration is used |
| `scopes` | yes |
| `elevated_scopes_explanation` | yes if any non-default scope is requested |
| `contents` (at least one item) | yes |
| `quality.evals` | yes if the template uses LLM |
| `quality.benchmark` | recommended |
| `support` | yes |
| `compliance` | yes |
| `publication.status` | set by TESKEL (creator can request `draft` / `review`) |

---

## 4. Validation pipeline

The `publish-template` skill executes:

1. **Schema validation** — Zod parser against `manifest_version: 1`
   schema at `packages/marketplace/schemas/template-manifest.v1.ts`.
2. **Path existence** — every `path:` resolves to a file in the
   bundle.
3. **Signature check** — bundle SHA-256 is hashed; the creator's
   signature is verified against pinned creator public key.
4. **License sanity** — `license` matches `LICENSE.md` content
   approximately (heuristic; reviewer confirms).
5. **Linting** — workflow JSONs validate against workflow schema;
   prompt YAMLs validate against prompt-slot schema; pages validate
   against Puck schema.
6. **Eval execution** — `quality.evals[]` runs in a sandbox; pass
   rate must meet `threshold_pass_rate`.
7. **E2E fixture run** — `quality.e2e_fixtures[]` runs against a
   throwaway sandbox tenant; must succeed.
8. **Compliance flags** — if `exports_personal_data: true` or HIPAA
   needed, the listing requires a stricter human review.
9. **Cost estimate** — benchmark runs N=20 sample runs in canary
   sandbox; record p50/p95 cost; warn if `expected_avg_cost_usd`
   off by >25%.
10. **Auto-moderation** — content classifier on description /
    screenshots; flag for human review if score above threshold.
11. **Human moderation** — see Plan §61. Decision: `approved`,
    `changes_requested`, `rejected`.

If any step fails, the publish call returns a typed error (per
[`docs/api/conventions.md`](../api/conventions.md) §4.2) and no
listing is created.

---

## 5. Install / fork

When a buyer installs:

1. Buyer reviews the listing → clicks "Install" → Stripe Checkout
   (skipped for free templates).
2. On webhook success (Stripe `checkout.session.completed`),
   `template_purchases` row inserted (per `docs/data/retention.md`
   class D rules).
3. Installer worker runs in buyer's tenant context (`withTenant`):
   - Materializes workflows, prompt slots, KB docs, pages, blocks
     under buyer's namespace.
   - Wires required integrations (prompts buyer if any are missing).
   - Records `template_installations` row with manifest pin
     (`canonical_id` + `version`).
4. Buyer can **update** to a newer template version via the
   marketplace UI; update is opt-in, manifest pinning prevents
   surprise migrations.

Forking is the same flow but creates a draft owned by the buyer's
org with no upstream link (the buyer can republish under their own
`creator.org_id`).

---

## 6. Updates & versioning

- **Patch** (1.4.2 → 1.4.3): bug fixes, doc fixes. Auto-updates
  enabled for tenants that opted in.
- **Minor** (1.4.2 → 1.5.0): additive features, new prompt slot
  versions. Auto-updates by default; opt-out per tenant.
- **Major** (1.x.y → 2.0.0): breaking changes (renamed inputs,
  removed nodes). Always opt-in; old version remains installable for
  **12 months**.

Manifest `version` is semver; bumped on every publish. The
`canonical_id` never changes. The `slug` never changes once
published (shadow-redirected if name changes).

---

## 7. Retirement & takedown

| Action | Trigger | Effect |
| --- | --- | --- |
| **Retire** | Creator decides to stop selling. | Existing installs keep working; new buyers blocked; listing badged "retired". |
| **Takedown — soft** | Moderation flag (e.g. license dispute, broken integration). | Listing hidden from search; existing installs keep working; creator notified, given 14 days to respond. |
| **Takedown — hard** | Severe violation (malware, IP infringement, illegal content). | Listing removed; existing installs disabled; refunds processed per refund policy. |
| **DMCA / legal** | Counsel-driven. | Per legal team; documented in `audit_log`. |

Takedown decisions and appeals are tracked in
`marketplace_moderation_decisions` and exposed to creators in their
dashboard.

---

## 8. Permissions a template can request

Templates can request *only* permissions explicitly listed in
[`docs/security/rbac-matrix.md`](../security/rbac-matrix.md), grouped
into the `scopes` map of the manifest.

A template that exceeds its declared scopes at runtime fails closed
— the workflow node returns a typed `SCOPE_DENIED` error.

Templates **cannot** request:

- `system.*` permissions.
- Cross-tenant reads/writes.
- Direct DB access.
- Direct R2 bucket access (must go through signed-URL helper).
- `audit_log:write`.

---

## 9. Required disclosures

Marketplace listings prominently display:

- Total installs, last 30 d installs, average rating, refund rate.
- Required integrations + scopes (with explanations).
- PII processed (`requires.data.pii_processed`).
- Whether the template exports PII to a vendor
  (`compliance.exports_personal_data`).
- Last benchmark cost / latency.
- Eval pass rate of the latest version.
- License.

A buyer cannot install without acknowledging these disclosures.

---

## 10. Telemetry

Per-listing metrics emitted to the `Marketplace — Listings`
dashboard:

| Metric | Notes |
| --- | --- |
| impressions | listing-card and detail-page views |
| installs | succeeded |
| install conversion | installs / detail-page-views |
| refund rate | refunds / installs |
| update adoption rate | tenants on latest version / total tenants |
| eval pass rate over time | per published version |
| support ticket volume | per listing |

These feed into the **Marketplace Health Score** for ranking +
moderation prioritization.

---

## 11. Manifest spec versioning

| `manifest_version` | Status | Notes |
| --- | --- | --- |
| `1` | **Current.** | Initial spec. |
| `2` | reserved for future breaking changes (e.g. agent-tool manifests) | not yet defined |

Adding a key is **additive** and does not bump `manifest_version`.
Renaming, removing, or changing semantics of a key requires a new
`manifest_version` AND a transition window during which both
versions are accepted by the publisher.

---

## 12. References

- Plan §57–§61 (marketplace + creator economy).
- [`docs/security/rbac-matrix.md`](../security/rbac-matrix.md).
- [`docs/data/retention.md`](../data/retention.md).
- [`docs/billing/plans.md`](../billing/plans.md).
- `publish-template` skill.
- (planned) `marketplace-takedown` skill.
