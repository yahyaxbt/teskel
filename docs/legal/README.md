# Legal documents

> **Status:** Skeleton. Each document below is a **placeholder**
> until counsel reviews and approves the production text. Do **not**
> publish any of these to customers from this directory — they are
> drafts intended to capture intent and seed the lawyer's work.
>
> **Governed by:** Plan §53 (compliance roadmap), §50 (data
> protection), §62 (billing), §63 (marketplace).

This directory holds the **engineering-side** view of the legal
documents we ship as a platform. Counsel owns the final text;
engineering owns the *behavior* the text describes.

When the production text differs from this directory, **the
production text wins** — but the lived behavior should still match.
File a story to reconcile.

---

## 1. Documents we publish

| Document | Audience | Where it lives publicly | Skeleton in this dir | Owner |
| --- | --- | --- | --- | --- |
| **Terms of Service** (ToS) | All users | https://teskel.app/legal/terms | (planned) `tos.md` | Legal + Backend |
| **Privacy Policy** | All users | https://teskel.app/legal/privacy | (planned) `privacy.md` | Legal + Identity |
| **Acceptable Use Policy** (AUP) | All users | https://teskel.app/legal/aup | (planned) `aup.md` | Legal + Trust & Safety |
| **Data Processing Agreement** (DPA) | B2B / Business+ tenants | per request via legal@ | (planned) `dpa.md` | Legal + Backend |
| **Sub-processors list** | All users | https://teskel.app/legal/subprocessors | (planned) `subprocessors.md` | Legal + Backend |
| **Cookie / Tracking Policy** | All users (esp. EU) | https://teskel.app/legal/cookies | (planned) `cookies.md` | Legal + Frontend |
| **Marketplace Creator Agreement** | Creators publishing templates | sign-on at marketplace onboarding | (planned) `creator-agreement.md` | Legal + Marketplace |
| **Marketplace Buyer Terms** | Anyone installing a paid template | shown at install | (planned) `buyer-terms.md` | Legal + Marketplace |
| **AI Usage Policy** | All users | https://teskel.app/legal/ai-usage | (planned) `ai-usage.md` | Legal + AI |
| **Service Level Agreement** (paid plans) | Business+ / Enterprise | per contract or public page | (planned) `sla.md` | Legal + SRE |
| **Vulnerability Disclosure Policy** | Security researchers | https://teskel.app/security and `SECURITY.md` | covered in [`SECURITY.md`](../../SECURITY.md) | Security |
| **Code of Conduct** | Contributors / community | repo root | covered in [`CODE_OF_CONDUCT.md`](../../CODE_OF_CONDUCT.md) | Community |
| **DMCA Policy** | Rightsholders | https://teskel.app/legal/dmca | (planned) `dmca.md` | Legal + Marketplace |
| **EU Representative** (Phase 4+) | EU regulators | https://teskel.app/legal/eu-rep | (planned) `eu-rep.md` | Legal |

Each row above eventually lands as a markdown file in this
directory **plus** a rendered page on the public site. Engineering
keeps the markdown in step with reality; counsel owns the legal
language.

---

## 2. What engineering must keep in sync

Several legal documents reference operational facts. Engineering is
responsible for keeping the docs accurate, not for choosing the
words.

