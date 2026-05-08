---
name: publish-template
description: End-to-end checklist for publishing a marketplace template (creator flow) — manifest validation, screenshots, pricing, license, demo project, moderation submission, and post-publish monitoring. Use when shipping seed templates or implementing the creator publish UI.
---

# publish-template — publish a marketplace template

> Plan ref: [Sec. 56 (Marketplace lifecycle)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#56-marketplace--lifecycle),
> [Sec. 57 (Creator economy)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#57-creator-economy--payouts),
> [Sec. 58 (Manifest spec)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#58-template-manifest-spec),
> [Sec. 59 (Moderation)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#59-moderation--abuse).

This skill covers two cases:
1. **Manual seed template** — TESKEL team publishing canonical
   templates during Phase 1–3.
2. **Implementation reference** — when building the creator-facing
   publish wizard, every step must be enforced by the wizard or
   moderation.

## Manifest

`template.yaml` lives at the root of the template package.

```yaml
# Required fields
id: tmpl_lead-magnet-funnel        # globally unique, kebab-case, prefixed tmpl_
slug: lead-magnet-funnel            # url-safe slug
name: Lead Magnet Funnel
version: 1.2.0                      # semver; bump on every publish
authorOrgId: org_abc                # org publishing the template
license: SPDX:MIT                   # SPDX id; or "proprietary"
category: marketing                 # one of plan §58 categories
tags: [lead-gen, email, automation]

# Pricing
pricing:
  mode: paid                        # free | paid | tiered
  amountUsd: 49                     # cents-precision; only if paid
  currency: USD

# Compatibility
compat:
  teskel: ">=0.5 <1"                # required platform version range
  blocks:                           # builder block dependencies
    - hero-centered@^1
    - cta-banner@^1
  workflowNodes:                    # workflow node dependencies
    - http.request@^1
    - llm.complete@^1
    - email.send@^1

# Required external services (will be checked at install)
requires:
  - kind: oauth
    provider: gmail
    scopes: ['gmail.send']
  - kind: api-key
    provider: stripe
    optional: true

# What the template installs
install:
  workflows: [./workflows/main.json]
  pages: [./pages/landing.json, ./pages/thank-you.json]
  blocks: [./blocks/*.json]
  prompts: [./prompts/welcome-email.json]
  seedData: [./data/sample-leads.csv]

# Marketing
description: |
  Two-page lead magnet flow with welcome-email automation, integrated
  with Stripe for an upsell offer.
heroImage: ./media/hero.png
gallery:
  - ./media/preview-1.png
  - ./media/preview-2.png
  - ./media/preview-3.png
demoUrl: https://demo.teskel.dev/templates/lead-magnet-funnel

# Privacy & data handling
dataHandling:
  collectsPii: true
  thirdPartySharing: [stripe, gmail]

# Telemetry
analyticsEvents: [funnel.lead.captured, funnel.email.sent]

# Author / support
support:
  email: support@example.com
  docsUrl: https://example.com/templates/lead-magnet-funnel

changelog: ./CHANGELOG.md
```

Schema source of truth: `packages/template-engine/src/manifest-schema.ts`.
CI fails any PR whose `template.yaml` doesn't pass schema + lint.

## Steps (publish flow)

1. **Idea & scope.** Decide the template's value prop. Stick to one
   primary outcome ("capture leads", not "do everything").

2. **Build the project.** Build the template inside a real org/project
   in TESKEL Studio. Iterate until it works end-to-end.

3. **Extract & version.** Use the CLI:

   ```bash
   pnpm teskel template extract --project p_xxx --out ./templates/lead-magnet-funnel
   ```

   This produces the directory: `template.yaml`, `workflows/`,
   `pages/`, `blocks/`, `prompts/`, `data/`, `media/`.

4. **Fill in `template.yaml`** with all required fields. Use the
   schema as a guide; CI will fail otherwise.

5. **License.** Pick one:
   - `SPDX:MIT` (default, most permissive).
   - `SPDX:Apache-2.0`.
   - `SPDX:BUSL-1.1` (source-available).
   - `proprietary` — must include EULA at `./LICENSE.md`.
   No GPL/AGPL on marketplace (incompatible with embed scenarios).

6. **Pricing.**
   - Free templates: `mode: free`.
   - Paid templates: `mode: paid` and an `amountUsd`.
   - Tiered (rare): `mode: tiered` + `tiers: [...]`.
   - Author orgs need an active Stripe Connect account in good standing
     (`stripe_account.charges_enabled = true`).

7. **Media.**
   - Hero image: 1600×900 PNG/JPG, ≤500KB.
   - Gallery: 3–8 screenshots, ≤500KB each.
   - Optional 30–60s demo video (mp4, ≤25MB) at `./media/demo.mp4`.
   - All images alt-text required (`heroAlt`, `galleryAlt[]`).
   - Lighthouse a11y on the listing page must stay ≥95.

8. **Demo project.** Provision a public demo at
   `https://demo.teskel.dev/templates/<slug>` and link via `demoUrl`.
   This is auto-deployed by CI from the template's `pages/`.

9. **Compatibility.** Run:
   ```bash
   pnpm teskel template check ./templates/lead-magnet-funnel
   ```
   This validates manifest, workflow node versions, block versions,
   prompt slot ids, and dependency graph against the current platform.

10. **Tests.** Templates have automated install tests.
    - Install into a clean test org.
    - Assert all workflows compile.
    - Assert pages render (Playwright snapshot of demo URL).
    - Run any included scripted scenarios (`./tests/*.spec.ts`).

11. **Submit for moderation.** Manual templates: open a PR adding the
    template directory under `seeds/templates/`. Creator-published:
    the wizard sends the manifest + assets to the moderation queue.

12. **Moderation pipeline.**
    - Automated checks: schema, license, malware scan on assets,
      profanity/PII scan on copy, plagiarism check (cosine similarity
      ≥0.95 against existing templates).
    - Human review: 2 reviewers on first publish, 1 on minor updates.
    - SLA: 2 business days first review.

13. **Publish.** Once approved:
    - Manifest copied to `marketplace_listings`.
    - Asset bundle pinned to R2 with content-hash naming.
    - Listing page goes live.
    - Author email + Resend.
    - PostHog event `marketplace.template.published`.

14. **Post-publish monitoring.**
    - Track installs (`marketplace.install.completed`), refund rate,
      review distribution.
    - First 7 days: author + moderator on watch.
    - Crash-rate alert: if installs fail >5%, listing auto-paused +
      author notified.

## Updates / new versions

- Bump `version` in manifest (semver).
- Patch (1.0.x): bug fixes, copy tweaks. Auto-approved if no schema
  changes and the diff is small.
- Minor (1.x.0): additive features. Manual review.
- Major (x.0.0): breaking. Existing installs are NOT auto-upgraded;
  user opts in. Requires migration notes in `CHANGELOG.md`.

## Removal / unpublish

- `state: deprecated`: still installable, banner warns users.
- `state: archived`: no new installs; existing keep working.
- `state: takedown`: legal/abuse only; existing installs disabled.
  Audit-logged with reason.

## Pitfalls

- Hard-coding org-specific data (real customer emails, internal
  domains) in workflows or pages — fails moderation.
- Including `.env` files or secrets — auto-rejected.
- Marketing-style superlatives in description ("BEST template EVER!")
  — moderation rejects.
- Forgetting `compat:` block — installs break on next platform
  release.
- License mismatch: code says BSL but manifest says MIT — manual
  review will catch, but slows down publish.
- Demo URL not provisioned — listing page looks broken.

## Done when

- Manifest passes schema + lint.
- Demo URL up and matches manifest.
- Install tests green.
- Moderation approved.
- Listing live and discoverable in marketplace.
- Author payout settings verified (paid templates).
- Author notified.