| Operational fact | Drives | Source of truth |
| --- | --- | --- |
| Sub-processors list | `subprocessors.md` | `packages/integrations/*` registry; updated on `add-integration` |
| Data retention windows | Privacy Policy, DPA, AUP | [`docs/data/retention.md`](../data/retention.md) |
| RBAC capabilities | DPA (technical & organizational measures) | [`docs/security/rbac-matrix.md`](../security/rbac-matrix.md) |
| Threat model & controls | DPA | [`docs/architecture/threat-model.md`](../architecture/threat-model.md) |
| Plan tiers / pricing | ToS, SLA | [`docs/billing/plans.md`](../billing/plans.md) |
| Marketplace economics + refund window | Creator Agreement, Buyer Terms | [`docs/marketplace/manifest-spec.md`](../marketplace/manifest-spec.md) §2 + [`docs/billing/plans.md`](../billing/plans.md) §7 |
| AI usage + opt-out | AI Usage Policy, Privacy | [`docs/architecture/threat-model.md`](../architecture/threat-model.md) §2.5 + Plan §52 |
| SLO / SLA values | SLA | [`docs/observability/slo-sli.md`](../observability/slo-sli.md) |
| Incident response | DPA (breach notification) | [`incident-sev1`](../../.agents/skills/incident-sev1/SKILL.md), [`incident-sev2`](../../.agents/skills/incident-sev2/SKILL.md) |
| DSAR pipeline | Privacy Policy | [`gdpr-data-request`](../../.agents/skills/gdpr-data-request/SKILL.md) |

A change to any of those operational sources triggers a story to
update the matching legal document. The story owner is the team
that made the operational change; counsel reviews the resulting
copy.

---

## 3. Compliance roadmap (per Plan §53)

| Phase | Compliance posture | Documents that ship |
| --- | --- | --- |
| **Phase 1** | Privacy baseline | Privacy Policy, ToS, AUP, Cookie Policy, sub-processors list |
| **Phase 2** | Marketplace go-live | Creator Agreement, Buyer Terms, DMCA Policy |
| **Phase 3** | Stripe Connect KYC | Tax disclosures, payout terms (folded into Creator Agreement) |
| **Phase 4** | SOC2 Type 1 → Type 2 + ISO 27001 | DPA (formal), SLA (Business+), Sub-processor change-notice flow, EU Representative appointed |
| **Phase 5** | EU AI Act mapping | AI Usage Policy expanded; AI risk classification page; transparency notes per slot |
| **Phase 6** | HIPAA (BAA) | BAA template, HIPAA-specific addenda; region pinning disclosure |

Stories for each item are filed at the start of the phase per
[`kickoff-phase`](../../.agents/skills/kickoff-phase/SKILL.md) skill.

---

## 4. How customers find these documents

- Public marketing site: footer links to ToS, Privacy, AUP, Cookies,
  Sub-processors, DMCA, Security.
- In-app: `Settings → Legal` lists all of the above plus DPA on
  Business+ plans.
- Stripe Checkout / billing UI: links to ToS + Privacy.
- Marketplace install flow: Buyer Terms + the specific Creator
  Agreement of the listing.
- Email footers: links to Privacy + Unsubscribe (per `add-email-template`
  skill).

A broken link to a legal document is a **Sev2** — see
[`incident-sev2` skill](../../.agents/skills/incident-sev2/SKILL.md).

---

## 5. Versioning legal docs

- Every legal doc has a **last-updated date** at the top.
- Every change is recorded in a `CHANGELOG.md` next to the doc.
- Material changes (e.g. adding a sub-processor that processes PII)
  trigger a customer email **at least 30 days** before they take
  effect, except changes legally required to be immediate.
- The previous version is archived (never deleted) for 7 years to
  serve any audit / dispute.

---

## 6. Disclaimers

This directory is **not** a substitute for legal advice. The
content here describes engineering's best understanding of intended
behavior; customers and counsel should consult the live published
versions on https://teskel.app/legal.

Nothing in this directory waives any right reserved by either party
in the published agreements.

---

## 7. References

- [`SECURITY.md`](../../SECURITY.md) — vulnerability disclosure.
- [`CODE_OF_CONDUCT.md`](../../CODE_OF_CONDUCT.md).
- [`docs/data/retention.md`](../data/retention.md).
- [`docs/security/rbac-matrix.md`](../security/rbac-matrix.md).
- [`docs/billing/plans.md`](../billing/plans.md).
- [`docs/marketplace/manifest-spec.md`](../marketplace/manifest-spec.md).
- Plan §50, §53, §62, §63.
