# TESKEL — Fullstack Build Breakdown v2

**AI Startup Factory + Infrastructure Platform**

Clean • Modern • Global-Class • AI-Native • Production-Ready

> Versi 2 dari plan ini dirombak total agar bisa langsung dipakai sebagai
> *single source of truth* untuk full build. Setiap keputusan dibuat eksplisit
> (bukan opsi tergantung selera), setiap modul punya **deliverable** dan
> **definition of done**, dan setiap area risiko punya **mitigasi terukur**.
>
> Plan ini ditulis untuk dibaca oleh: founder, tech lead, engineer, designer,
> data, ops, marketing, legal, dan AI coding agents (Devin/Cursor/Copilot).

---

## Daftar Isi

- [0. Executive Summary](#0-executive-summary)
- [1. Konvensi Dokumen, Notasi, Status](#1-konvensi-dokumen-notasi-status)
- [2. North Star, Mission, Vision, OKR](#2-north-star-mission-vision-okr)
- **Part I — Strategy & Product**
  - [3. Product Positioning](#3-product-positioning)
  - [4. ICP & Persona Deep Dive](#4-icp--persona-deep-dive)
  - [5. Competitive Landscape & Moat](#5-competitive-landscape--moat)
  - [6. Business Model & Pricing](#6-business-model--pricing)
  - [7. KPI Framework](#7-kpi-framework)
  - [8. Hypotheses & Research Backlog](#8-hypotheses--research-backlog)
- **Part II — Design**
  - [9. Design Philosophy](#9-design-philosophy)
  - [10. Design System Spec](#10-design-system-spec)
  - [11. Information Architecture](#11-information-architecture)
  - [12. Golden UX Flows](#12-golden-ux-flows)
  - [13. Empty / Error / Loading Patterns](#13-empty--error--loading-patterns)
  - [14. Accessibility (WCAG 2.2 AA)](#14-accessibility-wcag-22-aa)
  - [15. Internationalization](#15-internationalization)
- **Part III — Architecture**
  - [16. Architecture Overview (C4-Light)](#16-architecture-overview-c4-light)
  - [17. Monorepo & Build Tooling](#17-monorepo--build-tooling)
  - [18. Frontend Architecture](#18-frontend-architecture)
  - [19. Backend Architecture](#19-backend-architecture)
  - [20. API Design](#20-api-design)
  - [21. Database & Data Modeling](#21-database--data-modeling)
  - [22. Multi-Tenancy & RLS](#22-multi-tenancy--rls)
  - [23. AI Infrastructure](#23-ai-infrastructure)
  - [24. Workflow Runtime](#24-workflow-runtime)
  - [25. Visual App Builder](#25-visual-app-builder)
  - [26. Real-Time & Collaboration](#26-real-time--collaboration)
  - [27. Storage & Files](#27-storage--files)
  - [28. Queues, Jobs, Schedulers](#28-queues-jobs-schedulers)
  - [29. Caching Layers](#29-caching-layers)
  - [30. Search](#30-search)
  - [31. Notifications System](#31-notifications-system)
  - [32. Sandbox Execution Layer](#32-sandbox-execution-layer)
  - [33. SDK & Plugin Architecture](#33-sdk--plugin-architecture)
  - [34. CLI Tool (`teskel`)](#34-cli-tool-teskel)
- **Part IV — Operations**
  - [35. Environments](#35-environments)
  - [36. Deployment Infrastructure](#36-deployment-infrastructure)
  - [37. CI/CD Pipelines](#37-cicd-pipelines)
  - [38. Configuration & Secrets](#38-configuration--secrets)
  - [39. Observability](#39-observability)
  - [40. SLI / SLO / SLA](#40-sli--slo--sla)
  - [41. Incident Response & Runbooks](#41-incident-response--runbooks)
  - [42. Backup, DR, Retention](#42-backup-dr-retention)
  - [43. Performance Budgets & Load Testing](#43-performance-budgets--load-testing)
  - [44. FinOps & Cost Management](#44-finops--cost-management)
- **Part V — Security & Compliance**
  - [45. Threat Model & STRIDE](#45-threat-model--stride)
  - [46. AuthN/AuthZ](#46-authnauthz)
  - [47. RBAC Matrix](#47-rbac-matrix)
  - [48. API Keys & Token Management](#48-api-keys--token-management)
  - [49. Secrets Vault & Encryption](#49-secrets-vault--encryption)
  - [50. Webhook Security](#50-webhook-security)
  - [51. Workflow & Code Sandbox Security](#51-workflow--code-sandbox-security)
  - [52. AI Safety & Guardrails](#52-ai-safety--guardrails)
  - [53. Compliance Roadmap](#53-compliance-roadmap)
  - [54. Privacy & Data Residency](#54-privacy--data-residency)
  - [55. Vulnerability Management & SDLC](#55-vulnerability-management--sdlc)
- **Part VI — Growth & Ecosystem**
  - [56. Marketplace Architecture](#56-marketplace-architecture)
  - [57. Creator Economy Mechanics](#57-creator-economy-mechanics)
  - [58. Template Manifest Spec](#58-template-manifest-spec)
  - [59. Marketplace Review & Moderation](#59-marketplace-review--moderation)
  - [60. Pricing & Billing Implementation](#60-pricing--billing-implementation)
  - [61. Onboarding & Activation](#61-onboarding--activation)
  - [62. SEO & Content Strategy](#62-seo--content-strategy)
  - [63. Analytics Implementation](#63-analytics-implementation)
  - [64. A/B Testing & Feature Flags](#64-ab-testing--feature-flags)
  - [65. Customer Support Tooling](#65-customer-support-tooling)
- **Part VII — Build Plan**
  - [66. Roadmap (Phase 0–6)](#66-roadmap-phase-0-6)
  - [67. Phase Deliverables](#67-phase-deliverables)
  - [68. Detailed Epic & Story Breakdown](#68-detailed-epic--story-breakdown)
  - [69. MVP Cut Lines](#69-mvp-cut-lines)
  - [70. DoR / DoD / Quality Gates](#70-dor--dod--quality-gates)
  - [71. ADRs to Write](#71-adrs-to-write)
  - [72. Team Sizing & RACI](#72-team-sizing--raci)
  - [73. Hiring & Contractor Plan](#73-hiring--contractor-plan)
- **Part VIII — Risks & Quality**
  - [74. Risk Register](#74-risk-register)
  - [75. Failure Modes & Rollback](#75-failure-modes--rollback)
  - [76. Test Pyramid & Strategy](#76-test-pyramid--strategy)
- **Part IX — Reference**
  - [77. Glossary](#77-glossary)
  - [78. Recommended Templates v1](#78-recommended-templates-v1)
  - [79. References & Sources](#79-references--sources)
  - [80. Appendix A — Sample DDL](#80-appendix-a--sample-ddl)
  - [81. Appendix B — Hono Route + RPC Client](#81-appendix-b--hono-route--rpc-client)
  - [82. Appendix C — Sample `template.yaml`](#82-appendix-c--sample-templateyaml)
  - [83. Appendix D — Sample Workflow JSON](#83-appendix-d--sample-workflow-json)
  - [84. Appendix E — Environment Variables](#84-appendix-e--environment-variables)
  - [85. Appendix F — Pre-Launch & Launch Checklist](#85-appendix-f--pre-launch--launch-checklist)
  - [86. Appendix G — RFC / ADR Template](#86-appendix-g--rfc--adr-template)
- **Part X — Build Walkthrough (Senior Fullstack Playbook)**
  - [87. Pre-Flight Checklist](#87-pre-flight-checklist-sebelum-mulai-coding)
  - [88. Phase 0 Walkthrough — Foundation](#88-phase-0-walkthrough--foundation-minggu-14)
  - [89. Phase 1 Walkthrough — Core Workflow + AI](#89-phase-1-walkthrough--core-workflow--ai-minggu-510)
  - [90. Phase 2 Walkthrough — Visual Builder + Sandbox](#90-phase-2-walkthrough--visual-builder--sandbox-minggu-1116)
  - [91. Phase 3 Walkthrough — Marketplace + Creator](#91-phase-3-walkthrough--marketplace--creator-economy-minggu-1722)
  - [92. Phase 4 Walkthrough — Scale, SLA, Enterprise](#92-phase-4-walkthrough--scale-sla-enterprise-minggu-2334)
  - [93. Phase 5 Walkthrough — AI Native + Verticals](#93-phase-5-walkthrough--ai-native-ux--vertical-templates-minggu-3546)
  - [94. Phase 6 Walkthrough — Enterprise + Compliance Plus](#94-phase-6-walkthrough--enterprise--compliance-plus-minggu-4760)
  - [95. Final Cutover & Public Launch Playbook](#95-final-cutover--public-launch-playbook)
  - [96. Day-2 Operations](#96-day-2-operations-setelah-final--steady-state)
  - [97. Common Pitfalls Per Phase](#97-common-pitfalls-per-phase--mitigation)
  - [98. Local Dev → Prod Promotion Checklist](#98-local-dev--prod-promotion-checklist-per-story)
  - [99. Senior-Eng Decision Cheatsheet](#99-senior-eng-decision-cheatsheet-quick-reference)
- [Changelog v1 → v2 → v2.1](#changelog-v1--v2)

---

## 0. Executive Summary

TESKEL adalah **AI Startup Factory + Infrastructure Platform** — operating
system untuk membangun, meluncurkan, memonetisasi, dan menskalakan bisnis
AI-native. TESKEL bukan website builder, bukan AI wrapper, bukan marketplace
template biasa, dan bukan klon n8n / Vercel.

Empat pilar produk:

1. **Build** — AI SaaS, workflow automation, dan AI agent dari template,
   visual builder, dan AI commands.
2. **Deploy** — publish app/workflow/agent ke production dengan domain, SSL,
   secrets, dan rollback otomatis.
3. **Monetize** — subscription, usage-based AI billing, marketplace template
   sales, revenue sharing untuk creator.
4. **Scale** — analytics, observability, AI cost control, multi-region infra,
   plugin & SDK ecosystem.

Promise utama:

> **From idea to deployed AI business in under 10 minutes.**

Plan v2 ini merombak v1 dengan:

- Memilih satu jawaban untuk tiap *fork in the road* (auth, ORM, queue, dll.)
  dengan rasionalisasi tertulis dan ADR.
- Menambahkan modul ops & compliance yang sebelumnya hilang (SLO, runbooks,
  DR, SOC2 path, FinOps).
- Memberikan *deliverables*, *DoD*, dan *cut lines* per fase agar tim bisa
  langsung eksekusi tanpa interpretasi ulang.
- Menambahkan appendix konkret (DDL, manifest, workflow JSON, env vars,
  launch checklist, RFC template).

---

## 1. Konvensi Dokumen, Notasi, Status

### 1.1 Status Tag

- `DECIDED` — keputusan final, ubah hanya via ADR baru.
- `DEFAULT` — pilihan default; boleh diganti jika ada *spike* yang
  menggugurkan asumsi.
- `OPEN` — belum diputuskan, butuh riset/spike.
- `LATER` — di-defer eksplisit, jangan dibahas di MVP.

### 1.2 Priority Tag

`P0` (must, MVP), `P1` (should, post-MVP), `P2` (could), `P3` (won't, this year).

### 1.3 Confidence

`(C: high)` keputusan kuat, `(C: med)` masih ada ketidakpastian, `(C: low)`
hipotesis kerja.

### 1.4 Definisi Singkat

- **MVP** = Phase 1 selesai (bukan Phase 0).
- **GA** = Generally Available, public sign-up tanpa waitlist.
- **Tenant** = `Organization` (bukan user).
- **Workspace** = sinonim `Project` di UI.

---

## 2. North Star, Mission, Vision, OKR

### 2.1 Mission

Memberi siapa pun cara tercepat dan teraman untuk membangun, men-deploy, dan
memonetisasi bisnis AI-native.

### 2.2 Vision

5 tahun ke depan, mayoritas AI SaaS baru yang launch berasal dari TESKEL atau
ekosistemnya — sebagaimana mayoritas web app modern di-deploy via Vercel atau
turunan Next.js.

### 2.3 North Star Metric

**Weekly Successfully Deployed AI Products (WSDAP)**

Definisi: jumlah unique `Project` yang sukses deploy ≥1 kali dalam 7 hari
terakhir, dengan health check pass dan minimal 1 production AI request.

### 2.4 OKR Tahun 1 (sample)

| Objective | Key Result |
|---|---|
| Bukti *time-to-deploy* < 10 menit | Median `signup → first deploy` ≤ 10 menit |
| Bukti retensi produk | W4 retention pengguna aktif ≥ 30% |
| Marketplace bermutu | ≥ 50 template berkualitas, NPS install ≥ 40 |
| AI cost terkendali | Gross margin AI usage ≥ 55% |
| Reliabilitas | Uptime API & worker ≥ 99.9%, Sev1 < 1/kuartal |

### 2.5 Leading vs Lagging

- **Leading**: signup completion %, time-to-first-AI-call,
  time-to-first-deploy, template install conversion, workflow first-run
  success.
- **Lagging**: MRR, churn, marketplace GMV, gross margin, NPS, CAC payback.

---

# Part I — Strategy & Product

## 3. Product Positioning

### 3.1 One-Liner

> TESKEL helps founders, creators, and developers launch AI-native businesses
> from templates, workflows, agents, and deployable infrastructure.

### 3.2 Category Map

| Kategori | Sudut TESKEL |
|---|---|
| AI App Builder | Klon SaaS dari template, edit visual + AI |
| Workflow Automation | Visual + agentic workflow + durable execution |
| SaaS Boilerplate Platform | Auth, billing, deploy, dashboard out-of-the-box |
| Creator Marketplace | Template, workflow, agent, plugin, theme economy |
| Deployment Platform | Containerized publish, custom domain, SSL, rollback |
| AI Infrastructure Layer | Model routing, prompt registry, RAG, eval, obs |

### 3.3 Anti-Positioning

TESKEL **bukan**:

- Builder website non-AI generic (Webflow/Framer).
- Wrapper tipis di atas satu LLM.
- Forum jualan template tanpa runtime.
- Klon n8n (workflow tanpa monetisasi/deploy app).
- Klon Vercel (deploy tanpa template + AI billing).
- Replit/Bolt (in-browser code-first IDE, bukan launch-first).

### 3.4 Wedges (Initial GTM)

Mulai dengan 2 wedge, jangan lebih:

1. **Wedge A — "Launch AI SaaS dari template"** (target: solo founder & indie
   hacker). KPI: time-to-first-deploy.
2. **Wedge B — "Launch AI workflow yang bisa dijual"** (target: AI builder &
   agency). KPI: time-to-first-paid-run.

Wedge marketplace creator economy dijalankan paralel namun *demand-driven*
dari Wedge A & B.

---

## 4. ICP & Persona Deep Dive

### 4.1 Persona Inti

| Persona | Goal | Pain Lama | TESKEL Win |
|---|---|---|---|
| **Solo Founder Aira** | Validasi ide AI SaaS dalam 1 minggu | Setup auth/billing/deploy makan 2 minggu | Template + 1-click deploy |
| **AI Builder Bima** | Jual workflow & agent | n8n self-host, susah monetize | Marketplace + revenue share |
| **Agency Owner Citra** | Deliver AI ke klien tanpa rebuild | Tiap klien rebuild dari nol | Multi-org, white-label, template fork |
| **Creator Dimas** | Income passive dari template | Itch.io tidak punya runtime | Template + install + payout |
| **Senior Dev Eka** | Production-ready start point | Boilerplate basi, AI bolt-on | TESKEL repo modular AI-native |

### 4.2 Jobs-To-Be-Done

- *When* saya punya ide AI tool, *I want to* lihat working clone dalam 10
  menit, *so I can* memutuskan apakah serius dilanjutkan.
- *When* saya bangun workflow yang bagus, *I want to* publish jadi product
  berbayar, *so I can* dapat revenue tanpa rebuild infrastruktur.
- *When* AI cost meningkat, *I want to* lihat per-tenant per-model breakdown,
  *so I can* fix margin sebelum bleeding.

### 4.3 Anti-Persona

- Enterprise dengan kebutuhan SSO/VPC isolation hari pertama.
- Pengguna no-AI murni.
- Pengguna yang butuh klon Figma/Notion penuh.

---

## 5. Competitive Landscape & Moat

### 5.1 Map

| Pesaing | Strength | Weakness vs TESKEL |
|---|---|---|
| **Bolt.new / Lovable** | In-browser code agent | Tidak punya marketplace + monetize |
| **Replit Agent** | Eksekusi code | Bukan AI SaaS template, billing native lemah |
| **n8n / Make / Zapier** | Workflow mature | Tidak punya AI SaaS deploy + marketplace |
| **Vercel + AI SDK** | Infra kelas dunia | Tidak punya template marketplace + workflow runtime |
| **Retool** | App builder | Internal-tool oriented |
| **Pinecone/LangSmith/etc** | AI infra spesifik | Bukan platform end-to-end |
| **Gumroad/Lemon Squeezy** | Marketplace mature | Tidak punya runtime/deploy |
| **Dify / Coze / Flowise** | Open AI workflow | Tidak ada SaaS template marketplace + payout |

### 5.2 Moat (progresif)

1. **Template ecosystem** *production-grade* (bukan demo).
2. **Workflow runtime durable** + AI cost & guardrails terintegrasi.
3. **AI billing rail** menyatukan Stripe + token metering + payout.
4. **Creator economy network effects** (creator → user → creator).
5. **Deployment pipeline** *opinionated* tapi reliable.
6. **Data flywheel**: prompt/workflow eval data dari semua tenant.

### 5.3 What Could Kill Us

- Bolt/Lovable menambahkan marketplace + Stripe Connect.
- Vercel meluncurkan "AI SaaS Templates" first-class.
- Open-source alternatif (Dify/Coze/Trigger.dev) menggabungkan deploy +
  marketplace.

Mitigasi: kecepatan iterasi + creator lock-in via revenue share & data.

---

## 6. Business Model & Pricing

`DECIDED` — kombinasi seat + usage + marketplace commission.

### 6.1 Plan Tiers (sketsa awal, validasi pricing test di Phase 1)

| Tier | Seat / mo | Inclusive AI Credits | Workflow Runs | Deploys | Marketplace |
|---|---|---|---|---|---|
| **Free** | $0 | $1 | 100 | 1 | install only |
| **Starter** | $19 / seat | $10 | 2,000 | 3 | install only |
| **Pro** | $49 / seat | $30 | 10,000 | 10 | publish + sell |
| **Team** | $129 / seat | $90 | 50,000 | 25 | publish + sell + bulk |
| **Enterprise** | custom | custom | custom | custom | custom + SSO + SLA |

Overage: AI credits & workflow runs di-meter via Stripe meter events.

### 6.2 Marketplace Take Rate

- Creator: **80%** default, **85%** untuk Verified Creator (≥ $5k GMV/yr).
- Refund window: 7 hari, full refund, dengan moderation arbitration.
- Payout: minimum $25, monthly via Stripe Connect Standard.

### 6.3 Margin Targets

- AI usage gross margin ≥ 55% (markup 1.8–2.2× cost provider).
- Infra gross margin ≥ 70% (deploy + storage + workflow compute).
- Marketplace contribution margin ≥ 90%.

### 6.4 Discounting

- Annual: 20% off.
- Startup program: 50% off 1 tahun untuk pre-seed.
- Creator program: gratis Pro untuk creator dengan ≥ $1k GMV/bulan.

---

## 7. KPI Framework

### 7.1 Funnel Inti

```text
Visit → Signup → Org Created → First AI Call → First Deploy → Billing
   → Paid → Retained (W4) → Expansion → Marketplace Publish (creator path)
```

### 7.2 Metric Inti per Tahap

| Tahap | Metric Wajib | Target Awal |
|---|---|---|
| Visit→Signup | Signup CR | ≥ 6% |
| Signup→Org | Org create CR | ≥ 80% |
| Org→AI Call | TT-First-AI ≤ 5 mnt | ≥ 70% |
| AI→Deploy | TT-First-Deploy ≤ 10 mnt | ≥ 50% |
| Deploy→Billing | Paid CR W2 | ≥ 8% |
| Paid→Retained | W4 retention | ≥ 30% |
| Creator | Publish→First Install ≤ 7 hari | ≥ 60% |

### 7.3 Health Metrics

- Workflow first-run success rate.
- AI request error rate per provider.
- Worker queue lag p95.
- Deployment success rate.
- Cost per active org (AI + infra) vs revenue per active org.

---

## 8. Hypotheses & Research Backlog

`DEFAULT` (semua harus diuji di Phase 1):

1. *Pengguna memilih template dibanding scratch* jika onboarding ≤ 10 menit.
2. *Creator akan publish* jika revenue share + payout reliable.
3. *Workflow + AI agent bisa di-monetize* lewat per-run pricing.
4. *AI billing transparan* mengurangi churn vs flat unlimited.
5. *Coolify cukup* untuk 0–500 paying orgs sebelum perlu K8s.

Research log disimpan di Notion/Linear `RES-###` dengan template:

```text
Hypothesis: ...
Test: ...
Owner: ...
Decision date: ...
Result: validated | refuted | inconclusive
Action: ...
```

---

# Part II — Design

## 9. Design Philosophy

| Trait | Style |
|---|---|
| Modern | Minimal, sharp, current |
| Fast | Low friction, instant feedback |
| Premium | Polished, thoughtful, subtle |
| Technical | Developer-native, clear data model |
| Clean | Uncluttered, readable |
| Global | SaaS-grade, not local-only aesthetic |
| AI-Native | Futuristic but simple |

Reference set:

- **Vercel** — clean dev platform, deploy-first UX.
- **Linear** — keyboard-first, polish.
- **Framer** — visual builder quality, motion.
- **Supabase** — developer dashboard & DB UX.
- **Notion** — modular blocks, creation flow.
- **Raycast** — command palette, action-first.
- **Retool** — app/workflow builder patterns.
- **n8n** — workflow automation mental model.
- **Resend** — dev DX untuk SaaS.

Design keywords: *calm interface, precise layout, high whitespace, micro
animation, clear hierarchy, no decorative noise, strong empty states,
keyboard-first, command-driven*.

---

## 10. Design System Spec

### 10.1 Tokens

`DECIDED` — design tokens W3C-style; sumber kebenaran di
`packages/ui/tokens/tokens.json`. Tailwind config dan CSS variables di-generate
dari sumber ini (Style Dictionary).

### 10.2 Typography

- **Geist Sans** untuk UI (`font-sans`).
- **Geist Mono** untuk code, logs, payload, API keys (`font-mono`).
- Skala: `xs 12 / sm 13 / base 14 / md 16 / lg 18 / xl 20 / 2xl 24 / 3xl 30 /
  4xl 36 / 5xl 48`.
- Weight: 400 default, 500 emphasis, 600 heading, 700 hero only.

### 10.3 Color (Light-first, Dark-ready)

| Area | Light | Dark |
|---|---|---|
| Background | `white` | `zinc-950` |
| Surface | `zinc-50` | `zinc-900` |
| Elevated | `white` | `zinc-900` (border) |
| Border | `zinc-200` | `zinc-800` |
| Muted Border | `zinc-100` | `zinc-800/50` |
| Text Primary | `zinc-950` | `zinc-50` |
| Text Secondary | `zinc-600` | `zinc-400` |
| Text Muted | `zinc-400` | `zinc-500` |
| Accent | `violet-600` | `violet-400` |
| Success | `emerald-600` | `emerald-400` |
| Warning | `amber-500` | `amber-400` |
| Danger | `red-600` | `red-400` |

Kontras: minimal AA (4.5:1) untuk text di semua kombinasi.

### 10.4 Radius, Spacing, Shadow

- Radius: `lg=8px`, `xl=12px`, `2xl=16px`, `3xl=24px`. Default cards: `xl`.
- Spacing: Tailwind default; layout grid 4/8/12/16/24/32/48.
- Shadow: default border-only; hover `shadow-sm`; floating `shadow-md`.

### 10.5 Motion

- Duration: 120ms (micro), 200ms (default), 320ms (panel), 600ms (page).
- Easing: `cubic-bezier(.2,.8,.2,1)` enter; `cubic-bezier(.4,0,.2,1)` exit.
- Hormati `prefers-reduced-motion: reduce`.

### 10.6 Iconography

- **Lucide** untuk UI icons (1.5px stroke).
- **Tabler** atau custom set untuk node icons di workflow.
- Brand icons via `simple-icons`.

### 10.7 Components Source

- Baseline: **shadcn/ui** dengan Radix primitives.
- Ownership: copy ke `packages/ui` (jangan lock ke shadcn registry).
- Component manifest YAML di `packages/ui/manifest.yaml` (versi, deprecation,
  a11y notes).

### 10.8 Density Modes

`comfortable` (default) dan `compact`. User toggle di settings.

---

## 11. Information Architecture

### 11.1 Top-Level Navigation

```text
[Dashboard] [Projects] [Workflows] [Templates] [Marketplace] [Deployments]
[AI] [Billing] [Settings]
```

### 11.2 Dashboard (Org Home)

Active projects, recent deploys, AI usage sparkline, workflow runs sparkline,
top errors, marketplace earnings (jika creator).

### 11.3 Project Workspace

Tab: Overview, App, Workflows, AI, Data (KB), Deployments, Logs, Settings.
Builder full-screen dari tab App; workflow editor full-screen dari tab
Workflows.

### 11.4 Marketplace

Browse, Categories, Collections, Creator profile, Listing detail, Reviews.
Creator Studio: Drafts, Published, Analytics, Payouts, Reviews inbox.

### 11.5 Settings

General, Members, Roles, API Keys, Secrets, Domains, Billing, Audit Log,
Webhooks, Notifications, Compliance, Data export.

### 11.6 Command Palette (`Cmd/Ctrl-K`)

- Navigate to anywhere.
- Run AI command (generate workflow, explain workflow, fix node).
- Create resource (new project, new workflow, new template).
- Search docs.

---

## 12. Golden UX Flows

### 12.1 Flow A — Launch AI SaaS dari Template (≤ 10 menit)

1. Visit landing → CTA "Launch AI SaaS".
2. Signup (magic link / OAuth Google/GitHub) — 30 dtk.
3. Auto-create default `Organization` + `Project`.
4. Pilih template (default: AI Support Bot SaaS).
5. Wizard: brand (nama, logo, warna), AI provider (default OpenRouter +
   Claude Haiku), domain (subdomain `*.teskel.app`), Stripe (skip → trial).
6. Klik **Deploy** → progress bar (build → push → provision → SSL → health).
7. Success state: link `https://your-app.teskel.app` + tombol "Open" + tombol
   "Edit branding" + tombol "Connect Stripe".
8. Empty state checklist: 1) test AI, 2) connect billing, 3) custom domain.

Target: median 6–9 menit, p95 ≤ 12 menit.

### 12.2 Flow B — Build & Publish Workflow

1. Project → Workflows → New Workflow.
2. AI prompt: "Buat workflow lead enrichment dari email" → graph terisi.
3. Edit nodes (drag, configure).
4. Test run (panel bawah, logs per node).
5. Save version v1.
6. Toggle `Trigger`: webhook / schedule / manual.
7. Publish → workflow URL public (jika creator) atau internal-only.
8. Optional: List ke Marketplace (lihat 12.4).

### 12.3 Flow C — Customize Visual App

1. Project → App → Open Builder.
2. Layers panel (kiri), canvas (tengah), properties (kanan).
3. Drag block dari palette atau **AI command**: "Tambah testimonial section".
4. Theme editor (font, radius, accent).
5. Responsive preview.
6. Publish version.

### 12.4 Flow D — List ke Marketplace (Creator)

1. Creator Studio → New Listing.
2. Pilih template/workflow source.
3. Cover, screenshots, description (markdown), tags, demo URL.
4. Pricing (gratis / one-time / subscription via Stripe Connect).
5. Submit for review.
6. Review (auto + manual).
7. Approved → live.

### 12.5 Flow E — Install Template ke Project

1. Marketplace → Listing → Install.
2. Pilih project tujuan.
3. Review required env vars & integrations.
4. Confirm → template di-fork ke project.
5. Auto open onboarding wizard.

### 12.6 Flow F — Debug Failed Workflow

1. Notification → buka run.
2. Timeline view: node hijau / merah.
3. Klik node merah → input/output/log/error stack.
4. Tombol "Ask AI to fix" → AI patch suggest.
5. Apply patch → re-run from failed node (idempotent).

---

## 13. Empty / Error / Loading Patterns

### 13.1 Empty States

Heading + 1 kalimat penjelasan + ilustrasi minimal + 1 primary CTA + 1
secondary CTA (dokumentasi/video).

### 13.2 Error States

- *Permission*: jelaskan role yang dibutuhkan + link kontak admin.
- *Network*: tombol retry + status link.
- *AI provider*: nama provider + suggestion fallback.
- *Workflow run*: stack trace ringkas + tombol "Open in run inspector".

### 13.3 Loading

- Skeleton untuk konten list/card.
- Streaming untuk AI text (token streaming).
- Indeterminate untuk waktu < 2 dtk; progress untuk > 2 dtk.

---

## 14. Accessibility (WCAG 2.2 AA)

- Semua interactive: keyboard reachable + visible focus ring.
- ARIA pakai Radix primitives default; jangan reinvent.
- Form label wajib; error pakai `aria-describedby`.
- Color tidak boleh jadi satu-satunya state indicator.
- Min target size 24×24 (mobile 44×44).
- Screenreader: NVDA + VoiceOver test sebelum GA.
- Test tool: `axe-core` di CI (Playwright + `@axe-core/playwright`).

---

## 15. Internationalization

- `DECIDED` — i18n disiapkan dari Phase 0, GA Phase 1 = English-only.
- Library: `next-intl` (App Router-friendly).
- Locale awal: `en` (default), `id` (prioritas growth lokal).
- Format: `Intl.DateTimeFormat`, `Intl.NumberFormat`.
- RTL: di-support struktural, tidak diaktifkan sebelum demand.

# Part III — Architecture

## 16. Architecture Overview (C4-Light)

### 16.1 Context (Level 1)

```text
                +-------------------------+
                |          Users          |
                |  founders, creators,    |
                |  end-users dari app     |
                |  yang di-deploy         |
                +-----------+-------------+
                            |
                            v
+-------------------------------+      +------------------+
|        TESKEL Platform        |<---->|  Stripe          |
|  (web + api + worker + edge)  |      |  Connect/Billing |
+-------------------------------+      +------------------+
   |        |          |     |
   v        v          v     v
 Postgres  Redis  Object  AI Providers (OpenRouter, OpenAI,
                Storage  Anthropic, Google, self-host, Groq)
                  R2
                          ^
                          |
                +-------------------+
                |  Deployed Apps    |
                |  (per project)    |
                +-------------------+
```

### 16.2 Container View (Level 2)

| Container | Tech | Purpose |
|---|---|---|
| `apps/web` | Next.js 15 (App Router) | Marketing, Auth, Dashboard, Builder, Marketplace UI |
| `apps/api` | Hono on Node 22 | REST + RPC API, webhooks, billing, OAuth callbacks |
| `apps/worker` | Node 22 + BullMQ | Workflow runs, AI jobs, deploys, embeddings, emails |
| `apps/edge` (P1) | Cloudflare Workers | Webhook ingress, edge cache, AI gateway proxy |
| `apps/cli` | Node 22 + clipanion | `teskel` CLI |
| `packages/*` | Shared | UI, db, ai, workflow-engine, template-engine, sdk |
| **Postgres 16** | RDS/Neon/Supabase | Primary data, pgvector, RLS |
| **Redis 7** | Upstash/managed | Cache, BullMQ queue, pub/sub, rate limit |
| **R2** | Cloudflare | Files, exports, build artifacts |
| **Coolify VM** | Self-hosted | Deployed customer apps (MVP) |
| **PostHog** | Cloud/self-host | Analytics, feature flags |
| **Sentry** | Cloud | Errors |
| **Langfuse** | Self-host first | AI observability |
| **Inngest** (P1) | SaaS | Durable workflow execution alt |
| **E2B** (P1) | SaaS | Code-node sandbox |

### 16.3 Component View (Level 3) — Backend Modules

```text
apps/api
├── modules/
│   ├── auth/          (better-auth integration)
│   ├── org/           (tenants, members, invitations, RBAC)
│   ├── project/       (workspaces, env, secrets refs)
│   ├── ai/            (router, providers, prompts, eval)
│   ├── workflow/      (graph, version, run, runtime API)
│   ├── template/      (templates, versions, install)
│   ├── marketplace/   (listings, reviews, ratings, payouts)
│   ├── billing/       (Stripe sub, meter events, webhooks)
│   ├── deploy/        (build, push, domain, SSL, health)
│   ├── analytics/     (events, funnels, retention)
│   ├── usage/         (quotas, rate limit, cost meter)
│   ├── secrets/       (encrypted store, rotate)
│   ├── audit/         (audit log, export)
│   ├── webhook/       (in/out webhooks, signing)
│   ├── notification/  (email/in-app/Slack/discord)
│   └── api-keys/      (token mgmt, rotation, scopes)
```

### 16.4 Deployment Topology

```text
[Cloudflare DNS + WAF + R2]
        |
        v
[Coolify reverse proxy / Traefik]
        |
   +----+--------+-----------+
   |             |           |
[apps/web]  [apps/api]   [apps/worker x N]
   |             |           |
   +------+------+-----------+
          |
          v
   [Postgres primary]  [Postgres replica (P1)]
          |
   [Redis primary]
```

### 16.5 Architectural Principles

1. **Modular monolith first**, extract services hanya jika ada nyeri scale
   nyata.
2. **Type-safe end-to-end**: schema (Drizzle) → API (Hono RPC + Zod) →
   client (TanStack Query). No `any`, no `getattr`-style escapes.
3. **Event sourcing untuk billing & audit**, tidak untuk semua domain.
4. **Idempotency wajib** di endpoint mutasi & webhook ingestion.
5. **Outbox pattern** untuk konsistensi DB → queue → external.
6. **Tenant isolation di tiga lapis**: app code (org guard), DB (RLS),
   secrets (KMS scope).
7. **No silent failures** — semua error handler wajib log + propagate ke
   observability.
8. **Default secure** — fail-closed, deny-by-default, opt-in untuk fitur
   sensitif.

---

## 17. Monorepo & Build Tooling

### 17.1 Struktur

```text
teskel/
├── apps/
│   ├── web/                    # Next.js 15 frontend
│   ├── api/                    # Hono backend
│   ├── worker/                 # BullMQ worker
│   ├── edge/                   # Cloudflare Workers (P1)
│   ├── cli/                    # `teskel` CLI
│   └── docs/                   # Docusaurus / Mintlify
├── packages/
│   ├── ui/                     # Design system + shadcn
│   ├── tokens/                 # Design tokens
│   ├── config/                 # ESLint, Prettier, TS, Tailwind, Vitest
│   ├── db/                     # Drizzle schema + migrations
│   ├── shared/                 # Zod schemas, types, errors, utils
│   ├── ai/                     # Model router, providers, prompt engine
│   ├── workflow-engine/        # Graph compiler + runtime
│   ├── template-engine/        # Template manifest + installer
│   ├── billing/                # Stripe wrappers, meter helpers
│   ├── auth/                   # Better Auth wrapper + plugins
│   ├── sdk/                    # @teskel/sdk (npm publish)
│   └── eslint-config/          # Eslint preset
│   └── tsconfig/               # TS preset
├── tools/
│   ├── scripts/                # repo scripts (release, codegen)
│   └── docker/                 # docker-compose dev
├── infra/
│   ├── coolify/                # Coolify config
│   ├── traefik/                # reverse proxy
│   └── terraform/ (P2)         # IaC
├── .github/
│   └── workflows/              # CI/CD
├── turbo.json
├── package.json
├── pnpm-workspace.yaml
├── README.md
├── AGENTS.md                   # AI coding agent instructions
└── CONTRIBUTING.md
```

### 17.2 Tooling Pinned

| Area | Tool | Version Pin |
|---|---|---|
| Package Manager | **pnpm** | 9.x |
| Monorepo | **Turborepo** | 2.x |
| Node | **Node** | 22 LTS |
| TypeScript | **TS** | 5.6+, `strict: true` |
| Lint | **ESLint** | 9 (flat config) |
| Format | **Prettier** | 3.x |
| Git Hooks | **Lefthook** | latest |
| Test | **Vitest** + **Playwright** | latest |
| Commits | **Conventional Commits** + commitlint | — |
| Release | **Changesets** | latest |

### 17.3 ESLint / TS Preset

- `packages/eslint-config` extends: `eslint`, `@typescript-eslint`, `import`,
  `unicorn`, `tailwindcss`, `react`, `next`, `vitest`.
- `packages/tsconfig`:
  - `base.json`, `nextjs.json`, `node.json`, `react-library.json`.
  - `strict: true`, `noUncheckedIndexedAccess: true`,
    `verbatimModuleSyntax: true`.

### 17.4 Turborepo Pipelines

- `dev` — concurrent web/api/worker/docs.
- `build` — topo build dengan cache.
- `lint`, `typecheck`, `test:unit`, `test:int`, `test:e2e`.
- Remote cache di self-hosted S3-compatible (R2) lewat `turbo` env.

### 17.5 Code Ownership

`CODEOWNERS` dengan path-based assignment (lihat §72 untuk team mapping).

---

## 18. Frontend Architecture

### 18.1 Stack

| Area | Tech | Status |
|---|---|---|
| Framework | Next.js 15 (App Router, Server Components) | `DECIDED` |
| UI Runtime | React 19 | `DECIDED` |
| Styling | Tailwind CSS 4 | `DECIDED` |
| Components | shadcn/ui (di-vendor) + Radix primitives | `DECIDED` |
| Icons | Lucide | `DECIDED` |
| Forms | React Hook Form | `DECIDED` |
| Validation | Zod | `DECIDED` |
| Client State | Zustand | `DECIDED` |
| Server State | TanStack Query | `DECIDED` |
| Tables | TanStack Table v8 | `DECIDED` |
| Charts | Recharts (default) + custom Tremor-style | `DEFAULT` |
| Command | `cmdk` | `DECIDED` |
| Animations | Motion (formerly Framer Motion) | `DECIDED` |
| Workflow Canvas | `@xyflow/react` (React Flow) | `DECIDED` |
| Visual Builder | **Puck** (default) — fallback ke Craft.js bila butuh kontrol penuh | `DECIDED` |
| Real-time | Liveblocks (text & presence) + Yjs (workflow CRDT) | `DEFAULT` |
| Streaming AI | Vercel **AI SDK v5** (`ai`, `@ai-sdk/*`) | `DECIDED` |
| i18n | `next-intl` | `DECIDED` |

### 18.2 Folder Structure (`apps/web`)

```text
apps/web/
├── src/
│   ├── app/
│   │   ├── (marketing)/
│   │   ├── (auth)/
│   │   ├── (dashboard)/
│   │   ├── (builder)/        # full-screen builder
│   │   ├── (workflow)/       # full-screen workflow editor
│   │   └── api/              # Next route handlers (BFF, webhooks UI)
│   ├── components/
│   │   ├── ui/               # design-system imports
│   │   ├── layout/
│   │   └── shared/
│   ├── features/             # feature-based modules
│   │   ├── auth/
│   │   ├── dashboard/
│   │   ├── projects/
│   │   ├── templates/
│   │   ├── workflows/
│   │   ├── builder/
│   │   ├── ai/
│   │   ├── billing/
│   │   ├── marketplace/
│   │   ├── deployments/
│   │   └── settings/
│   ├── hooks/
│   ├── store/                # zustand
│   ├── lib/                  # api client, query client, utils
│   ├── styles/
│   └── env.ts                # @t3-oss/env-nextjs
├── next.config.ts
├── tailwind.config.ts
└── tsconfig.json
```

### 18.3 Server vs Client

- Default: Server Components untuk read-paths.
- Client Components hanya untuk interaktivitas (form, builder, charts
  realtime).
- Mutations: Server Actions untuk form sederhana; Hono RPC + TanStack Query
  untuk yang kompleks.
- Streaming responses (AI chat) via Edge runtime di route handler.

### 18.4 Data Fetching

- `apps/web/src/lib/api.ts` expose Hono RPC client `hc<AppType>(...)`.
- Wrap di `useApi` hooks per resource (TanStack Query).
- Cache key berdasarkan tenant id + resource id.
- Optimistic updates untuk action ringan (rename, toggle).

### 18.5 Routing & Layout

- Group routes: marketing, auth, dashboard, builder, workflow.
- Layout per group; `dashboard` punya sidebar persistent.
- `loading.tsx` skeleton + `error.tsx` recovery.

### 18.6 SEO

- `metadata` API per page; OG image generated via `next-og`.
- Sitemap via `app/sitemap.ts`; robots di `app/robots.ts`.
- Structured data JSON-LD untuk marketplace listing & blog.

### 18.7 Performance Budgets

- LCP ≤ 2.0s p75 di marketing.
- TTFB ≤ 200ms p75 untuk authenticated routes.
- JS shipped per route ≤ 180KB gzipped (App routes).
- Lighthouse perf ≥ 90 (mobile) untuk marketing.

---

## 19. Backend Architecture

### 19.1 Stack

| Area | Tech | Status |
|---|---|---|
| Runtime | Node 22 LTS (Bun di evaluasi P1) | `DECIDED` |
| Framework | **Hono** | `DECIDED` |
| Validation | Zod (+ `@hono/zod-validator`) | `DECIDED` |
| ORM | **Drizzle ORM** (Postgres) | `DECIDED` |
| Migrations | Drizzle Kit | `DECIDED` |
| API | REST + Hono RPC (type-safe internal) + OpenAPI generated | `DECIDED` |
| Auth | **Better Auth** (organizations, RBAC, passkeys, 2FA, magic link) | `DECIDED` |
| Queue | **BullMQ** (default) + Inngest (durable AI workflow) | `DECIDED` |
| Cache | Redis 7 (Upstash dev / managed prod) | `DECIDED` |
| Object Storage | Cloudflare R2 | `DECIDED` |
| Email | Resend (transactional) | `DECIDED` |
| Payments | Stripe + Stripe Connect Standard | `DECIDED` |
| AI | Vercel AI SDK v5 + custom router | `DECIDED` |
| Observability | OpenTelemetry → Grafana (logs, metrics, traces) + Sentry + Langfuse | `DECIDED` |

### 19.2 Why These Choices (rasionalisasi singkat)

- **Hono** — modern, edge-ready, RPC type-safety, OpenAPI middleware.
- **Drizzle vs Prisma** — Drizzle dipilih sebagai *DECIDED* karena: SQL-first
  (cocok untuk Postgres RLS), bundle ringan, kontrol query, RLS support
  resmi, edge-friendly. Prisma masih jadi *fallback* untuk tim yang lebih
  familiar (lihat ADR-002).
- **Better Auth** — TypeScript-native, plugin organizations + RBAC + passkeys
  + 2FA + magic link bawaan, self-hosted, no per-MAU cost.
- **BullMQ + Inngest** — BullMQ untuk job sederhana & deploy yang sudah ada
  Redis. Inngest untuk durable AI workflow agar tidak reinvent retry/step
  semantics. ADR-006 mendokumentasikan pemisahan workload.
- **OpenRouter** dipakai sebagai AI gateway default (banyak model 1 API),
  tapi router internal dibuat agar provider tidak lock-in.

### 19.3 Folder Structure (`apps/api`)

```text
apps/api/src/
├── modules/                   # domain modules (lihat 16.3)
├── services/                  # cross-module services
│   ├── ai/                    # gateway, router, prompt registry
│   ├── billing/
│   ├── deploy/
│   ├── workflow/
│   └── notification/
├── lib/
│   ├── db.ts                  # drizzle client + RLS context
│   ├── redis.ts
│   ├── r2.ts
│   ├── stripe.ts
│   ├── logger.ts              # pino
│   ├── otel.ts
│   ├── error.ts               # AppError + http mapping
│   └── env.ts
├── middleware/
│   ├── auth.ts
│   ├── tenant.ts              # set RLS session var
│   ├── rate-limit.ts
│   ├── idempotency.ts
│   ├── audit.ts
│   ├── request-id.ts
│   └── cors.ts
├── routes/
│   ├── v1/                    # versioned REST routes
│   └── webhooks/
├── jobs/                      # job definitions (queued from api, run in worker)
├── openapi/                   # generated openapi spec
└── server.ts
```

### 19.4 Modular Monolith Rules

- Modul TIDAK saling import code lain langsung kecuali via:
  - Service interfaces (`packages/shared/contracts`).
  - Domain events (in-process bus + persisted outbox).
- Cross-module DB writes harus melewati service layer.
- Setiap modul punya `*.routes.ts`, `*.service.ts`, `*.repo.ts`,
  `*.schema.ts`, `*.types.ts`.

### 19.5 Error Handling

- `AppError` class dengan kode (`UNAUTHORIZED`, `FORBIDDEN`, `NOT_FOUND`,
  `CONFLICT`, `RATE_LIMITED`, `BAD_REQUEST`, `UPSTREAM_ERROR`,
  `INTERNAL`).
- Error mapping → HTTP status + redacted user-safe message + request-id.
- Sentry tag: `module`, `org_id`, `user_id`, `request_id`.

### 19.6 Idempotency

- Header `Idempotency-Key` (UUID) di semua POST/DELETE mutasi billing,
  webhook send, deploy.
- Storage: Redis 24h + Postgres 14d (recovery).
- Webhook in-bound: dedupe by `provider:event_id`.

### 19.7 Outbox & Domain Events

- Tabel `outbox(id, aggregate, type, payload, headers, status, scheduled_at,
  attempts, processed_at)`.
- Worker mem-poll outbox + publish ke BullMQ / external (Slack/webhook).
- Memastikan DB transaction + side-effect konsisten.

---

## 20. API Design

### 20.1 Style

- REST untuk public API & webhook (stabil, mudah debug).
- Hono RPC untuk internal client (web/CLI/sdk) — type-safety end-to-end.
- OpenAPI 3.1 di-generate (lihat §81 sampel).
- gRPC/`tRPC` `LATER` (tidak diperlukan saat mod monolith).

### 20.2 Versioning

- Path-based: `/v1/...`, `/v2/...`.
- Deprecation: header `Sunset`, `Deprecation`, plus changelog di docs.
- Internal RPC tidak di-version (re-deploy bersama web).

### 20.3 Convention

- Resource plural: `/v1/projects`, `/v1/projects/:id/workflows`.
- IDs: ULID (sortable, URL-safe). Prefix per resource: `org_`, `prj_`,
  `wf_`, `tpl_`, `dpl_`, `run_`, `usr_`, `key_`.
- Pagination: cursor-based (`?cursor=...&limit=...`), max 100.
- Filter: query params standar (`?status=running&since=...`).
- Errors: RFC 9457 (Problem Details) JSON.

### 20.4 Auth

- Session cookie untuk web (Better Auth).
- API key (`Authorization: Bearer tk_live_...`) untuk public API.
- OAuth (P1) untuk integrasi 3rd-party.

### 20.5 Rate Limiting

- Global: 60 req/sec per IP.
- Auth: 10 req/sec per user.
- AI endpoints: per-org token-bucket, configurable per plan.
- Webhook ingestion: 100 req/sec per source.
- Implementasi: Redis sliding window.

### 20.6 Public API Surface (MVP)

- Projects CRUD.
- Workflows CRUD + `runs`.
- Templates list/install.
- Deployments list/trigger/rollback.
- AI requests (proxy ke router).
- Usage / billing read-only.

Webhooks out:

- `deployment.succeeded`, `deployment.failed`.
- `workflow.run.succeeded`, `workflow.run.failed`.
- `template.installed`, `template.published`.
- `marketplace.payout.completed`.
- `billing.subscription.updated`, `billing.usage.threshold.crossed`.

### 20.7 OpenAPI

- Generated otomatis dari Hono routes via `hono-zod-openapi` /
  `@hono/zod-openapi`.
- Live docs di `/docs/api` (Scalar atau Redocly).
- Diff CI check antar PR (oasdiff).

---

## 21. Database & Data Modeling

### 21.1 Engine

- **PostgreSQL 16** (managed: Neon awal, RDS/Supabase opsional).
- Extensions: `pgcrypto`, `uuid-ossp`, `pgvector`, `pg_trgm`, `citext`,
  `pg_partman` (P1 untuk audit log).

### 21.2 ORM & Migration

- Drizzle ORM dengan `drizzle-kit` migrations (commit ke repo).
- Naming: snake_case, plural tables, `created_at` & `updated_at` ts UTC,
  `deleted_at` soft-delete optional.
- Foreign keys eksplisit, cascading hati-hati (lihat 21.7).

### 21.3 Core Tables (excerpt)

```text
users(id ulid pk, email citext unique, ...)
organizations(id ulid pk, slug unique, billing_customer_id, ...)
organization_members(org_id, user_id, role, status, invited_by, ...)
projects(id ulid pk, org_id fk, name, slug, status, ...)
project_environments(id, project_id, name, vars jsonb, ...)
api_keys(id, org_id, hashed_secret, prefix, scopes, last_used, ...)
secrets(id, org_id, project_id, name, ciphertext, key_version, ...)
audit_logs(id, org_id, actor_id, action, entity, entity_id, payload, ip, ua, ts)

workflows(id, project_id, name, ...)
workflow_versions(id, workflow_id, version, graph jsonb, published_at, ...)
workflow_runs(id, workflow_id, version_id, trigger, input jsonb,
              output jsonb, status, started_at, finished_at, cost_cents, ...)
workflow_run_logs(id, run_id, node_id, level, msg, payload jsonb, ts)

templates(id, owner_id, slug, status, ...)
template_versions(id, template_id, version, manifest jsonb, signed_hash, ...)
template_installs(id, template_id, target_project_id, version, ...)

marketplace_listings(id, template_id, price_cents, currency,
                     pricing_model, status, ...)
reviews(id, listing_id, author_id, rating, body, ...)
ratings_aggregate(listing_id, avg_rating, count, ...)
creator_profiles(user_id, handle, kyc_status, stripe_account_id, ...)
creator_payouts(id, user_id, period, amount_cents, status, ...)
licenses(id, listing_id, buyer_org_id, valid_from, valid_to, ...)

deployments(id, project_id, version, status, image_ref, domain,
            health_url, started_at, succeeded_at, ...)

ai_requests(id, org_id, project_id, provider, model, prompt_hash,
            input_tokens, output_tokens, cost_cents, latency_ms,
            status, trace_id, ts)
prompt_templates(id, org_id, name, version, body, vars jsonb, ...)
prompt_evals(id, prompt_id, suite, score, traces jsonb, ts)

knowledge_bases(id, project_id, name, ...)
documents(id, kb_id, source, mime, storage_key, status, ...)
document_chunks(id, doc_id, idx, text, tokens, embedding vector(1536), ...)
agent_memories(id, project_id, scope, key, value, expires_at, ...)

subscriptions(id, org_id, stripe_id, plan, status, current_period_end, ...)
usage_logs(id, org_id, metric, value, ts)
meter_events(id, stripe_event_id, org_id, name, value, ts, posted_at)

webhooks(id, org_id, url, secret, events[], status, ...)
webhook_deliveries(id, webhook_id, event_id, attempts, status,
                   response_code, body, ts)

outbox(id, aggregate, type, payload, headers, status, scheduled_at,
       attempts, processed_at)
```

Lihat **Appendix A — Sample DDL** untuk DDL Drizzle yang lebih lengkap.

### 21.4 Index Strategy

- B-tree pada FK + timestamps utama.
- GIN pada `jsonb` filter umum (`workflow_versions.graph`,
  `audit_logs.payload`).
- `pg_trgm` GIN untuk full-text search marketplace listing.
- pgvector: HNSW (`m=16, ef_construction=64`), cosine distance.
- Partial index: `WHERE status = 'active'` untuk hot tables.

### 21.5 Soft Delete vs Hard Delete

- Default soft-delete (`deleted_at`) untuk entitas user-owned
  (workflows, templates, projects).
- Hard-delete untuk `*_logs` di-arsipkan ke R2 (lihat retention 42).
- GDPR delete: hard-delete personal fields, anonymize FK rows.

### 21.6 Migrations

- Commit semua migration; review di PR.
- *Never* break-change schema tanpa expand-contract:
  1) tambahkan kolom baru,
  2) backfill,
  3) deploy code dual-read,
  4) deploy code single-read,
  5) drop kolom lama.
- Lock timeout di prod (`SET lock_timeout = '5s'`).

### 21.7 Cascading

- `ON DELETE CASCADE` HANYA untuk anak yang tidak boleh exist tanpa parent
  (e.g., `workflow_versions` → `workflows`).
- `ON DELETE RESTRICT` untuk billing/audit (jangan auto-delete).
- `ON DELETE SET NULL` untuk relasi opsional.

---

## 22. Multi-Tenancy & RLS

### 22.1 Tenant = `Organization`

- Semua tabel domain memiliki `org_id` (kecuali user-global tables).
- Service layer **wajib** memvalidasi org membership sebelum query.
- DB layer **wajib** memvalidasi via RLS sebagai *defense in depth*.

### 22.2 Postgres RLS

- Aktifkan RLS pada semua tabel tenant-scoped:
  ```sql
  ALTER TABLE projects ENABLE ROW LEVEL SECURITY;
  CREATE POLICY org_isolation ON projects
    USING (org_id = current_setting('app.current_org', true)::uuid);
  ```
- Policy untuk `INSERT` dan `UPDATE` dengan `WITH CHECK`.
- Setiap koneksi/transaksi men-set `app.current_org` & `app.current_user`
  sebelum query (Drizzle middleware).

### 22.3 Drizzle Integration

- Wrapper `db.withTenant(orgId, userId, fn)`:
  ```ts
  await db.transaction(async (tx) => {
    await tx.execute(sql`SET LOCAL app.current_org = ${orgId}`);
    await tx.execute(sql`SET LOCAL app.current_user = ${userId}`);
    return await fn(tx);
  });
  ```
- API middleware `tenant.ts` memanggil ini per request.
- Worker job: tenant context ada di payload, di-set saat mulai handler.

### 22.4 Cross-Tenant Operations

- Hanya allowed via `service_role` connection (admin tools).
- Audit log wajib untuk semua query cross-tenant.
- UI admin TESKEL (P1) menggunakan koneksi terpisah dengan audit ekstra.

### 22.5 Test

- Integration test wajib mencoba access lintas-org dan memastikan kosong.
- Suite RLS di `apps/api/test/rls/*.spec.ts`.

### 22.6 White-Label / Sub-Org (P2)

- Org dapat punya sub-org (parent_id) untuk agency multi-client.
- RBAC inheritance mengikuti hierarchy.

---

## 23. AI Infrastructure

### 23.1 Tujuan

- Provider-agnostic dengan default provider yang masuk akal.
- Cost transparan per request, per project, per org, per model.
- Eval otomatis sebelum prompt naik ke production.
- Guardrails (PII, jailbreak, moderation) wajib by default.
- Observability granular per request, per node workflow, per agent step.

### 23.2 Layered Architecture

```text
┌─────────────────────────────────────────────────────────┐
│  Application Code (apps/api, apps/worker, builder)      │
│  → @teskel/ai client                                     │
└────────────────────────┬────────────────────────────────┘
                         v
┌─────────────────────────────────────────────────────────┐
│  AI Service (packages/ai)                                │
│  - Router (provider/model selection)                     │
│  - Prompt Registry (versioned, evaluated)                │
│  - Vercel AI SDK v5 (`generateText`, `streamText`,       │
│    `ToolLoopAgent`, `streamObject`)                      │
│  - Usage Meter (cost calc, token count)                  │
│  - Guardrails (PII redact, jailbreak filter, moderation) │
│  - Cache (semantic + exact)                              │
│  - Tracing (OTel + Langfuse)                             │
└────────────────────────┬────────────────────────────────┘
                         v
┌─────────────────────────────────────────────────────────┐
│  Provider Adapters                                       │
│  - OpenRouter (default multi-model gateway)              │
│  - OpenAI / Anthropic / Google direct                    │
│  - Groq (low-latency Llama)                              │
│  - Self-host (Ollama / vLLM via OpenAI-compat)           │
│  - Image: Replicate / Fal / Stability                    │
│  - Audio: Deepgram / ElevenLabs                          │
└─────────────────────────────────────────────────────────┘
```

### 23.3 Model Router

- Strategi pemilihan: `task-tier` (`fast`, `balanced`, `reasoning`,
  `vision`, `embed`, `image`, `audio`) → daftar model dengan
  fallback chain + budget cap.
- Konfigurasi YAML di `packages/ai/config/router.yaml`:
  ```yaml
  routes:
    fast:
      models:
        - openrouter:google/gemini-2.5-flash
        - openrouter:anthropic/claude-haiku-4.5
        - openai:gpt-4o-mini
      max_cost_per_call_cents: 1
    reasoning:
      models:
        - openrouter:anthropic/claude-sonnet-4
        - openrouter:openai/gpt-5
      max_cost_per_call_cents: 50
    embed:
      models:
        - openai:text-embedding-3-small
      dim: 1536
  ```
- Fallback rules: timeout > 8s atau 5xx → next model.
- Model deprecation list di-track agar prompt yang masih pakai model tua
  ditandai.

### 23.4 Prompt Registry

- Tabel `prompt_templates(id, org_id, name, version, body, vars, model_hint,
  created_by, created_at)`.
- Variable interpolation `{{var}}` (Mustache-style, escape default).
- Setiap perubahan = versi baru (immutable). Active version per env.
- UI di Settings → AI → Prompts.

### 23.5 Eval Suite

- Format Promptfoo / custom JSON: `(input, expected, score_fn)`.
- Suites disimpan di repo `eval/` per template.
- Scorer: exact, regex, BLEU, embedding-similarity, LLM-judge (rubric).
- CI hook: `pnpm eval:ci` jalankan eval untuk prompts yang berubah; gagal
  bila skor regress > threshold.
- Tracking di Langfuse (run, dataset, score).

### 23.6 Guardrails

- **PII redaction** sebelum kirim ke provider (regex + LLM detector).
- **Jailbreak filter**: heuristics + small classifier (e.g., guard model).
- **Moderation**: OpenAI moderation API + provider-native (default deny on
  hate/CSAM/self-harm/violence policy violation).
- **Output filter**: schema validation (Zod) untuk structured output, length
  cap, banned strings.
- Lihat §52 untuk implementasi detail.

### 23.7 Caching

- **Exact-match** (key = hash prompt + model + params) di Redis dengan TTL
  konfigurable.
- **Semantic cache** (P1): embed prompt → cek nearest embedding (>0.95
  cosine) → reuse output. Disable per-call lewat header.

### 23.8 Tool Calling & Agents

- Pakai `ToolLoopAgent` dari Vercel AI SDK v5.
- Tools: `searchDocs`, `httpRequest`, `runWorkflow`, `queryDb`, `runCode`
  (E2B), `slackPost`, dst.
- Agent step di-trace per langkah; cost dihitung per step.
- Stop conditions: `stepCountIs(n)`, `tokenBudget(n)`, `humanApproval(...)`.

### 23.9 RAG

- KB per project. Document ingestion job: parse → chunk (semantic
  splitter) → embed → store ke `document_chunks`.
- Retrieval: hybrid (BM25 via `pg_trgm` + vector cosine) → rerank
  (Cohere/BGE) → top-K.
- Guardrails: max KB size per plan, total tokens per query.

### 23.10 Knowledge Base UX

- Sources: file upload, URL crawl (cheerio), Notion, Google Drive (P1).
- Re-index on update; status tracking per document.
- Embedding cost diteruskan ke org meter.

### 23.11 Cost Accounting

- Setiap call → row di `ai_requests` (lihat 21.3) + meter event ke Stripe
  (`ai_credits_used`).
- Markup per plan dikonfigurasi di `packages/billing/markup.ts`.
- Dashboard real-time `Project → AI` menunjukkan cost per model dan trace
  link Langfuse.

---

## 24. Workflow Runtime

### 24.1 Tujuan

- Engine eksekusi durable, observable, dan dapat di-monetize per-run.
- Mendukung event/cron/manual/webhook trigger.
- Mendukung node-tipe: trigger, transform, AI, agent, code, branch, loop,
  delay, http, db query, integration, output.

### 24.2 Arsitektur Eksekusi

`DECIDED` — split workload menjadi dua jalur:

1. **BullMQ-based runtime** (default) untuk workflow umum.
2. **Inngest-based durable runtime** (P1) untuk workflow yang butuh durable
   step semantics, retry kompleks, fan-out besar (cron + multi-day).

ADR-006 mendokumentasikan kapan pakai yang mana.

```text
[apps/api] -- enqueue --> [BullMQ queue: workflow-run] --> [worker]
                                 |
                                 v
                       [Workflow Engine]
                       - load graph version
                       - topological exec
                       - per-node retry/backoff
                       - context.persist(run_log)
                       - emit progress events
                                 |
            +--------------------+------------------+
            v                                       v
    [Postgres run/log]                  [Pub/Sub for live UI]
```

### 24.3 Graph Format

- JSON di-store di `workflow_versions.graph` (lihat Appendix D).
- Schema: `nodes[]`, `edges[]`, `triggers[]`, `vars{}`, `policies{}`.
- Versi schema (`graph_schema_version`) untuk migration.

### 24.4 Execution Semantics

- **Idempotent step** — node punya `step_key` deterministik. Re-run bisa
  resume dari step terakhir yang sukses.
- **Retry policy** per-node: `attempts`, `backoff` (`exponential`/`fixed`),
  `max_delay`, `jitter`.
- **Timeout** default 60s per-node, override per-node hingga 15 menit.
- **Branching**: `if`, `switch`, `all`, `race`.
- **Loop**: `for-each` dengan concurrency cap.
- **Sub-workflow**: panggil workflow lain sebagai sub-run.
- **Pause / wait**: `wait_event(name, ttl)`, `wait_seconds(n)`.

### 24.5 Live UI

- Worker publish event `run.progress` ke Redis pub/sub channel
  `org:{id}:run:{id}`.
- Web subscribe via SSE; React Flow canvas highlight node status real-time.

### 24.6 Triggers

- **Manual** (UI / API).
- **Webhook**: URL unik per workflow `/v1/webhooks/wf/:slug` dengan signing.
- **Schedule**: BullMQ Repeatable Jobs (cron).
- **Event**: outbox / domain events (e.g., `template.installed`).
- **AI trigger** (P1): natural language → workflow run.

### 24.7 Pricing per Run

- Cost = AI tokens cost + node compute cost + add-on integrations.
- Meter event `workflow_run` (count) + `workflow_run_cost` (sum).
- Plan limit hard cap pada `runs/month`.

### 24.8 AI Workflow Generation

- "Generate workflow from prompt" — gunakan structured output (Zod) ke
  graph schema.
- Validasi: setiap node config divalidasi terhadap node spec sebelum
  disimpan.

### 24.9 Versioning & Drafts

- Edit di canvas = draft.
- "Save version" → snapshot v1, v2, ... immutable.
- Trigger panggil versi `published`.
- Roll-back: published pointer di-set ke versi sebelumnya.

### 24.10 Audit & Replay

- Setiap run punya seed input + state snapshots tiap step → replay tools di
  Run Inspector.

---

## 25. Visual App Builder

### 25.1 Pilihan Engine

`DECIDED` — gunakan **Puck** sebagai default builder; pertahankan jalur
fallback ke **Craft.js** untuk kebutuhan custom layout (e.g., absolute
positioning, complex grid editor) bila ada template yang membutuhkan.

Rasionalisasi:

- Puck: opinionated, MIT, *agentic visual editor* (page generation API,
  field types kaya), fokus produk SaaS landing/dashboard. Cocok untuk
  templated app builder.
- Craft.js: lebih low-level, fleksibel, cocok untuk advanced canvas
  (drag-resize). Disimpan di belakang `BUILDER_ENGINE` flag.

ADR-007.

### 25.2 Block Library

- Atomic: heading, text, button, image, divider.
- Composite: hero, feature grid, pricing table, testimonial, CTA, FAQ,
  navbar, footer.
- Data-bound: list (from data source), form (with validation), AI block
  (chat, generate text), workflow trigger.
- Auth-aware: gated section (visible only if user has plan X).

### 25.3 Theme Editor

- Token override: font, radius scale, accent color.
- Preview real-time.
- Diff viewer antar version.

### 25.4 Persistence

- Builder state JSON di `app_versions(id, project_id, version, schema_version,
  blocks jsonb, tokens jsonb, ...)`.
- Publish → snapshot, generate static + dynamic islands.

### 25.5 Render Output

- Default: Next.js page yang membaca block config + render via block
  registry (`packages/ui/blocks`).
- Hindari client-side renderer berat; pre-render tiap block sebagai RSC
  bila memungkinkan.

### 25.6 Collaboration (P1)

- Multi-user editing via Liveblocks / Yjs (lihat 26).
- Lock per block saat editing oleh user lain.

### 25.7 AI Commands

- "Add testimonial section dari list ini" → AI patch JSON state.
- Patch divalidasi (Zod) → preview → user accept/reject.

### 25.8 Plugin Blocks

- 3rd-party block registered via plugin (lihat §33). Sandbox iframe untuk
  block plugin tidak ter-sign.

---

## 26. Real-Time & Collaboration

### 26.1 Layanan Real-Time Per-Surface

| Surface | Mekanisme | Library |
|---|---|---|
| AI streaming chat | SSE | Vercel AI SDK |
| Workflow run live UI | SSE (server) + pub/sub Redis | custom |
| Notification toast | WebSocket | `socket.io` (P1) |
| Builder collab cursor & presence | Liveblocks | `@liveblocks/react` |
| Workflow canvas collab | Yjs + provider (Liveblocks Yjs / Hocuspocus) | `yjs`, `y-react-flow` |
| Comments/threads | Liveblocks Comments | `@liveblocks/react-comments` |

### 26.2 Authentication for Real-Time

- Liveblocks: token-based auth dari `apps/api` (`/v1/realtime/token`).
- SSE: cookie session (web) atau API key (sdk).

### 26.3 Conflict Resolution

- Yjs CRDT untuk graph & blocks (auto-merge).
- Last-write-wins untuk single-field properties (with timestamp).

### 26.4 Limits

- Free plan: max 2 concurrent collaborators.
- Pro: 10. Team: 50. Enterprise: unlimited.

---

## 27. Storage & Files

### 27.1 Engine

- **Cloudflare R2** sebagai object store (S3-compatible, no egress fees).
- Bucket: `teskel-prod-{public,private,build,backup}`.
- Region: `auto` (R2 global).

### 27.2 Pola Akses

- Upload via signed URL (PUT) dari API; max size 100MB default,
  configurable per plan.
- Download via signed URL (GET) untuk private files; CDN untuk public.
- Thumbnail/preview generated by worker (sharp, ffmpeg).

### 27.3 File Categories

| Bucket | Konten |
|---|---|
| `public` | Marketplace cover, screenshot, public assets, OG images |
| `private` | User uploads, KB documents, exports |
| `build` | Build artifacts (Docker contexts, source bundles) |
| `backup` | DB backup snapshots, audit log archives |

### 27.4 Virus Scan

- ClamAV worker (P1) scan upload sebelum di-publish.
- Status `pending|clean|infected`. Infected → delete + notify.

### 27.5 Lifecycle

- Audit logs > 90 hari → cold archive (R2 standard-IA equivalent).
- Build artifacts > 30 hari → delete kecuali di-tag.

---

## 28. Queues, Jobs, Schedulers

### 28.1 Queue Family

| Queue | Purpose | Concurrency | Retry |
|---|---|---|---|
| `workflow-run` | Eksekusi workflow | tinggi (CPU/IO) | 3, exp 1m–10m |
| `ai-job` | Embedding, batch AI | medium | 5, exp 30s–5m |
| `deploy` | Build & deploy | low (per host) | 2, exp 30s–2m |
| `email` | Resend kirim | high | 5, exp 10s–5m |
| `webhook-out` | Outbound webhook | high | 8, exp 1m–2h |
| `analytics` | Event ingest backfill | high | 3, fixed 30s |
| `outbox` | Outbox dispatcher | high | infinite, capped |
| `cleanup` | TTL cleanups | low (cron) | n/a |

### 28.2 Scheduler

- BullMQ Repeatable Jobs untuk cron.
- Lock by job-id agar tidak duplicate.
- Cron schedule disimpan di `schedules` table (per workflow & per system).

### 28.3 Inngest (P1)

- Untuk durable AI workflow & complex multi-step automations dengan event
  fan-out.
- Sebagai library SDK, bukan replace BullMQ.

### 28.4 Worker Topologi

- 1+ `worker` process (`apps/worker`) per environment, scale horizontal
  by queue.
- Health endpoint `/healthz` (Liveness) & `/readyz` (Readiness).
- Graceful shutdown: drain in-flight jobs sebelum exit.

---

## 29. Caching Layers

| Layer | Tool | Use |
|---|---|---|
| HTTP | Cloudflare CDN | Marketing, static assets, public marketplace |
| Edge | Next.js `revalidate` + `Cache-Tag` | List endpoints (per org) |
| App | Redis (`@unkey/cache` style) | Hot org metadata, plan limits |
| AI | Redis (exact + semantic P1) | LLM call cache |
| DB | PG `pg_stat_statements` aware queries; no in-app | — |

Cache invalidation rules:

- Tagged invalidation per `org:{id}:projects`, `org:{id}:billing`, dst.
- Mutasi yang sukses → invalidate via `cacheTag(...)`.

---

## 30. Search

### 30.1 MVP

- Postgres FTS (`tsvector`) untuk listing, template, workflow.
- pgvector untuk semantic.
- Hybrid: BM25 + cosine + reranker.

### 30.2 Skala (P2)

- Migrasi ke **Meilisearch** atau **Typesense** bila volume listing > 50k
  atau latency p95 query > 200ms.
- Indexer worker yang subscribe outbox events.

---

## 31. Notifications System

### 31.1 Channels

- Email (Resend, default).
- In-app toast / inbox (Postgres `notifications` table).
- Slack (incoming webhook + OAuth).
- Discord (webhook).
- Push (P2).

### 31.2 Preferences

- Per user: per-event channel toggle.
- Org-level digest setting (instant / hourly / daily).
- Quiet hours.

### 31.3 Templating

- `notifications/templates/*.tsx` (React Email).
- Localized via `next-intl`.

### 31.4 Delivery

- Worker `email` queue.
- Track open/click via Resend webhooks → write back ke
  `notifications.engagement`.

---

## 32. Sandbox Execution Layer

### 32.1 Use Cases

- "Code" node di workflow.
- AI tool execution (untrusted Python/JS).
- Template install scripts.

### 32.2 Choice

`DECIDED` — **E2B** sebagai default sandbox (cloud, ephemeral).
**Daytona** sebagai opsi P1 untuk persistent dev sandbox + fastest
cold-start (~90ms). **Modal** sebagai opsi P1 untuk GPU workload (image
generation, fine-tuning). ADR-008.

### 32.3 Architecture

```text
[worker] -- task --> [Sandbox Provider]
                          |
                  ephemeral container
                  (CPU/Mem/Net policy)
                  network egress allow-list
                          |
                  result (stdout, files) -> R2
```

### 32.4 Security

- Default deny outbound network kecuali whitelist domain.
- No host filesystem access.
- 60s timeout default; 5 min max.
- Rate-limit per org.

### 32.5 Fallback

- Self-host sandbox (Firecracker / Kata) `LATER` jika cost E2B menjadi
  pain point.

---

## 33. SDK & Plugin Architecture

### 33.1 SDK

- `@teskel/sdk` (Node + browser ESM) — wrap Hono RPC client + auth helper.
- `@teskel/cli` — gunakan SDK.
- `@teskel/react` (P1) — hooks (`useTeskelClient`, `useWorkflowRun`).
- `@teskel/server` (P1) — server-side helper untuk template.

Versioning via Changesets, semver strict.

### 33.2 Plugin Surfaces

| Surface | Plugin Type | Trust |
|---|---|---|
| Workflow node | `@teskel/plugin-node` package | Verified (manual review) |
| Builder block | `@teskel/plugin-block` | Sandboxed iframe |
| Provider (AI/Model) | `@teskel/plugin-ai` | Verified |
| Integration (Slack/Notion) | `@teskel/plugin-integration` | Verified |

### 33.3 Plugin Manifest

- `package.json#teskel`:
  ```json
  {
    "type": "node",
    "id": "com.acme.scrape-url",
    "version": "1.0.0",
    "permissions": ["http.fetch", "storage.read"],
    "ui": { "iconUrl": "...", "fields": "schema.json" },
    "entry": "dist/index.js"
  }
  ```
- Signed with creator key; verified pada install.

### 33.4 Distribution

- Verified plugin di-host npm + mirror R2.
- Marketplace plugin section (P1).

---

## 34. CLI Tool (`teskel`)

### 34.1 Tujuan

- Membantu developer / creator melakukan: login, scaffold project, dev
  preview, push template, run workflow lokal, deploy.

### 34.2 Stack

- `clipanion` + `commander` hybrid; `prompts` untuk interaktif.
- Distribusi: npm `@teskel/cli`, Homebrew tap `teskel/cli`.

### 34.3 Commands (MVP)

```text
teskel login
teskel logout
teskel whoami
teskel init [template]            # scaffold project lokal
teskel dev                        # local dev (proxy ke teskel cloud)
teskel template push              # publish template version
teskel template install <slug>
teskel workflow run <id>          # trigger run dari CLI
teskel deploy [--env]             # trigger deploy
teskel logs [resource]            # tail logs
teskel secrets {get,set,rm,ls}
teskel env {list,switch}
```

### 34.4 Auth

- OAuth device flow + token cache di `~/.teskel/credentials`.
- Ephemeral session untuk CI (`TESKEL_API_KEY` env).

### 34.5 DX

- Output: pretty (default) atau JSON (`--json`) untuk piping.
- Progress bars (Listr2).
- `teskel doctor` — diagnose env, credentials, version.

---

# Part IV — Operations

## 35. Environments

| Env | Domain | Purpose | Data |
|---|---|---|---|
| `local` | `localhost` | Dev | seed scripts |
| `preview` | `pr-<id>.preview.teskel.dev` | per-PR ephemeral | sanitized snapshot |
| `staging` | `staging.teskel.dev` | release rehearsal | sanitized prod snapshot |
| `prod` | `teskel.app` & `*.teskel.app` | end-users | live |

- Promotion path: `local → preview → staging → prod`. No skipping.
- Each env has separate Stripe account / mode (test vs live), separate
  Postgres/Redis instances, separate R2 buckets, separate KMS keys.
- Naming convention env vars: `TESKEL_ENV` ∈ `local|preview|staging|prod`.

## 36. Deployment Infrastructure

### 36.1 MVP — Coolify

- 1× control VM (4 vCPU / 8 GB) menjalankan Coolify + Traefik.
- 2–4× worker VM untuk customer apps + TESKEL workers.
- Postgres managed (Neon), Redis managed (Upstash atau self-host VM kecil).
- DNS Cloudflare; wildcard cert `*.teskel.app` (Let's Encrypt via Traefik).
- Image registry: GHCR.

### 36.2 Customer App Deploy Pipeline

1. User klik **Deploy**.
2. API enqueue `deploy` job dengan `project_id`, `version`.
3. Worker:
   1. Fetch source bundle (template + user customizations).
   2. Render `Dockerfile` (template's or generic Next.js + Hono mono).
   3. Build image (`buildx`) → push GHCR with tag `prj-<id>:<version>`.
   4. Call Coolify API to create/update service + domain.
   5. Wait health check (`/healthz` 200 within 60s).
   6. Mark `deployments.status = 'succeeded'`.
4. Failure → roll back to last green; emit `deployment.failed` event.

### 36.3 Roadmap

- Phase 3+: migrate growing tier to **Fly.io** atau **Railway** (multi-region,
  managed Postgres-by-region) untuk customer apps yang butuh global edge.
- Phase 5+: consider **Kubernetes** (EKS/GKE) jika daily deploy > 5k atau
  cust app TPS > 1k.

### 36.4 Domain Management

- Subdomain `*.teskel.app` (free).
- Custom domain: user POIN CNAME → `cname.teskel.app`. Auto-provision Let's
  Encrypt cert. Verify ownership via TXT record.

### 36.5 Rollback

- Tombol "Rollback" → re-tag previous image as current; deploy job sama
  flow.
- DB migration tidak pernah destruktif (lihat 21.6 expand-contract).

## 37. CI/CD Pipelines

### 37.1 Triggers

- `push` ke branch → CI lint + typecheck + test (parallel matrix).
- `pull_request` → above + preview deploy + e2e smoke.
- `merge to main` → staging deploy.
- `tag v*` → prod deploy (manual approval gate).

### 37.2 GitHub Actions Workflows

| Workflow | Trigger | Steps |
|---|---|---|
| `ci.yml` | PR | install, lint, typecheck, unit, int (db service) |
| `e2e.yml` | PR (label `e2e`) | Playwright matrix (chromium, mobile) |
| `eval.yml` | PR yang ubah prompts | run prompt eval suite |
| `oasdiff.yml` | PR yang ubah API | OpenAPI diff check |
| `preview.yml` | PR | deploy preview (Coolify), comment URL |
| `release.yml` | Tag | build images, push GHCR, deploy prod, run smoke |
| `nightly.yml` | Schedule | full e2e + load smoke |

### 37.3 Required Status Checks

- lint, typecheck, unit, int, e2e (smoke), oasdiff, eval (jika applicable).

### 37.4 Deploy Approval

- Prod deploy butuh 1 reviewer dari `@teskel/release-managers`.
- Auto-rollback bila smoke test post-deploy fail.

### 37.5 Secrets

- GitHub Actions secrets di-scope per env via Environments.
- Rotate quarterly (lihat §49).

## 38. Configuration & Secrets

### 38.1 Config Loading

- Library: `@t3-oss/env-nextjs` + `zod` untuk web.
- API: custom `lib/env.ts` (Zod schema, validate at boot).
- 12-factor: env vars saja; tidak ada file config tergantung path.

### 38.2 Secret Categories

| Kategori | Storage |
|---|---|
| Platform infra (DB url, Redis, OAuth keys) | Coolify env / GH Actions secrets |
| Per-org / per-project secrets | Encrypted di `secrets` table (KMS-backed) |
| API keys user | Hashed (`argon2id`), prefix only stored plain |

### 38.3 KMS

- AWS KMS atau Cloudflare encryption (DEK per org, KEK rotated yearly).
- Key versioning: `key_version` di tabel; dukung re-encrypt tanpa downtime.
- Audit semua decrypt.

### 38.4 Secret Rotation

- Quarterly rotate platform secrets (DB password, OpenRouter key).
- Per-user API key: user-driven; auto-revoke after 365 hari unused.

### 38.5 Local Dev Secrets

- `.env.local.example` checked-in dengan placeholder.
- `direnv` direkomendasikan; CI guard untuk file `.env*` accidentally
  committed.

## 39. Observability

### 39.1 Pillars

| Pillar | Tool | Notes |
|---|---|---|
| Logs | Loki + Grafana | Structured JSON via pino; redact PII |
| Metrics | Prometheus → Grafana | RED, USE; per-route p50/p95/p99 |
| Traces | OpenTelemetry → Tempo | Distributed tracing across web→api→worker |
| Errors | Sentry | Source map upload from CI; release tagging |
| AI Obs | **Langfuse** | LLM trace, eval, cost; self-hosted |
| Product Analytics | PostHog | Funnel, retention, replay |
| Uptime | Better Stack atau Statuspage | External probes |

### 39.2 Conventions

- Trace ID propagated via `traceparent` header.
- Each log line includes `request_id`, `org_id`, `user_id`, `module`,
  `route` jika applicable.
- Sentry release = git sha; deploy hook auto-tag.

### 39.3 Dashboards (Grafana, di-IaC kan)

- API: req rate, latency p50/p95/p99, 5xx %, top routes.
- Worker: queue depth, job latency, retry rate per queue.
- Postgres: connections, slow queries, replication lag.
- Redis: memory, ops/sec, evictions.
- Stripe: webhook lag, failed events, MRR.
- AI: tokens/sec, cost/min per model, error rate per provider.

### 39.4 Alerts (Pager rules)

| Alert | Threshold | Severity |
|---|---|---|
| API 5xx rate | >1% over 5m | Sev1 |
| API p95 latency | >1s over 10m | Sev2 |
| Queue lag (workflow) | >5m | Sev2 |
| Stripe webhook backlog | >100 pending | Sev2 |
| AI provider error rate | >5% per provider 10m | Sev2 |
| Postgres replica lag | >30s | Sev2 |
| Disk usage | >85% | Sev2 |

### 39.5 Logging Pipeline

- pino → stdout → Vector → Loki.
- Sampling untuk endpoint sangat ramai (1% sample) di luar error.

## 40. SLI / SLO / SLA

### 40.1 SLI

- API availability: % 2xx/3xx of all requests (excl. 4xx user errors).
- API latency: p95 ms.
- Worker job success: % success / total non-canceled.
- Deployment success: % successful deploys / triggered.
- AI router success: % successful provider call (post fallback).

### 40.2 SLO Targets (MVP)

| Service | Availability | Latency | Window |
|---|---|---|---|
| API | 99.9% | p95 < 400ms | 30d |
| Worker | 99.5% | p95 job-start < 5s | 30d |
| Deploy | 99% success | p95 < 5 min total | 30d |
| AI router | 99% | p95 < 8s | 30d |

### 40.3 SLA (External, Phase 5+)

- Enterprise: 99.95% with 25% credit if breached.
- Pro/Team: 99.9% with credit policy.

### 40.4 Error Budget

- Hitung budget = `(1 - SLO) * window`.
- Stop merge non-critical jika budget < 10% dalam 30 hari.

## 41. Incident Response & Runbooks

### 41.1 On-Call

- Primary + secondary (PagerDuty atau betterstack).
- Rotation weekly; handoff Friday 17:00 local.

### 41.2 Severity

| Sev | Definisi | SLA Response |
|---|---|---|
| Sev1 | Customer-impacting outage / data loss | 5 menit |
| Sev2 | Feature degraded, workaround tersedia | 30 menit |
| Sev3 | Bug minor, non-blocking | next business day |

### 41.3 Incident Lifecycle

1. Trigger (alert / ticket / customer).
2. Acknowledge (≤ SLA).
3. Mitigate (rollback / disable feature flag / scale).
4. Communicate (statuspage update tiap 30 menit untuk Sev1).
5. Resolve.
6. Postmortem (Sev1/2 wajib dalam 5 hari kerja, blameless).

### 41.4 Runbooks (di repo `apps/api/docs/runbooks/`)

- `db-down.md`, `redis-down.md`, `stripe-webhook-backlog.md`,
  `ai-provider-outage.md`, `deploy-stuck.md`, `disk-full.md`,
  `replica-lag.md`, `key-leak.md`, `data-leak-rls-test.md`,
  `creator-payout-failure.md`.
- Setiap runbook: gejala → diagnosa → tindakan → verifikasi → rollback →
  log entry.

## 42. Backup, DR, Retention

### 42.1 Backup

- Postgres: managed snapshot daily (Neon PITR 7d MVP, 30d Phase 3+).
- R2: versioned bucket; cold archive monthly.
- Redis: tidak di-backup (cache/queue stateless-friendly), tetapi BullMQ
  job persistence di Redis disnapshot ke Postgres `outbox` mirror untuk
  job kritis (deploy, payout).

### 42.2 RPO / RTO

| Service | RPO | RTO |
|---|---|---|
| Postgres | 5 menit (PITR) | 60 menit |
| R2 | 1 jam | 30 menit |
| App tier | n/a | 15 menit (re-deploy) |

### 42.3 Disaster Recovery

- Game day quarterly (simulate primary DB lose; restore from snapshot).
- Runbook `dr-restore.md` tested.

### 42.4 Retention

| Data | Retention | Archive |
|---|---|---|
| Audit logs | 365 hari (SOC2 minimum) | R2 cold |
| Workflow run logs | 30 hari | R2 (paid tier 90d) |
| AI request logs | 30 hari | R2 (paid tier 90d) |
| Email events | 90 hari | — |
| Backups | 30 hari hot + 1 yr cold | R2 |
| Deleted user data | hard delete 30 hari (GDPR), audit row anonymized | — |

## 43. Performance Budgets & Load Testing

### 43.1 Budgets

- API: p95 < 400ms (read), < 800ms (write), < 8s (AI streaming first token).
- Web: LCP < 2.0s p75 marketing; TTI < 3s p75 dashboard.
- Workflow run startup: < 5s p95.

### 43.2 Load Testing

- k6 scripts di `tools/load/` untuk:
  - signup + create project + first AI call,
  - workflow run burst (1k concurrent),
  - marketplace listing browse,
  - deploy throughput (10 concurrent deploys).
- Run pre-launch + monthly.

### 43.3 Profiling

- Node `--prof`, Clinic.js untuk hot path.
- Postgres `auto_explain` + `pg_stat_statements`.
- Slow query alert > 200ms (top queries reviewed weekly).

## 44. FinOps & Cost Management

### 44.1 Cost Categories

| Bucket | Tracking | Budget Alert |
|---|---|---|
| AI providers | per-request `ai_requests` | per-org & global daily caps |
| Compute (VM) | provider bill | weekly budget review |
| DB | Neon / managed | 80% storage alert |
| R2 | bill | 80% bandwidth alert (jika ada egress) |
| Email (Resend) | meter | 80% volume alert |
| Sentry/Langfuse/PostHog | meter | quarterly tier review |

### 44.2 Per-Org Cost Attribution

- Tabel `org_cost_attribution(org_id, date, ai_cost_cents,
  infra_cost_cents, storage_cost_cents, ...)`.
- Daily job hitung dari `ai_requests`, `deployments`, `usage_logs`,
  estimasi infra (VM allocation per app).
- Bandingkan dengan `subscriptions` + `meter_events` revenue → per-org
  margin.

### 44.3 AI Cost Controls

- Hard cap AI cost per org per day (soft warn 70%, hard at 100%, override
  by upgrade plan).
- Provider preference tier: cheapest yang masih lulus eval.
- Aggressive caching untuk prompt sama.
- Truncate context window: `tiktoken` aware budget per call.

### 44.4 Reserved Capacity (P2)

- Untuk OpenAI/Anthropic prepay/credit deals jika volume justifies.

### 44.5 Vendor Reviews

- Quarterly: bandingkan harga OpenRouter vs direct providers, R2 vs S3,
  Coolify VM vs Fly. Migrasi jika spread > 25% dengan ROI <= 1 kuartal.

---

# Part V — Security & Compliance

## 45. Threat Model & STRIDE

### 45.1 Aktor

- Anonymous attacker (web).
- Authenticated user (multi-tenant exfil attempt).
- Malicious creator (template/plugin code).
- Compromised admin / employee.
- Supply chain (npm package compromise).

### 45.2 Surface Area

- Web auth, public API, webhooks (in/out), workflow runtime, sandbox,
  marketplace upload, deploy pipeline, billing, KB upload.

### 45.3 STRIDE Checklist (per surface)

Setiap modul wajib mengisi tabel STRIDE:

| Threat | Spoofing | Tampering | Repudiation | Information Disclosure | DoS | Elevation |
|---|---|---|---|---|---|---|
| Mitigation | OAuth + MFA | HMAC + idempotency | Audit log | RLS + KMS | Rate limit + circuit breaker | RBAC + scope |

Stored di `apps/api/docs/security/stride-<module>.md`.

## 46. AuthN/AuthZ

### 46.1 Pilihan

`DECIDED` — **Better Auth** (open-source, TypeScript, plugins:
organizations, RBAC, magic link, passkeys, 2FA TOTP, session management,
email verification, password reset).

Rasionalisasi:

- Tidak ada per-MAU cost (vs Clerk).
- Native Hono + Next.js adapter.
- Plugins: Organization (multi-tenant), Admin (impersonate), Passkeys
  (WebAuthn), Two-Factor (TOTP/SMS), API Keys, OIDC (P1), SAML (P2).
- Self-host & ownership penuh atas data session.

### 46.2 Sign-in Methods (MVP)

- Email + magic link (default).
- OAuth: Google, GitHub.
- Passkeys (WebAuthn) sebagai *promote* setelah pertama login.
- 2FA TOTP opsional (Pro+).
- SSO SAML (P2, Enterprise).

### 46.3 Session

- Cookie `__teskel_sess` `Secure`, `HttpOnly`, `SameSite=Lax`.
- Sliding 30 hari dengan rotation.
- Device list + revoke per device.

### 46.4 Authorization

- RBAC at org level (lihat 47).
- Scope-based for API keys (lihat 48).
- Resource-level checks selalu di service layer:
  ```ts
  await requireOrgRole(ctx, 'admin');
  const project = await projectRepo.findByIdInOrg(orgId, projectId);
  ```

### 46.5 Anti-Abuse

- Rate-limit sign-in per email + per IP (sliding window).
- Captcha (Turnstile) setelah 5 failed attempts.
- Suspicious login email (new IP/UA) dengan device verification.

## 47. RBAC Matrix

### 47.1 Roles

| Role | Scope | Notes |
|---|---|---|
| **Owner** | Org | full, billing, delete org |
| **Admin** | Org | members, secrets, settings, no delete org |
| **Member** | Org | projects CRUD, workflows, deploy |
| **Viewer** | Org | read-only |
| **Billing** | Org | billing tab only |
| **Auditor** | Org | audit log + read |
| **Guest** | Project | invited to a single project |
| **Service** | Org | API key actor, scope-limited |

### 47.2 Permission Matrix (excerpt)

| Resource | Action | Owner | Admin | Member | Viewer | Billing | Auditor |
|---|---|---|---|---|---|---|---|
| Project | create | ✓ | ✓ | ✓ | – | – | – |
| Project | delete | ✓ | ✓ | – | – | – | – |
| Workflow | run | ✓ | ✓ | ✓ | – | – | – |
| Deploy | trigger | ✓ | ✓ | ✓ | – | – | – |
| Members | invite | ✓ | ✓ | – | – | – | – |
| Billing | edit | ✓ | – | – | – | ✓ | – |
| Audit log | view | ✓ | ✓ | – | – | – | ✓ |

Matrix lengkap di `apps/api/docs/security/rbac.md`.

### 47.3 Implementation

- `packages/auth/permissions.ts` — Zod schema action enum.
- `requirePermission(ctx, 'project.delete', { orgId, projectId? })`.
- Middleware audit log di setiap denied permission.

## 48. API Keys & Token Management

### 48.1 Format

- `tk_live_<random>` dan `tk_test_<random>` (env-aware).
- Stored as `argon2id` hash; prefix (first 8 chars) saved plain untuk UI
  display.
- Scope: comma-separated (`project.read,workflow.run,deploy.trigger`).

### 48.2 UX

- Create → display once → user wajib copy.
- Last-used time, IP, UA tracked.
- Rotate / revoke.
- IP allowlist optional (Pro+).

### 48.3 Lifetime

- No expiry default.
- Auto-revoke jika tidak dipakai 365 hari.
- Mandatory rotate setiap 365 hari (hint, not block) untuk security
  hygiene.

### 48.4 OAuth Apps (P1)

- 3rd-party `client_id` + `client_secret` flow standard.
- Scopes terbatas; admin approve aplikasi org-wide.

## 49. Secrets Vault & Encryption

### 49.1 Encryption At Rest

- Postgres: storage-level encryption (managed DB default).
- Sensitive columns (e.g., `secrets.ciphertext`,
  `creator_profiles.kyc_payload`): app-level AES-256-GCM dengan KMS DEK.

### 49.2 Encryption In Transit

- TLS 1.3 di semua hop.
- Mutual TLS (mTLS) antar service di prod (P1).

### 49.3 Field-Level Encryption

- Helper `encrypt(field, kekVersion)` dan `decrypt(...)`.
- Index pada hash (`SHA-256`) jika perlu searchable encryption (lookup),
  tetap non-reversible.

### 49.4 Key Management

- AWS KMS / Cloudflare KMS.
- KEK rotated yearly; DEK rotated per-org on plan change atau on-demand.
- Re-encrypt job background (idempotent).

## 50. Webhook Security

### 50.1 Inbound

- Signature header (`x-teskel-signature`) HMAC-SHA256 dari raw body +
  timestamp + secret.
- Replay window 5 menit.
- Dedupe `event_id`.
- Per-source IP allowlist optional.

### 50.2 Outbound

- TESKEL emit dengan signing key per `webhook` row.
- Retry policy 8 kali eksponensial sampai 24 jam.
- Customer dapat verify dengan helper di SDK.

### 50.3 Stripe

- Pakai Stripe SDK verify, simpan `event_id` di `meter_events.stripe_event_id`
  untuk dedupe.
- Process di queue (idempotent handler), tidak inline di HTTP handler.

## 51. Workflow & Code Sandbox Security

### 51.1 Threats

- Arbitrary code via "code" node mencoba SSRF, exfil, fork bomb.
- Workflow yang panggil endpoint internal (metadata service, RDS).

### 51.2 Mitigations

- Sandbox via E2B (default deny network egress; allowlist domain).
- Outbound HTTP node: blocklist private CIDR (`10/8`, `172.16/12`,
  `192.168/16`, `169.254.169.254`).
- Per-workflow time/memory cap.
- Per-org rate limit.
- "Code" node disabled di Free plan.
- Audit log on each code run.

### 51.3 Plugin Security

- Plugins di-review (kode + manifest) sebelum verified.
- Permissions explicit di manifest; runtime enforce.
- Unsigned plugins jalan di sandbox + warning UI.

## 52. AI Safety & Guardrails

### 52.1 Input

- PII regex (email, phone, credit card) → optional redaction (org-config).
- Jailbreak detector (small classifier; flag > 0.8 → block + log).
- Max prompt length per plan.

### 52.2 Output

- Moderation (OpenAI moderation atau guard model self-host).
- Output filter: schema validation jika structured output expected.
- Token cap; auto-truncate dengan summary tail.

### 52.3 Tool Use

- Tool execution requires user approval pada agent run sensitif (delete
  records, send funds).
- "Human-in-the-loop" mode default untuk Enterprise tier.

### 52.4 Logging

- Prompt + completion logged ke Langfuse (encrypted, retention 30 hari
  default; configurable per org).
- Privacy mode: org dapat opt-out logging completion (only metadata).

### 52.5 Red Team

- Quarterly red-team run melawan template default + agent default.
- Report hasil + remediasi.

## 53. Compliance Roadmap

### 53.1 Phased

| Phase | Compliance | Tooling |
|---|---|---|
| Phase 1 (MVP) | Privacy policy, ToS, DPA template, sub-processors list, GDPR baseline | Termly / iubenda |
| Phase 2 | SOC2 Type 1 readiness | Vanta / Drata |
| Phase 3 | SOC2 Type 1 attestation | auditor (e.g., Sensiba) |
| Phase 4 | SOC2 Type 2 (3-12 month observation) | Vanta + auditor |
| Phase 5 | ISO 27001 (kalau market enterprise demand) | TBD |
| Phase 5 | HIPAA (jika healthcare verticals) | optional |
| Phase 6 | EU AI Act readiness | legal review |

### 53.2 Controls Highlights (untuk SOC2 Type 1)

- Access review quarterly.
- Background checks for engineers with prod access.
- Security awareness training (yearly).
- Code review required (≥1 reviewer; 2 untuk auth/billing).
- Secrets in vault; no plaintext in repo (pre-commit guard).
- Backup tested quarterly.
- Vendor risk review docs.

### 53.3 DPA & Sub-processors

- DPA template tersedia di `legal/dpa.md`.
- Sub-processors list di `legal/subprocessors.md`, update ≥30 hari sebelum
  perubahan, notify orgs Enterprise/Team.

### 53.4 Audit Log Surface

- Min 365 hari, exportable JSON & CSV.
- Wajib events: auth, role change, secret CRUD, billing change, deploy,
  data export.

## 54. Privacy & Data Residency

### 54.1 Default

- Region utama EU (Frankfurt / NL) atau US East — pilih sesuai market
  awal. Plan v2 default `EU-West` (Frankfurt) untuk neutrality.
- Org dapat memilih residency `EU` atau `US` (P2) saat create org.

### 54.2 PII Handling

- Minimal collection; explicit fields untuk profile.
- DSR (data subject request): export & delete UI; SLA 30 hari.
- Cookie banner (EU) via consent management.

### 54.3 Children

- Tidak ditujukan untuk anak < 16; ToS forbid.

## 55. Vulnerability Management & SDLC

### 55.1 Dependencies

- Renovate + npm audit di CI.
- Pin major; auto-bump minor weekly; manual review for major.
- SBOM (CycloneDX) generated in release.

### 55.2 Static Analysis

- ESLint plugin-security, Semgrep custom rules untuk:
  - SQL string concat,
  - missing tenant scope,
  - missing idempotency,
  - secret strings.
- TruffleHog di CI scan secrets in commits.

### 55.3 DAST

- ZAP baseline scan nightly terhadap staging.

### 55.4 Bounty / Responsible Disclosure

- `security.txt` + `SECURITY.md`.
- HackerOne / private bounty (P3) setelah Phase 5.

### 55.5 Pentest

- Annual external pentest (Phase 4+).

---

# Part VI — Growth & Ecosystem

## 56. Marketplace Architecture

### 56.1 Entitas

```text
templates ── (1:N) ── template_versions ── (1:N) ── template_installs
templates ── (1:1) ── marketplace_listings (jika di-publish)
marketplace_listings ── (1:N) ── reviews
marketplace_listings ── (1:N) ── licenses ── (N:1) ── orgs
creator_profiles ── (1:1) ── users
creator_profiles ── (1:N) ── creator_payouts
```

### 56.2 Lifecycle Listing

```text
draft → submitted → in_review → (approved | rejected) → live → updated → ...
                                                                ↓
                                                            archived
```

### 56.3 Pricing Models

- Free.
- One-time purchase (with revenue share, payout via Stripe Connect).
- Subscription (per-listing recurring).
- Usage-based (per-install dengan pay-as-you-go) — `LATER`.

### 56.4 Versioning

- Semver (`major.minor.patch`).
- Buyer dapat lock ke major version; auto-update minor/patch opsional.

### 56.5 Search & Discovery

- Categories, tags, collections (curated), trending, new, top.
- Search hybrid (BM25 + embedding).
- Sort: relevance, rating, recent, price.

### 56.6 Trust Signals

- Verified Creator badge.
- Number installs, rating, reviews, last update.
- "Built with TESKEL" stamp pada template fork preview.

## 57. Creator Economy Mechanics

### 57.1 Onboarding Creator

1. Apply Creator Studio.
2. KYC via Stripe Connect (Standard).
3. Submit creator profile (handle, bio, links).
4. Verifikasi email + identitas.
5. Approved → bisa publish.

### 57.2 Revenue Share

- 80% creator default; 85% Verified Creator.
- Platform fee: 20% / 15% (termasuk Stripe fee + ops).
- Refund: full refund 7 hari, debit dari wallet creator (negatif balance
  blocked dari payout berikutnya).

### 57.3 Payout

- Monthly, minimum $25.
- Stripe Connect Standard payout (creator melihat di dashboard Stripe).
- Tax docs handled via Stripe (1099 US, others applicable).

### 57.4 Fraud Prevention

- Transaction velocity limit per creator.
- IP heuristics + device fingerprint untuk self-purchase.
- Manual review untuk anomaly.

### 57.5 Creator Tools

- Analytics: views, install conv, revenue, refunds, ratings.
- Promo code (P1).
- Coupon stacking rules.

## 58. Template Manifest Spec

### 58.1 File: `template.yaml`

Lihat **Appendix C** untuk full sample.

### 58.2 Schema Sketch

```yaml
schema_version: 1
id: ai-support-bot-saas
slug: ai-support-bot-saas
name: AI Support Bot SaaS
version: 1.2.0
type: saas | workflow | agent | landing
license: proprietary | mit | custom
author:
  handle: dimas
  url: https://teskel.app/u/dimas
description: "..."
preview_urls: [demo.teskel.app/...]

required:
  env:
    - OPENAI_API_KEY
    - STRIPE_SECRET_KEY
  integrations:
    - stripe
    - openai
  plan_min: starter

resources:
  app:
    framework: nextjs15
    pages_dir: app/
  workflows:
    - file: workflows/triage.json
    - file: workflows/respond.json
  agents:
    - file: agents/support.yaml
  knowledge_bases:
    - name: docs
      sources: [docs/**/*.md]
  prompts:
    - file: prompts/system.md
  database:
    migrations_dir: db/migrations/
  branding:
    accent_token: violet
    radius_token: xl

install:
  steps:
    - run: pnpm install
    - run: pnpm db:migrate
    - run: pnpm seed
  post_install_url: /onboarding

permissions:
  - http.fetch
  - storage.read
  - workflow.run

signing:
  signature: <hex>
  pubkey: <hex>
```

### 58.3 Validation

- JSON Schema (generated from Zod) di `packages/template-engine/schema/`.
- CLI `teskel template validate` cek manifest + signing.

### 58.4 Signing

- Ed25519 sign payload manifest + content hashes.
- Public key creator stored di `creator_profiles.signing_pubkey`.
- Verifier di server saat install.

## 59. Marketplace Review & Moderation

### 59.1 Pipeline

```text
submit → automated checks → human review queue → decision
```

### 59.2 Automated Checks

- Manifest validation (schema, required fields).
- Static scan: `npm audit`, Semgrep rules custom.
- Dependency allowlist + license scan.
- Plagiarism: hash compare terhadap repo public ternama.
- Prompt sensitivity scan (jailbreak templates ditolak).
- Sandbox install on test org → smoke run workflow + AI calls.

### 59.3 Human Review Rubric

- UX quality (screenshots, demo).
- Description accuracy.
- Pricing fairness (no $0.01 abuse).
- IP / trademark check.
- Policy compliance (no scraping, no malware).

### 59.4 SLA

- First response 3 business days.
- Decision dalam 7 business days.

### 59.5 Trust & Safety Actions

- Soft warn → hide from listing → unpublish → ban creator (eskalasi).
- Audit trail di `moderation_actions` table.

### 59.6 Reviews & Reports

- 1-5 stars + body text.
- Flag for: spam, scam, harassment, broken.
- 24-jam SLA tindak flag P0 (scam).

## 60. Pricing & Billing Implementation

### 60.1 Stripe Integration

- Stripe Billing untuk subscription + meter.
- Stripe Connect Standard untuk marketplace payout.
- Webhook events di-process di queue (idempotent).

### 60.2 Meter Events

- Endpoint internal `/v1/internal/meter` mengirim ke Stripe Meter Events.
- Pre-aggregated buffer 30 detik di Redis untuk reduce API call.
- Idempotency-key = `org:metric:bucket-time`.
- Topic per metric: `ai_credits_used`, `workflow_runs`, `deploy_count`.

### 60.3 Plan Change

- Upgrade: prorate immediately.
- Downgrade: end-of-cycle.
- Cancel: end-of-cycle, retention email + offer.

### 60.4 Invoice & Tax

- Stripe Tax (P1) untuk auto sales tax / VAT.
- Manual tax mapping table (P0) untuk EU MOSS, ID PPN, US states (lazy).

### 60.5 Failed Payment

- Smart retries (Stripe).
- Dunning email day 1, 3, 7, 14.
- Grace period 14 hari, lalu `past_due` → feature freeze, lalu `unpaid` →
  read-only.

### 60.6 Coupons & Trials

- 14-day trial Pro pada signup (toggle).
- Coupon code (Stripe coupon) di-apply via UI / link.

## 61. Onboarding & Activation

### 61.1 Goal

- Aha moment within 10 menit; activation = first deploy + first AI call.

### 61.2 Flow

- Welcome screen with role pick (founder / builder / creator).
- Per role: tampilkan template stack relevan.
- Inline checklist yang mengarah ke deploy + AI call.
- Tooltips kontekstual.
- Email cadence (Resend): D0 welcome, D1 nudge, D3 case study, D7 paid
  upsell, D14 win-back.

### 61.3 Personalization

- Pertanyaan singkat (apa yang ingin dibangun) → suggest template.
- AI assistant: "Saya bantu setup, ceritakan bisnismu" → propose stack.

### 61.4 Activation Metrics

- Track per cohort di PostHog.
- A/B test variants of onboarding (lihat 64).

## 62. SEO & Content Strategy

### 62.1 Pages

- Marketing (`/`), Pricing, Templates, Use Cases, Customers, Changelog,
  Blog, Docs.
- Setiap template punya landing page (`/templates/[slug]`).
- Setiap creator punya profile (`/u/[handle]`).

### 62.2 Tech SEO

- App Router metadata, Open Graph, Twitter cards, JSON-LD.
- Sitemap multi-file (per directory).
- Hreflang `en` & `id`.

### 62.3 Content Engine

- Programmatic SEO untuk template comparison ("AI X vs Y") &
  vertical landing (`/for/healthcare-saas`).
- Blog publishing dipush via MDX di repo `apps/docs/content`.

### 62.4 Backlinks

- Open-source companion repos (boilerplate, sdk).
- Hacker News, Indie Hackers, Product Hunt launch plan.

## 63. Analytics Implementation

### 63.1 Tooling

- **PostHog** (cloud or self-host) untuk product analytics, funnel,
  retention, session replay.
- **Plausible** atau Cloudflare Web Analytics untuk marketing privacy-first
  metrics (P2).
- DB-internal dashboards via Grafana + Postgres views (anti-PostHog
  failure).

### 63.2 Event Schema

- Naming: `noun.verb` (`project.created`, `workflow.run.succeeded`).
- Properties wajib: `org_id`, `user_id`, `plan`, `env`.
- Schema di `packages/shared/analytics/events.ts` (Zod).

### 63.3 Privacy

- IP anonymization toggle.
- Opt-out for Enterprise (PostHog flag).
- No replay for sensitive routes (auth, billing).

### 63.4 Experimentation

- Feature flags + experiments via PostHog.
- ID hashing user → consistent assignment.

## 64. A/B Testing & Feature Flags

### 64.1 Flags

- Boolean flags untuk fitur baru (rollout %).
- Multivariate flags untuk pricing test.
- Flag definitions di-mirror di `packages/shared/flags.ts`.

### 64.2 Targeting

- Cohort by org plan, region, signup date.
- Override per org (sales-led).

### 64.3 Process

- New flag default OFF.
- Rollout 1% → 10% → 50% → 100%.
- Cleanup: flag harus dihapus dalam 30 hari setelah 100%.

## 65. Customer Support Tooling

### 65.1 Channels

- In-app help widget (Crisp / Intercom-light) — P1.
- Email (Resend inbound).
- Public docs site.
- Discord community (P0 launch).

### 65.2 Internal Admin

- Admin panel di `/__admin` (auth-gated, audit-logged).
- Fitur: impersonate (with reason + audit), refund, feature flag override,
  reissue API key, resend webhook.

### 65.3 Knowledge Base

- Mintlify atau Docusaurus.
- AI-generated FAQ dari support tickets (P1).

### 65.4 Status Page

- Statuspage.io / Better Stack.
- Auto-update dari alerts (Sev1 = incident publik).

---

# Part VII — Build Plan & Delivery

## 66. Roadmap (Phase 0–6)

### 66.1 Phase 0 — Foundation (Minggu 1–4)

**Tujuan:** Repo + infra dasar siap, satu hello-world e2e (login → buat
project → deploy hello-world).

**Scope:**

- Monorepo (pnpm + Turborepo) dengan `apps/web`, `apps/api`, `packages/db`,
  `packages/ui`, `packages/shared`.
- Postgres + Redis (Coolify managed).
- Auth (Better Auth) email + password + magic link, 2FA optional.
- Org + member + invite (Better Auth Organizations plugin).
- Billing skeleton (Stripe customer create, no charge).
- Coolify deploy untuk `apps/web` (production + preview branch).
- CI pipeline: lint, typecheck, unit test, build.
- Sentry, Loki, Grafana, PostHog wiring.
- ADR-0001 (Auth), ADR-0002 (ORM), ADR-0003 (Queue).

**Exit criteria:** `pnpm test` hijau di CI; `pnpm dev` jalan; satu staff
demo deploy hello-world.

### 66.2 Phase 1 — Core Workflow + AI (Minggu 5–10)

**Tujuan:** Workflow visual + AI calls bisa dieksekusi.

**Scope:**

- Workflow Studio (React Flow) dengan node set minimal (start, llm, http,
  branch, end).
- BullMQ queue + worker, durable run state.
- AI Gateway (OpenRouter), prompt registry P0, Langfuse trace.
- Token-based pricing (meter events).
- Run viewer dengan streaming logs.
- Knowledge base loader (file + URL ingest, chunking, embed,
  pgvector).
- Marketplace skeleton (browse, no payment yet).
- 3 template seed (Lead Gen, Support Bot, Content Engine).

**Exit criteria:** End-to-end run sebuah template "Lead Gen": form input
→ LLM → email → tersimpan di DB.

### 66.3 Phase 2 — Visual App Builder + Sandbox (Minggu 11–16)

**Tujuan:** Pengguna bisa generate app dari template + sandbox aman.

**Scope:**

- Puck visual editor + theme tokens.
- Block library v1 (header, hero, list, form, table, chart, faq, cta,
  footer).
- Sandbox layer E2B (default), policy enforcement.
- Workflow node baru: code (sandbox), data.transform.
- AI assistant intra-app ("/ask").
- Project export (zip + git push).
- Deploy preview environment per project.
- SDK v0 (TypeScript) dan CLI v0.

**Exit criteria:** User bikin SaaS sederhana (form + AI processing +
table output) tanpa nulis kode lebih dari satu callback.

### 66.4 Phase 3 — Marketplace & Creator Economy (Minggu 17–22)

**Tujuan:** Creator bisa publish & monetize template; buyer beli & install.

**Scope:**

- Stripe Connect onboarding, KYC.
- Listing flow + moderation queue + review rubric.
- Buyer flow: detail → trial → buy → install.
- Refund / dispute pipeline.
- Reviews + reports.
- Verified Creator badge program (manual issuance).
- Programmatic SEO MVP.

**Exit criteria:** 25 template live, 5 creators paid out, paid GMV
running.

### 66.5 Phase 4 — Scale, SLA, Enterprise (Minggu 23–34)

**Tujuan:** Siap kontrak Enterprise (SOC2 Type 1) + multi-region opsional.

**Scope:**

- SAML SSO + SCIM (Better Auth + plugin or custom).
- Audit log eksternal export (S3 / SIEM).
- Pgvector → opt-in self-hosted Qdrant (per Enterprise).
- Multi-region read replica + standby.
- DR runbooks, GameDay quarter cadence.
- SOC2 Type 1 audit (Vanta / Drata).
- Data residency EU (option).

**Exit criteria:** Audit SOC2 Type 1 lulus; ≥3 customers Enterprise paid.

### 66.6 Phase 5 — AI Native UX & Vertical Templates (Minggu 35–46)

**Tujuan:** AI Designer (NL → app) GA + 5 vertical bundle.

**Scope:**

- "Build me an X" input → multi-step plan + visual editor handoff.
- Inline copilot di workflow (suggest next node, refactor branch).
- Voice-to-app (P3, optional).
- Vertical packs (Healthcare-CRM, Education-LMS, Retail-Loyalty,
  Logistics-Tracker, Finance-Reporter).
- SOC2 Type 2 audit.

**Exit criteria:** AI Designer di tangan 50% Pro+, 5 vertical packs ada,
SOC2 Type 2 lulus.

### 66.7 Phase 6 — Enterprise & Compliance Plus (Minggu 47–60)

**Tujuan:** ISO 27001 + (opsional) HIPAA/GDPR posture matang.

**Scope:**

- ISO 27001 audit.
- HIPAA addendum + BAA.
- Region pinning (per-tenant placement).
- Self-hosted Enterprise package (Helm chart).
- Custom contract pricing tier.
- EU AI Act compliance per template kategori risiko.

**Exit criteria:** Pertama deal ≥$100k ARR ditandatangani.

### 66.8 Timeline Visual

```text
W1     W4   W10   W16   W22         W34         W46         W60
| P0   | P1 | P2  | P3  |    P4     |    P5     |    P6     |
| Foun.|Core|Buil.|Mkt. | Scale/SOC | AI/Vert.  | ISO/HIPAA |
```

(Estimasi; lihat 73 untuk dependensi headcount.)

## 67. Phase Deliverables (Per Phase Yang Dikirim)

Setiap fase wajib menghasilkan:

- Perubahan kode + ADR.
- Dokumen rilis (changelog publik + internal).
- Update docs.teskel.app section terkait.
- Update marketing site (landing, pricing, blog post launch).
- Sample template / contoh untuk fitur baru.
- Runbook + dashboard (observability) untuk komponen baru.
- Test pack (unit, integration, e2e, load minimum).

## 68. Detailed Epic & Story Breakdown

Berikut gambaran prioritas stories per phase. Penyimpanan resmi:
Linear / GitHub Projects.

### 68.1 Phase 0

- AUTH-001 Sign-up email + magic link.
- AUTH-002 Sign-in + 2FA optional.
- ORG-001 Create organization on signup.
- ORG-002 Invite member, role assign (owner, admin, member).
- INFRA-001 Coolify project + Postgres provisioning.
- INFRA-002 Sentry + PostHog client wiring.
- BILL-001 Stripe customer mirror.
- DEV-001 Pre-commit + lint + tsc + test in CI.

### 68.2 Phase 1

- AI-001 OpenRouter gateway + key vault.
- AI-002 Prompt registry + version + diff.
- AI-003 Langfuse tracing wrapper.
- WF-001 React Flow studio canvas.
- WF-002 Node types: start, llm, http, branch, code (basic).
- WF-003 BullMQ run queue + worker.
- WF-004 Run viewer + streaming logs.
- KB-001 Ingest URL/file → chunk → embed → pgvector.
- MKT-001 Browse marketplace, free templates only.
- TPL-001 3 seed templates Lead Gen, Support Bot, Content Engine.

### 68.3 Phase 2

- VBE-001 Puck editor mount + persistence.
- VBE-002 Block library v1 + theme tokens.
- VBE-003 AI command palette intra-app.
- SBX-001 E2B integration + policy.
- WF-005 Code node (sandbox) + data.transform node.
- DEPL-001 Per-project preview environment.
- SDK-001 TS SDK packages publish.
- CLI-001 `teskel dev`, `init`, `deploy`.
- EXP-001 Project export zip + git push.

### 68.4 Phase 3

- MKT-010 Submit listing flow.
- MKT-011 Moderation queue + reviewer UI.
- MKT-012 Stripe Connect onboarding.
- MKT-013 Buy / install flow.
- MKT-014 Refund + dispute pipeline.
- MKT-015 Reviews + reports.
- MKT-016 Verified Creator badge.
- SEO-001 Programmatic landing pages (template + use case).

### 68.5 Phase 4

- SSO-001 SAML SSO (per org).
- SSO-002 SCIM provisioning.
- AUD-001 Audit log SIEM export.
- DR-001 Cross-region read replica.
- DR-002 Backup attestation + restore drill.
- KB-010 Optional Qdrant cluster per Enterprise.
- COMP-001 SOC2 Type 1 audit close.

### 68.6 Phase 5

- AID-001 NL → multi-step plan generator.
- AID-002 Visual editor handoff.
- AID-003 Inline workflow copilot.
- VERT-001..005 5 vertical bundle.
- COMP-002 SOC2 Type 2.

### 68.7 Phase 6

- COMP-003 ISO 27001.
- COMP-004 HIPAA add-on + BAA.
- ENT-001 Region pinning.
- ENT-002 Self-host Helm chart.

## 69. MVP Cut Lines

**MVP (Phase 1 + 2):**

- Auth + Org + Billing + 1 Workflow Studio + AI Gateway + 5 Template + 1
  Visual Builder.
- Single-region (Singapore by default), Coolify deploy.
- Free tier + Pro tier (Stripe).
- No marketplace monetization (free listings only).

**Bukan MVP:**

- Marketplace paid + creator payout.
- SAML SSO.
- Multi-region.
- Vertical bundles.
- AI Designer NL→app.
- HIPAA/SOC2 Type 2.

**Cut lines kalau telat:**

- Drop Visual Builder Phase 2 → masuk Phase 2.5 (akhirnya tetap Phase 2,
  tapi block library disederhanakan).
- Drop multi-tenant RLS deepscope di MVP (tetap RLS, tapi
  cross-tenant tools dipush ke Phase 4).

## 70. DoR / DoD / Quality Gates

### 70.1 Definition of Ready (story siap masuk sprint)

- User story + acceptance criteria + UX wireframe (kalau ada UI).
- Dependency cleared (API contract, env vars, design tokens).
- Estimasi confident (≤3 days mid).
- Telemetry plan (event names, dashboards) ditulis.

### 70.2 Definition of Done (story selesai)

- Kode merged ke `main`, CI hijau.
- Unit + integration test ditambahkan.
- E2E happy-path (kalau user-facing).
- Telemetry ditambahkan (sesuai DoR).
- Docs/changelog updated.
- Pemilik fitur sign-off.

### 70.3 Quality Gates per Phase

- Phase 0: e2e smoke (login + deploy).
- Phase 1: synthetic workflow run hourly (Better Stack).
- Phase 2: load test workflow run 100 RPS.
- Phase 3: payment correctness audit (sample 100 transaksi).
- Phase 4: GameDay restore + SOC2 readiness checklist.
- Phase 5: red-team AI scenarios.
- Phase 6: full BCP/DR drill quarter.

## 71. ADRs to Write

Wajib (harus ditulis di Phase 0 atau awal phase terkait):

- ADR-0001 Auth library (Better Auth vs alternatives).
- ADR-0002 ORM (Drizzle vs Prisma).
- ADR-0003 Queue (BullMQ vs Inngest vs Trigger.dev).
- ADR-0004 Real-time (Liveblocks vs y-websocket self-host).
- ADR-0005 Visual builder (Puck vs Craft.js).
- ADR-0006 LLM observability (Langfuse vs LangSmith).
- ADR-0007 Multi-tenancy strategy (RLS vs schema per tenant).
- ADR-0008 Sandbox provider (E2B vs Daytona vs Modal).
- ADR-0009 Frontend framework (Next.js 15 RSC vs Remix).
- ADR-0010 Deployment platform (Coolify vs Northflank vs Render).

Setiap ADR memakai template di **Appendix G**.

## 72. Team Sizing & RACI

### 72.1 Team Size per Phase

- Phase 0: 3 (1 fullstack lead, 1 frontend, 1 platform).
- Phase 1–2: 6 (Lead, 2 fullstack, 1 AI, 1 platform/devops, 1 designer).
- Phase 3: 9 (+1 PM, +1 fullstack growth, +1 community).
- Phase 4: 13 (+1 SRE, +1 security/compliance, +1 DBA, +1 enterprise SE).
- Phase 5: 16 (+1 AI lead, +2 fullstack, +1 vertical SE rotational).
- Phase 6: 20 (+1 customer success, +1 contracts, +1 partner manager,
  +1 SDR).

### 72.2 RACI (Sample)

| Aktivitas | R | A | C | I |
| --- | --- | --- | --- | --- |
| Release production | SRE | CTO | PM, QA | Whole team |
| ADR baru | Author | CTO/Tech Lead | Senior eng | Team |
| Incident Sev1 | IC | CTO | All eng | Customers |
| Compliance audit prep | Compliance | CTO | Security/Legal | Sales |
| Pricing change | PM | CEO | Finance, Legal | Customers |

(R=Responsible, A=Accountable, C=Consulted, I=Informed.)

## 73. Hiring & Contractor Plan

### 73.1 Hire-First List

- Founding engineer fullstack (Phase 0).
- AI/Workflow engineer (Phase 1).
- Designer with product chops (Phase 1–2).
- Devrel/Community lead (Phase 3).
- SRE (Phase 4).
- Security & compliance lead (Phase 4).

### 73.2 Contractor Buckets

- Markup design polish.
- Pentest annual.
- SOC2/ISO consultant.
- Devrel content writer (per launch).
- Solutions engineer rotational (per vertical).

### 73.3 Geographic Strategy

- Core team Asia (Indonesia/SG).
- Hub kedua US East (timezone overlap untuk customer SE).
- All-remote, async-first; standup 30 menit harian zona inti, mingguan
  global.

### 73.4 Compensation Notes

- Equity-heavy untuk first 5; market salary + ESOP fokus.
- SOC2/ISO bonus untuk compliance lead.
- Creator partnerships paid via Stripe Connect (formal contractor invoice
  flow tetap diperlukan untuk milestone).

---

# Part VIII — Risks & Reliability

## 74. Risk Register

Format: `ID | Kategori | Deskripsi | P (1-5) | I (1-5) | Skor | Mitigasi |
Owner | Status`. P = probability, I = impact, Skor = P × I.

### 74.1 Strategic / Market

| ID | Risiko | P | I | Skor | Mitigasi | Owner |
| --- | --- | --- | --- | --- | --- | --- |
| R-S01 | Inkumben (Vercel/Replit) merilis equivalent platform | 4 | 5 | 20 | Wedge vertical + creator marketplace + harga lokal; defensible data moat; community lock-in | CEO |
| R-S02 | Pasar AI-builder saturasi sebelum kita PMF | 3 | 5 | 15 | Fokus pada vertical Indo/SEA dulu; cepat iterate dengan creator program | PM |
| R-S03 | Regulasi (EU AI Act, Indo PDP Law) menyaring fitur | 3 | 4 | 12 | Compliance roadmap fase awal; vendor agnostic; data residency | Legal |
| R-S04 | Pendanaan tidak terkumpul tepat waktu | 3 | 5 | 15 | Path ke profitability Phase 3; cost discipline; revenue first | CEO |
| R-S05 | Creator marketplace gagal capai liquidity | 3 | 4 | 12 | Seed templates internal; bayar 5 creators awal; promotion + revenue share lebih besar di awal | PM |

### 74.2 Teknis / Arsitektur

| ID | Risiko | P | I | Skor | Mitigasi | Owner |
| --- | --- | --- | --- | --- | --- | --- |
| R-T01 | Workflow runtime tidak durable, kehilangan eksekusi | 3 | 5 | 15 | BullMQ persisted state + outbox; tests load 100 RPS; replay capability | Eng Lead |
| R-T02 | Sandbox jebol → tenant lain terkena | 2 | 5 | 10 | E2B managed; egress allowlist; anti-affinity; pentest tahunan | Security |
| R-T03 | Postgres bottleneck di tabel runs | 4 | 4 | 16 | Partition by `created_at`, archive ke object storage 90 hari; read replica | DBA |
| R-T04 | RLS policy bug → cross-tenant leak | 3 | 5 | 15 | Test pyramid: row-level test + integration test khusus; review wajib 2 reviewer untuk perubahan policy | Security |
| R-T05 | Vendor lock-in (OpenRouter, E2B, Coolify) | 3 | 3 | 9 | Layer abstraksi (interface); fallback path tertulis; ADR | Architect |
| R-T06 | LLM provider perubahan breaking | 4 | 3 | 12 | Provider-agnostic gateway; prompt eval suite; alert latency/cost spike | AI Lead |
| R-T07 | Real-time service (Liveblocks) outage | 2 | 3 | 6 | Soft-degrade ke autosave + non-collab mode; doc fallback flow | Eng |
| R-T08 | Pgvector tidak scale di >50M chunks | 3 | 3 | 9 | Watcher metric; switch ke Qdrant per tenant Enterprise | DBA |

### 74.3 Operasional / Bisnis

| ID | Risiko | P | I | Skor | Mitigasi | Owner |
| --- | --- | --- | --- | --- | --- | --- |
| R-O01 | AI biaya membengkak melebihi margin | 4 | 4 | 16 | Per-org budget cap; cache hit ratio ≥30%; mode hemat (smaller models) | Finance |
| R-O02 | Stripe fee + refund chargeback merosotkan margin marketplace | 3 | 3 | 9 | Reserve account 5%; refund window ketat; fraud detection | Ops |
| R-O03 | Data breach → GDPR/PDP fine | 2 | 5 | 10 | SOC2/ISO; encryption; vault; pentest; bug bounty; cyber insurance | Security |
| R-O04 | Penalti Stripe karena policy violation creator | 2 | 4 | 8 | Moderation rubric; KYC; terms creator | Legal |
| R-O05 | Customer support backlog → churn naik | 3 | 4 | 12 | AI assistant tier 1; SLA matrix; status page transparan | CS |

### 74.4 SDM / Eksekusi

| ID | Risiko | P | I | Skor | Mitigasi | Owner |
| --- | --- | --- | --- | --- | --- | --- |
| R-H01 | Founding engineer keluar Phase 0–2 | 3 | 5 | 15 | Equity vesting; mentoring; redundansi; ADR + docs | CEO |
| R-H02 | Kompleksitas build mengakibatkan burnout | 3 | 4 | 12 | Timebox phase; on-call rotasi; PTO mandat | Eng Lead |
| R-H03 | Hire AI engineer telat | 3 | 3 | 9 | Talent pipeline jalan dari Phase 0; konsultan stand-in | CTO |

### 74.5 Reputasi / Trust

| ID | Risiko | P | I | Skor | Mitigasi | Owner |
| --- | --- | --- | --- | --- | --- | --- |
| R-R01 | Creator memuat malware via template | 2 | 5 | 10 | Pipeline review otomatis + manual; sandbox install; signing | Security |
| R-R02 | AI hallucination merugikan customer end-user | 3 | 4 | 12 | Guardrails; eval suite; user-facing disclaimers; logging | AI Lead |
| R-R03 | Bug billing → customer overcharged | 2 | 5 | 10 | Test suite billing; canary org; alert anomaly; refund SLA | Eng |

### 74.6 Cara Kerja

- Review risk register tiap bulan; tutup yang skor turun, tambah yang
  baru.
- Risiko skor ≥15 wajib punya kontingensi tertulis di Notion / Linear.
- Setiap incident Sev1 wajib menambah entri risk register baru atau
  upgrade existing.

## 75. Failure Modes & Rollback

### 75.1 Top Failure Modes

1. **Workflow run loop infinite** → guard via `max_steps` per run + budget
   per workflow. Auto-cancel + alert.
2. **AI provider 5xx storm** → circuit breaker per provider; switch ke
   alternate model; tampilkan banner ke user.
3. **DB connection pool exhausted** → PgBouncer transaction mode; alert di
   80%; auto-scale worker downward.
4. **Redis OOM** → eviction policy `allkeys-lru`; alert pre-OOM; queue
   pause publish.
5. **Webhook spam** → rate limit per source; dedup window; signature
   wajib.
6. **Coolify host failure** → snapshot harian; standby host; rebuild via
   IaC dalam ≤2 jam.
7. **Stripe outage** → checkout disabled banner; queue meter events local;
   replay setelah pulih.
8. **Sandbox provider outage** → switch ke fallback provider (Daytona);
   degrade mode "no code execution" sementara.

### 75.2 Rollback Strategy

- **App deploy:** Coolify rollback ke versi sebelumnya (button); CI tag
  semver; rollback ≤5 menit.
- **DB migration:** versi sebelumnya kompatibel (no destructive in-place
  drop); revert script dipersiapkan untuk migrasi yang berdampak.
- **Workflow runtime:** versioning workflow + flag `runtime_version`;
  ability to pin run baru ke versi lama.
- **AI prompt:** prompt registry version; quick swap pointer.
- **Feature flag:** kill switch ke 0% in <1 menit.

### 75.3 Communication

- Sev1: status page update <5 menit, customer email <30 menit.
- Sev2: status page update <30 menit.
- Postmortem dalam 5 hari kerja, publik untuk Sev1.

## 76. Test Pyramid & Strategy

### 76.1 Pyramid

```text
                ┌──────────────┐
                │   Manual /   │  10%   exploratory, UAT, audit
                │  Exploratory │
                ├──────────────┤
                │     E2E      │  10%   playwright golden flows
                ├──────────────┤
                │ Integration  │  25%   db + api + queue
                ├──────────────┤
                │     Unit     │  55%   pure functions, hooks, util
                └──────────────┘
```

### 76.2 Tools

- Unit: Vitest.
- Integration: Vitest + testcontainers (Postgres, Redis).
- E2E: Playwright (chromium + webkit), parallel.
- Visual regression: Chromatic atau Playwright trace + Percy.
- Load: k6 atau Artillery.
- Security: Semgrep + Snyk + zap (DAST P2).
- AI eval: prompt eval suite (Promptfoo / Langfuse evaluators).

### 76.3 Coverage Targets

- Unit ≥70% lines.
- Integration cover semua route P0 + queue handler.
- E2E cover 6 golden flows (Sec. 12).

### 76.4 Test Data Strategy

- Factory functions di `packages/testing/factories.ts`.
- Seed deterministik untuk e2e (`seed-test.ts`).
- Tenant isolation per test (org baru).
- Fake provider (LLM, Stripe) di test mode.

### 76.5 CI Integration

- Per-PR: lint + typecheck + unit + integration affected.
- Pre-merge: full unit + integration + e2e smoke.
- Nightly: full e2e + load smoke + security scan.
- Weekly: load test penuh + chaos test (Phase 4+).

### 76.6 Chaos & Resilience

- Tooling: Litmus / chaos-mesh (Phase 4+).
- Scenario: kill worker, drop replica, slow network LLM, fill disk.
- Cadence quarter; output → runbooks.

---

# Part IX — Reference & Appendix

## 77. Glossary

- **Aha moment** — momen pengguna pertama melihat nilai produk.
- **ADR (Architecture Decision Record)** — dokumen ringkas yang
  mencatat keputusan arsitektur, alternatif, dan konsekuensi.
- **Better Auth** — open-source TypeScript auth framework, MIT-licensed,
  default pilihan TESKEL.
- **BullMQ** — Redis-backed job queue Node.js dengan dukungan flow,
  scheduler, dan repeat.
- **C4-Light** — pendekatan dokumentasi arsitektur dengan 3–4 level
  (context, container, component, code).
- **Coolify** — open-source PaaS yang dipakai TESKEL untuk deploy
  baik aplikasi sendiri maupun customer apps.
- **Creator** — pengguna yang menerbitkan template di marketplace.
- **CRDT** — Conflict-free Replicated Data Type, dipakai Yjs untuk
  multiplayer.
- **DECIDED / DEFAULT / OPEN / LATER** — status keputusan plan.
- **DoR / DoD** — Definition of Ready / Done untuk story.
- **E2B** — sandbox provider untuk eksekusi code AI yang tepercaya.
- **Hono** — web framework Node.js minimalis, dipakai backend TESKEL.
- **JTBD** — Jobs To Be Done.
- **Langfuse** — open-source LLM observability + prompt management.
- **Meter event** — event Stripe untuk usage-based billing.
- **OpenRouter** — gateway multi-provider LLM.
- **Outbox pattern** — pola untuk transactional message publishing.
- **Puck** — visual page builder open-source berbasis React.
- **RLS (Row Level Security)** — fitur Postgres untuk pemfilteran row
  berdasarkan kebijakan.
- **SLO/SLI/SLA** — Service Level Objective / Indicator / Agreement.
- **WSDAP** — Weekly Successful Deploys & AI Productions, north star
  metric TESKEL.

## 78. Recommended Templates v1 (Seed)

Phase 1–2 wajib seed 3 template, total ekspansi target Phase 3 = 25.

| # | Nama | Kategori | Stack | Catatan |
| --- | --- | --- | --- | --- |
| 1 | AI Lead Capture & Enrichment | Marketing | Form + LLM + Resend | Phase 1 seed |
| 2 | AI Customer Support Bot | SaaS | Workflow + KB + Slack | Phase 1 seed |
| 3 | Content Engine Multi-Lang | Marketing | Workflow + LLM + S3 | Phase 1 seed |
| 4 | AI Sales Assistant | Sales | Workflow + CRM + LLM | Phase 2 |
| 5 | Inbox Triage & Reply Drafter | Productivity | Email + LLM | Phase 2 |
| 6 | Resume Screener | HR | Workflow + LLM + DB | Phase 2 |
| 7 | Doc-to-FAQ Generator | Knowledge | KB + LLM | Phase 2 |
| 8 | Smart Lead Form (no-code) | Marketing | Builder + Workflow | Phase 2 |
| 9 | Internal Tool: KPI Dashboard | Internal | Builder + DB | Phase 2 |
| 10 | Multi-Tenant SaaS Boilerplate | Foundation | Auth + Org + Bill | Phase 2 |
| 11 | Telegram/WhatsApp Bot Adapter | Integration | Workflow + WAAPI | Phase 3 |
| 12 | E-commerce Product Q&A | Retail | KB + LLM | Phase 3 |
| 13 | Doctor Appointment Booking | Healthcare | Builder + Workflow | Phase 3 |
| 14 | LMS Student Tracker | Education | Builder + DB | Phase 3 |
| 15 | Loyalty Card Manager | Retail | Builder + DB | Phase 3 |
| 16 | Logistics Tracker | Logistics | Workflow + Maps | Phase 3 |
| 17 | Finance Reporting Bot | Finance | Workflow + Sheets | Phase 3 |
| 18 | Real Estate Listing & Inquiry | Real Estate | Builder + Workflow | Phase 3 |
| 19 | Restaurant Reservation + AI Concierge | F&B | Builder + WAAPI | Phase 3 |
| 20 | NGO Donation + Receipt Bot | Non-profit | Workflow + Stripe | Phase 3 |
| 21 | Recruitment Funnel | HR | Workflow + Calendar | Phase 3 |
| 22 | AI Tutor Chat | Education | Workflow + KB | Phase 3 |
| 23 | Knowledge Base Builder | SaaS | Builder + KB | Phase 3 |
| 24 | RAG Chat for Docs | SaaS | KB + Workflow | Phase 3 |
| 25 | Mini-CRM with AI Notes | Productivity | Builder + Workflow | Phase 3 |

## 79. References & Sources

### 79.1 Frameworks & Tools

- Next.js — <https://nextjs.org/docs>.
- React 19 — <https://react.dev>.
- Hono — <https://hono.dev>.
- Drizzle ORM — <https://orm.drizzle.team>.
- Prisma — <https://www.prisma.io/docs>.
- Better Auth — <https://better-auth.com>.
- Auth.js (next-auth) — <https://authjs.dev>.
- Clerk — <https://clerk.com/docs>.
- WorkOS — <https://workos.com/docs>.
- BullMQ — <https://docs.bullmq.io>.
- Inngest — <https://www.inngest.com/docs>.
- Trigger.dev — <https://trigger.dev/docs>.
- Liveblocks — <https://liveblocks.io/docs>.
- Yjs — <https://yjs.dev>.
- React Flow — <https://reactflow.dev>.
- Puck — <https://puckeditor.com>.
- Craft.js — <https://craft.js.org>.
- E2B — <https://e2b.dev/docs>.
- Daytona — <https://www.daytona.io>.
- Modal — <https://modal.com/docs>.
- Coolify — <https://coolify.io/docs>.

### 79.2 AI & Observability

- OpenRouter — <https://openrouter.ai/docs>.
- Langfuse — <https://langfuse.com/docs>.
- LangSmith — <https://docs.smith.langchain.com>.
- Helicone — <https://docs.helicone.ai>.
- pgvector — <https://github.com/pgvector/pgvector>.
- Promptfoo — <https://www.promptfoo.dev/docs>.

### 79.3 Billing & Payment

- Stripe Billing — <https://stripe.com/docs/billing>.
- Stripe Connect — <https://stripe.com/docs/connect>.
- Stripe Meter Events — <https://stripe.com/docs/billing/subscriptions/usage-based>.

### 79.4 Standards & Compliance

- OWASP ASVS — <https://owasp.org/www-project-application-security-verification-standard/>.
- OWASP Top 10 — <https://owasp.org/www-project-top-ten/>.
- WCAG 2.2 — <https://www.w3.org/TR/WCAG22/>.
- SOC2 — <https://www.aicpa-cima.com>.
- ISO 27001 — <https://www.iso.org/standard/27001>.
- EU AI Act — <https://artificialintelligenceact.eu>.
- GDPR — <https://gdpr.eu>.
- Indonesia PDP Law — <https://peraturan.bpk.go.id/Details/229798/uu-no-27-tahun-2022>.

### 79.5 Architecture / Engineering Practices

- C4 Model — <https://c4model.com>.
- 12-Factor App — <https://12factor.net>.
- Google SRE Book — <https://sre.google/sre-book>.
- Architecture Decision Records — <https://adr.github.io>.

### 79.6 UX References

- Linear — <https://linear.app>.
- Vercel — <https://vercel.com>.
- Stripe — <https://stripe.com>.
- Notion — <https://notion.so>.
- Loom — <https://loom.com>.
- Framer — <https://framer.com>.

## 80. Appendix A — Sample DDL (Postgres)

```sql
-- Enable extensions
CREATE EXTENSION IF NOT EXISTS pgcrypto;
CREATE EXTENSION IF NOT EXISTS vector;
CREATE EXTENSION IF NOT EXISTS pg_trgm;

-- Multi-tenant context
CREATE OR REPLACE FUNCTION current_org() RETURNS uuid AS $$
  SELECT NULLIF(current_setting('app.current_org', true), '')::uuid
$$ LANGUAGE sql STABLE;

-- Core tables
CREATE TABLE orgs (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  slug text UNIQUE NOT NULL,
  name text NOT NULL,
  plan text NOT NULL DEFAULT 'free',
  region text NOT NULL DEFAULT 'sg',
  created_at timestamptz NOT NULL DEFAULT now(),
  deleted_at timestamptz
);

CREATE TABLE users (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  email citext UNIQUE NOT NULL,
  name text,
  created_at timestamptz NOT NULL DEFAULT now(),
  deleted_at timestamptz
);

CREATE TABLE memberships (
  org_id uuid NOT NULL REFERENCES orgs(id) ON DELETE CASCADE,
  user_id uuid NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  role text NOT NULL,
  created_at timestamptz NOT NULL DEFAULT now(),
  PRIMARY KEY (org_id, user_id)
);

CREATE TABLE projects (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  org_id uuid NOT NULL REFERENCES orgs(id) ON DELETE CASCADE,
  slug text NOT NULL,
  name text NOT NULL,
  type text NOT NULL,
  created_at timestamptz NOT NULL DEFAULT now(),
  UNIQUE (org_id, slug)
);

CREATE TABLE workflows (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  org_id uuid NOT NULL,
  project_id uuid NOT NULL REFERENCES projects(id) ON DELETE CASCADE,
  name text NOT NULL,
  graph jsonb NOT NULL,
  version int NOT NULL DEFAULT 1,
  created_at timestamptz NOT NULL DEFAULT now()
);

CREATE TABLE runs (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  org_id uuid NOT NULL,
  workflow_id uuid NOT NULL REFERENCES workflows(id) ON DELETE CASCADE,
  status text NOT NULL,
  trigger jsonb,
  started_at timestamptz NOT NULL DEFAULT now(),
  finished_at timestamptz,
  cost_credits numeric(12,4),
  error jsonb
) PARTITION BY RANGE (started_at);

CREATE TABLE runs_2026m01 PARTITION OF runs
  FOR VALUES FROM ('2026-01-01') TO ('2026-02-01');

CREATE TABLE templates (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  org_id uuid NOT NULL,
  slug text NOT NULL,
  name text NOT NULL,
  description text,
  current_version text,
  status text NOT NULL DEFAULT 'draft',
  created_at timestamptz NOT NULL DEFAULT now(),
  UNIQUE (org_id, slug)
);

CREATE TABLE knowledge_chunks (
  id uuid PRIMARY KEY DEFAULT gen_random_uuid(),
  org_id uuid NOT NULL,
  kb_id uuid NOT NULL,
  content text NOT NULL,
  embedding vector(1536) NOT NULL,
  metadata jsonb NOT NULL DEFAULT '{}',
  created_at timestamptz NOT NULL DEFAULT now()
);
CREATE INDEX ON knowledge_chunks USING ivfflat (embedding vector_cosine_ops);

-- RLS
ALTER TABLE projects ENABLE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation ON projects
  USING (org_id = current_org())
  WITH CHECK (org_id = current_org());

ALTER TABLE workflows ENABLE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation ON workflows
  USING (org_id = current_org())
  WITH CHECK (org_id = current_org());

ALTER TABLE runs ENABLE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation ON runs
  USING (org_id = current_org())
  WITH CHECK (org_id = current_org());

ALTER TABLE templates ENABLE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation ON templates
  USING (org_id = current_org())
  WITH CHECK (org_id = current_org());

ALTER TABLE knowledge_chunks ENABLE ROW LEVEL SECURITY;
CREATE POLICY tenant_isolation ON knowledge_chunks
  USING (org_id = current_org())
  WITH CHECK (org_id = current_org());

-- Outbox
CREATE TABLE outbox (
  id bigserial PRIMARY KEY,
  org_id uuid NOT NULL,
  topic text NOT NULL,
  payload jsonb NOT NULL,
  created_at timestamptz NOT NULL DEFAULT now(),
  published_at timestamptz
);
CREATE INDEX ON outbox (published_at);
```

## 81. Appendix B — Hono Route + RPC Client

`apps/api/src/routes/workflows.ts`:

```ts
import { Hono } from 'hono';
import { zValidator } from '@hono/zod-validator';
import { z } from 'zod';
import { auth } from '@teskel/auth';
import { db } from '@teskel/db';
import { workflows } from '@teskel/db/schema';
import { eq } from 'drizzle-orm';

export const workflowsRoute = new Hono()
  .use('*', auth())
  .get('/', async (c) => {
    const orgId = c.get('orgId');
    const list = await db.select().from(workflows).where(
      eq(workflows.orgId, orgId),
    );
    return c.json({ items: list });
  })
  .post(
    '/',
    zValidator(
      'json',
      z.object({
        name: z.string().min(1).max(80),
        projectId: z.string().uuid(),
        graph: z.unknown(),
      }),
    ),
    async (c) => {
      const orgId = c.get('orgId');
      const body = c.req.valid('json');
      const [wf] = await db.insert(workflows).values({
        orgId,
        name: body.name,
        projectId: body.projectId,
        graph: body.graph,
      }).returning();
      return c.json(wf, 201);
    },
  );
```

Mounted di `apps/api/src/index.ts`:

```ts
import { Hono } from 'hono';
import { workflowsRoute } from './routes/workflows';

const app = new Hono();
app.route('/v1/workflows', workflowsRoute);
export type AppType = typeof app;
export default app;
```

Client RPC di `packages/sdk/src/client.ts`:

```ts
import { hc } from 'hono/client';
import type { AppType } from '@teskel/api';

export const createClient = (baseUrl: string, token: string) =>
  hc<AppType>(baseUrl, {
    headers: { Authorization: `Bearer ${token}` },
  });

// Pemakaian
const client = createClient('https://api.teskel.app', token);
const res = await client.v1.workflows.$get();
const data = await res.json();
```

## 82. Appendix C — Full `template.yaml` Sample

```yaml
schema_version: 1
id: ai-support-bot-saas
slug: ai-support-bot-saas
name: AI Support Bot SaaS
version: 1.2.0
type: saas
license: proprietary

author:
  handle: dimas
  url: https://teskel.app/u/dimas

description: |
  Multi-tenant support bot SaaS dengan ingest dokumen, prompt
  templating, billing per ticket, dan dashboard monitoring.

preview_urls:
  - https://demo.teskel.app/ai-support-bot-saas

required:
  env:
    - OPENAI_API_KEY
    - STRIPE_SECRET_KEY
    - RESEND_API_KEY
  integrations:
    - stripe
    - openai
    - resend
  plan_min: starter

resources:
  app:
    framework: nextjs15
    pages_dir: app/
  workflows:
    - file: workflows/triage.json
      name: Triage incoming ticket
    - file: workflows/respond.json
      name: Generate response
    - file: workflows/escalate.json
      name: Escalate to human
  agents:
    - file: agents/support.yaml
      name: Support Agent
  knowledge_bases:
    - name: docs
      description: Customer documentation
      sources:
        - docs/**/*.md
        - faq.csv
  prompts:
    - file: prompts/system.md
      slot: agent.support.system
    - file: prompts/escalation.md
      slot: workflow.escalate.summary
  database:
    migrations_dir: db/migrations/
    seed: db/seed.ts
  branding:
    accent_token: violet
    radius_token: xl
    logo: assets/logo.svg

install:
  steps:
    - name: install dependencies
      run: pnpm install
    - name: migrate db
      run: pnpm db:migrate
    - name: seed db
      run: pnpm seed
    - name: build
      run: pnpm build
  post_install_url: /onboarding

permissions:
  - http.fetch
  - storage.read
  - storage.write
  - workflow.run
  - kb.read
  - kb.write

signing:
  algo: ed25519
  signature: 0xabc123...
  pubkey: 0xdef456...
```

## 83. Appendix D — Sample Workflow JSON

```json
{
  "schema_version": 1,
  "id": "wf_triage",
  "name": "Triage incoming ticket",
  "version": 3,
  "trigger": {
    "type": "webhook",
    "endpoint": "/hooks/ticket"
  },
  "nodes": [
    {
      "id": "start",
      "type": "start",
      "outputs": ["llm_classify"]
    },
    {
      "id": "llm_classify",
      "type": "llm",
      "config": {
        "model": "anthropic/claude-3.5-sonnet",
        "prompt_slot": "workflow.triage.classify",
        "temperature": 0.2
      },
      "outputs": ["branch_priority"]
    },
    {
      "id": "branch_priority",
      "type": "branch",
      "config": {
        "rules": [
          {
            "if": "$.classify.priority == 'urgent'",
            "then": "notify_oncall"
          },
          {
            "if": "$.classify.priority == 'normal'",
            "then": "draft_reply"
          }
        ],
        "default": "draft_reply"
      }
    },
    {
      "id": "notify_oncall",
      "type": "http",
      "config": {
        "method": "POST",
        "url": "{{ secrets.SLACK_WEBHOOK }}",
        "body": {
          "text": "Urgent ticket: {{ $.input.subject }}"
        }
      },
      "outputs": ["draft_reply"]
    },
    {
      "id": "draft_reply",
      "type": "llm",
      "config": {
        "model": "openai/gpt-4o-mini",
        "prompt_slot": "workflow.triage.reply"
      },
      "outputs": ["save"]
    },
    {
      "id": "save",
      "type": "code",
      "config": {
        "runtime": "ts",
        "code": "await ctx.db.insert('tickets', { ...input, draft });"
      },
      "outputs": ["end"]
    },
    {
      "id": "end",
      "type": "end"
    }
  ],
  "limits": {
    "max_steps": 50,
    "max_runtime_ms": 60000,
    "max_cost_credits": 0.5
  }
}
```

## 84. Appendix E — Environment Variables (Master List)

Lihat juga `.env.example` di repo. Wajib semua phase Phase 1 ke atas
kecuali ada catatan:

```text
# Core
NODE_ENV=production
APP_URL=https://app.teskel.app
API_URL=https://api.teskel.app
COOKIE_DOMAIN=.teskel.app

# Auth (Better Auth)
BETTER_AUTH_SECRET=__rotate__
BETTER_AUTH_URL=https://api.teskel.app
BETTER_AUTH_TRUSTED_ORIGINS=https://app.teskel.app
BETTER_AUTH_ENCRYPTION_KEY=__rotate__

# Database
DATABASE_URL=postgresql://...
DATABASE_POOL_MAX=20

# Redis
REDIS_URL=rediss://...

# Storage
R2_ACCOUNT_ID=...
R2_ACCESS_KEY_ID=...
R2_SECRET_ACCESS_KEY=__rotate__
R2_BUCKET=teskel-prod

# AI
OPENROUTER_API_KEY=__rotate__
LANGFUSE_PUBLIC_KEY=...
LANGFUSE_SECRET_KEY=__rotate__
LANGFUSE_HOST=https://langfuse.teskel.app

# Billing
STRIPE_SECRET_KEY=__rotate__
STRIPE_WEBHOOK_SECRET=__rotate__
STRIPE_CONNECT_CLIENT_ID=...
STRIPE_PUBLISHABLE_KEY=...

# Email
RESEND_API_KEY=__rotate__
EMAIL_FROM_DOMAIN=teskel.app

# Sandbox
E2B_API_KEY=__rotate__
DAYTONA_API_KEY=__rotate__   # P1
MODAL_TOKEN_ID=...           # P1
MODAL_TOKEN_SECRET=__rotate__

# Real-time
LIVEBLOCKS_SECRET_KEY=__rotate__

# Observability
SENTRY_DSN=https://...
SENTRY_AUTH_TOKEN=__rotate__
LOKI_PUSH_URL=https://...
GRAFANA_URL=https://...
POSTHOG_KEY=...
POSTHOG_HOST=https://app.posthog.com
OTEL_EXPORTER_OTLP_ENDPOINT=https://otel.teskel.app

# Deployment
COOLIFY_API_URL=https://coolify.teskel.app
COOLIFY_API_TOKEN=__rotate__

# Misc
INTERNAL_HMAC_SECRET=__rotate__
FEATURE_FLAG_PROVIDER=posthog
```

Catatan: gunakan `__rotate__` placeholder untuk wajib di-rotate min 90
hari (lihat 49.4).

## 85. Appendix F — Pre-Launch & Launch Checklist

### 85.1 Engineering

- [ ] Status page live + test publish.
- [ ] On-call rotation tegak; PagerDuty/OpsGenie alert flow uji-coba.
- [ ] Backups nightly verified (restore drill < 24 jam terakhir).
- [ ] Migration revert script siap.
- [ ] Feature flag launch toggles dipisah dari beta toggles.
- [ ] Synthetic monitor menyentuh semua endpoint kritis.
- [ ] Load test 5x peak diselesaikan.
- [ ] Sentry release tag + sourcemaps uploaded.

### 85.2 Security

- [ ] Pentest report close, P0/P1 fixed.
- [ ] CSP, HSTS, headers OK (Securityheaders.com A+).
- [ ] Secrets vault audit (no plaintext di repo).
- [ ] DPA + ToS published & legally reviewed.
- [ ] GDPR DSR pipeline siap (export + delete dalam 30 hari).

### 85.3 Product / UX

- [ ] Onboarding A/B test selesai; varian default ditentukan.
- [ ] Pricing & checkout uji-coba di live mode.
- [ ] Empty states + error pages reviewed.
- [ ] Accessibility audit Pa11y/axe lulus.
- [ ] Localization id/en QA pass.

### 85.4 Marketing / GTM

- [ ] Landing pages live + SEO meta valid.
- [ ] Blog post launch + case study (≥2).
- [ ] Product Hunt + Hacker News + Indie Hackers schedule.
- [ ] Email list welcome flow live.
- [ ] Social copy + visual ready (en + id).

### 85.5 Customer Success

- [ ] Support docs & FAQ published.
- [ ] Help widget connected.
- [ ] Discord rules + onboarding bot ready.
- [ ] SLA dashboard internal live.

## 86. Appendix G — RFC / ADR Template

```markdown
# ADR-NNNN: <Judul keputusan>

- Tanggal: YYYY-MM-DD
- Status: Proposed | Accepted | Superseded by ADR-NNNN | Deprecated
- Owner: <nama>
- Reviewer: <nama-nama>

## Konteks
Jelaskan masalah, kebutuhan, dan batasan. Sertakan data + tautan
relevan.

## Pilihan Yang Dipertimbangkan
1. <Opsi A> — pros/cons.
2. <Opsi B> — pros/cons.
3. <Opsi C> — pros/cons.

## Keputusan
Pilih satu opsi. Jelaskan singkat alasan.

## Konsekuensi
- Dampak positif (developer ergonomics, biaya, performa, kompatibilitas).
- Dampak negatif (lock-in, learning curve, migrasi).
- Rencana mitigasi.
- Rencana revisit (kapan ditinjau ulang).

## Referensi
- Tautan dokumentasi.
- Studi kasus.
- ADR terkait.
```

# Part X — Build Walkthrough (Senior Fullstack Playbook)

> Bagian ini adalah panduan eksekusi *minggu per minggu* dari Phase 0 ke
> production launch dan day-2 operations. Asumsi pembaca: senior fullstack
> engineer yang sudah memahami stack di Bagian III. Untuk konteks
> bisnis/produk, lihat Part I–II; untuk arsitektur, lihat Part III; untuk
> roadmap fase, lihat Sec. 66.

## 87. Pre-Flight Checklist (Sebelum Mulai Coding)

### 87.1 Akun & Layanan Yang Wajib Disiapkan

| Layanan | Tujuan | Plan awal | Catatan |
| --- | --- | --- | --- |
| GitHub Org | Source control, CI/CD, ADR | Team ($4/dev) | Aktifkan branch protection dan required checks |
| Vercel atau Coolify host | Deploy app `web` | Coolify VPS (1× 8 vCPU / 16GB) | Coolify default; Vercel hanya jika butuh edge |
| PostgreSQL (managed) | DB primary + standby | Neon / Supabase / Render | Pilih region SG (default tenant Indo/SEA) |
| Redis (managed) | Queue + cache | Upstash / Render | Min 1GB, persistence ON |
| Cloudflare R2 | Storage objek | Pay-as-you-go | Buat 3 bucket: `teskel-prod`, `teskel-staging`, `teskel-templates` |
| Cloudflare DNS + WAF | DNS, TLS, anti-DDoS | Free + Pro $20 | Pasang WAF rules dari Phase 1 |
| Stripe | Billing + Connect | Standard | Aktifkan Stripe Tax sandbox saat Phase 3 |
| Resend | Email transaksional | Free → $20 | Verifikasi domain `teskel.app` (SPF, DKIM, DMARC) |
| OpenRouter | AI gateway | Pay-as-you-go | Set spend limit & alert |
| Langfuse | LLM observability | Cloud free → self-host Phase 4 | DSN ke `LANGFUSE_*` |
| Sentry | Errors + tracing | Team plan | DSN per environment |
| Grafana Cloud | Logs + metrics | Free → Pro | Atau self-host Loki+Grafana di Coolify |
| PostHog | Product analytics | Cloud free → self-host opsional | EU region untuk data residency |
| E2B | Sandbox execution | Pay-as-you-go | Pasang `E2B_API_KEY` |
| Liveblocks | Realtime collab | Free → Starter | Pasang `LIVEBLOCKS_SECRET_KEY` |
| Better Stack atau Statuspage | Uptime + status page | Starter | Konek ke alert pipeline |
| 1Password / Doppler | Secret manager team | Team plan | Single source of truth env |
| Linear / GitHub Projects | PM/issue tracker | Free | Naming `TESKEL-*` |
| Notion atau Outline | Docs internal + ADR | Free | Folder `Engineering/ADR` |
| Discord | Community channel | Free | Setup Phase 1, public Phase 3 |

### 87.2 Tooling Lokal (Mesin Engineer)

```bash
# Versi pinned (lihat Sec. 17.2)
mise use --global node@20.18.0
mise use --global pnpm@9.12.3
mise use --global postgresql@16.6
mise use --global redis@7.4

# CLIs
npm i -g @stripe/stripe-cli wrangler vercel coolify-cli supabase   # opsional
brew install gh git-lfs jq direnv pre-commit
brew install --cask docker

# IDE
# VS Code dengan ekstensi: ESLint, Prettier, Biome, Drizzle Snippets, Tailwind CSS,
# Hono, GitLens, GitHub Copilot atau Continue.dev
```

### 87.3 Kebijakan Tim & Proses Yang Diset Lebih Dulu

- Branching: trunk-based dengan short-lived feature branches; merge via PR.
- Commit convention: Conventional Commits (`feat:`, `fix:`, `chore:`).
- PR template di `.github/pull_request_template.md` (DoD, screenshots,
  rollback plan, telemetry).
- Code owners di `CODEOWNERS` (lihat 17.5).
- Sprint cadence: 1 minggu (lihat Sec. 70).
- Standup async di Linear/Slack jam 09:00 WIB.

### 87.4 Rekening, Legal & Compliance Awal

- Entitas legal aktif (PT/Inc) supaya bisa buka Stripe live.
- Domain `teskel.app` (atau `.ai` / `.dev`) terdaftar + DNSSEC ON.
- Privacy Policy, Terms of Service, DPA template draft (Notion).
- Cyber insurance quote (akan aktif Phase 3).
- DPO / Privacy lead ditunjuk sejak Day 0 (orang internal).

### 87.5 Bootstrap Day-0 (Hari Sebelum Mulai Phase 0)

```bash
# 1. Init repo + branch protection
gh repo create yahyaxbt/teskel --private --description "TESKEL platform"
gh api repos/yahyaxbt/teskel/branches/main/protection -X PUT -f required_status_checks.strict=true ...

# 2. Tag baseline (kosong) supaya release semver bisa dimulai dari 0.1.0
git tag v0.0.0 && git push --tags

# 3. Provision Postgres + Redis di Coolify / managed; simpan connection
#    string ke 1Password vault `TESKEL/dev`

# 4. Buat 3 environment di Coolify: dev (preview), staging, prod
# 5. Setel DNS app.teskel.app → Coolify load balancer; aktifkan TLS

# 6. Set 1Password vault structure:
#    TESKEL/
#      ├── dev/
#      ├── staging/
#      ├── prod/
#      └── shared/
#    Tambahkan semua env wajib (lihat Sec. 84).
```

Exit gate Day-0: setiap engineer bisa `pnpm dev` di mesin lokal hingga
hello-world Next.js terlihat di `http://localhost:3000`.

## 88. Phase 0 Walkthrough — Foundation (Minggu 1–4)

Tujuan: arsitektur dasar berdiri (monorepo, DB, auth, deploy, CI),
hello-world deploy ke prod.

### 88.1 Minggu 1 — Monorepo Init + DB Schema Awal

**Day 1: scaffold monorepo.**

```bash
mkdir teskel && cd teskel
pnpm init -y
pnpm add -wD turbo typescript @types/node tsx prettier
pnpm add -wD eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin

mkdir -p apps/web apps/api packages/db packages/ui packages/shared \
         packages/auth packages/ai packages/queue packages/sdk \
         packages/testing infra docs

# pnpm-workspace.yaml
cat > pnpm-workspace.yaml <<EOF
packages:
  - "apps/*"
  - "packages/*"
EOF

# turbo.json minimal (lihat Sec. 17.4)
cat > turbo.json <<EOF
{
  "\$schema": "https://turbo.build/schema.json",
  "tasks": {
    "build": { "dependsOn": ["^build"], "outputs": ["dist/**", ".next/**"] },
    "dev": { "cache": false, "persistent": true },
    "lint": {},
    "typecheck": { "dependsOn": ["^typecheck"] },
    "test": { "dependsOn": ["^build"], "outputs": ["coverage/**"] }
  }
}
EOF
```

**Day 2: scaffold `apps/web` (Next.js 15) dan `apps/api` (Hono).**

```bash
# Web
cd apps/web
pnpm dlx create-next-app@latest . --ts --tailwind --eslint --app \
  --src-dir --import-alias "@/*"
pnpm add @tanstack/react-query zustand framer-motion lucide-react

# API
cd ../api
pnpm init -y
pnpm add hono @hono/node-server @hono/zod-validator zod
pnpm add -D @types/node tsx
mkdir src && cat > src/index.ts <<EOF
import { Hono } from 'hono';
const app = new Hono();
app.get('/healthz', (c) => c.json({ ok: true }));
export default app;
EOF
```

**Day 3-5: DB + Drizzle.**

```bash
cd ../../packages/db
pnpm init -y
pnpm add drizzle-orm postgres
pnpm add -D drizzle-kit

# schema.ts (lihat Sec. 21 + Appendix A)
# Buat tabel orgs, users, memberships dulu
pnpm drizzle-kit generate
pnpm drizzle-kit push  # dev only

# packages/db/index.ts ekspor `db` instance
```

**Akhir Minggu 1:**

- [ ] `pnpm dev` jalan di web + api.
- [ ] Connection ke Postgres prod bisa via `psql $DATABASE_URL`.
- [ ] Drizzle migration pertama applied.
- [ ] `apps/web/src/app/page.tsx` menampilkan "TESKEL".

### 88.2 Minggu 2 — Auth (Better Auth) + Org

**Day 1-2: install Better Auth.**

```bash
cd packages/auth
pnpm init -y
pnpm add better-auth better-auth/plugins
pnpm add -D drizzle-kit

# auth.ts: konfigurasi adapter Drizzle + plugin (organization, two-factor,
# passkey, magic-link)
```

`packages/auth/src/server.ts`:

```ts
import { betterAuth } from 'better-auth';
import { drizzleAdapter } from 'better-auth/adapters/drizzle';
import { organization, twoFactor, magicLink } from 'better-auth/plugins';
import { db } from '@teskel/db';

export const auth = betterAuth({
  database: drizzleAdapter(db, { provider: 'pg' }),
  emailAndPassword: { enabled: true, requireEmailVerification: true },
  plugins: [
    organization({
      allowUserToCreateOrganization: true,
      organizationLimit: 5,
    }),
    twoFactor(),
    magicLink({
      sendMagicLink: async ({ email, url }) => {
        await sendEmail(email, 'magic-link', { url });
      },
    }),
  ],
});
```

**Day 3-4: integrasi auth di web + api.**

- Mount `/api/auth/*` route di Hono / Next route handler.
- Buat helper `getSession()` dan `requireSession()`.
- Sign-up + sign-in pages di web.
- Organization create wizard pada signup.

**Day 5: RLS dasar.**

- Buat `current_org()` function, enable RLS pada tabel `projects`,
  `workflows`, `runs`, `templates` (lihat Appendix A).
- Wrap setiap API request dengan middleware `withTenant(orgId)`.

**Akhir Minggu 2:**

- [ ] User bisa sign-up email+password dan magic-link.
- [ ] User pertama otomatis bikin org.
- [ ] Cross-tenant test: sebagai user Org B tidak bisa baca data Org A
  (write integration test).

### 88.3 Minggu 3 — Coolify Deploy + CI/CD

**Day 1-2: dockerize + Coolify.**

`apps/web/Dockerfile`:

```dockerfile
FROM node:20-alpine AS base
RUN corepack enable
WORKDIR /app
COPY pnpm-lock.yaml package.json pnpm-workspace.yaml turbo.json ./
COPY apps/web ./apps/web
COPY packages ./packages
RUN pnpm install --frozen-lockfile && pnpm --filter web build

FROM node:20-alpine
WORKDIR /app
COPY --from=base /app /app
EXPOSE 3000
CMD ["pnpm", "--filter", "web", "start"]
```

- Buat resource `web` dan `api` di Coolify, point ke repo + branch.
- Set environment variables (semua dari Sec. 84) lewat Coolify UI atau
  Doppler integration.
- Aktifkan auto-deploy on push to `main`.

**Day 3-4: GitHub Actions.**

`.github/workflows/ci.yml`:

```yaml
name: CI
on:
  pull_request:
  push:
    branches: [main]
jobs:
  ci:
    runs-on: ubuntu-latest
    services:
      postgres:
        image: postgres:16
        env: { POSTGRES_PASSWORD: postgres }
        ports: ["5432:5432"]
        options: >-
          --health-cmd pg_isready
      redis:
        image: redis:7
        ports: ["6379:6379"]
    steps:
      - uses: actions/checkout@v4
      - uses: pnpm/action-setup@v3
        with: { version: 9 }
      - uses: actions/setup-node@v4
        with: { node-version: 20, cache: pnpm }
      - run: pnpm install --frozen-lockfile
      - run: pnpm turbo run lint typecheck test build
```

**Day 5: branch protection + secrets.**

- Required checks: `ci`.
- Reviewers: minimum 1.
- Set repository secrets dari 1Password (vault `TESKEL/shared/ci`).

**Akhir Minggu 3:**

- [ ] Push ke `main` auto-deploy ke `https://app.teskel.app`.
- [ ] PR build hijau dalam ≤6 menit.
- [ ] Preview environment per PR aktif (Coolify).

### 88.4 Minggu 4 — Observability + ADR + Exit Gate

**Day 1-2: wiring observability.**

```bash
# Sentry
pnpm add @sentry/nextjs @sentry/node
# init di apps/web/sentry.{client,server}.config.ts dan apps/api/src/sentry.ts

# OpenTelemetry
pnpm add @opentelemetry/api @opentelemetry/sdk-node \
         @opentelemetry/exporter-trace-otlp-http
# init di apps/api/src/otel.ts (lihat 39.1)

# PostHog
pnpm add posthog-js posthog-node
# init di apps/web/src/lib/analytics.ts (lihat 63)
```

**Day 3: dashboards & alerts.**

- Grafana Cloud dashboard "Phase 0 Health": HTTP 5xx rate, latency p95,
  DB connections, Redis memory, error rate per service.
- Alert: 5xx > 1% selama 5 menit → PagerDuty Sev2.

**Day 4: ADR initial batch.**

- ADR-0001 Auth (Better Auth).
- ADR-0002 ORM (Drizzle, Prisma sebagai fallback).
- ADR-0003 Queue (BullMQ; rationale untuk Inngest sebagai jalur kedua).
- ADR-0009 Frontend framework.
- ADR-0010 Deployment platform.

**Day 5: exit gate Phase 0.**

| Item | Verifikasi |
| --- | --- |
| Hello-world deploy | Buka `https://app.teskel.app` |
| Auth e2e | Sign-up + sign-in + 2FA opt-in |
| Org isolation | Test cross-tenant Postgres |
| CI hijau | Lihat last `main` run |
| Observability | Sentry event muncul saat trigger error sample |
| 5 ADR draft | Notion `Engineering/ADR` |

Tag release: `v0.1.0` (Phase 0 done).

## 89. Phase 1 Walkthrough — Core Workflow + AI (Minggu 5–10)

Tujuan: workflow visual + AI calls bisa dieksekusi end-to-end.

### 89.1 Minggu 5 — AI Gateway + Prompt Registry

**Implementasi `packages/ai`.**

```ts
// packages/ai/src/gateway.ts
import { OpenAI } from 'openai';
const openai = new OpenAI({
  apiKey: process.env.OPENROUTER_API_KEY,
  baseURL: 'https://openrouter.ai/api/v1',
});

export const aiClient = {
  chat: async ({ model, messages, traceId, orgId }) => {
    const res = await openai.chat.completions.create({
      model,
      messages,
      // tagging untuk Langfuse
      extra_headers: { 'X-Trace-Id': traceId, 'X-Org-Id': orgId },
    });
    await meterUsage(orgId, model, res.usage);
    await langfuse.trace({ id: traceId, model, messages, response: res });
    return res;
  },
};
```

**Prompt registry.**

- Tabel `prompts(id, slot, version, content, meta, created_at)`.
- API CRUD untuk prompt.
- UI Studio: list slots, lihat versi, diff.
- Helper `getPrompt(slot)` yang resolve versi terbaru atau yang dipin.

**Eval suite (Promptfoo).**

- File `evals/prompts/<slot>.yaml`.
- Run di GitHub Actions (cron daily).
- Threshold pass: regression detector dari golden set.

### 89.2 Minggu 6 — Workflow Studio (React Flow)

**Frontend.**

```bash
pnpm --filter web add reactflow @xyflow/react zustand
```

- Komponen `<WorkflowEditor/>` di `apps/web/src/features/workflow/`.
- Custom node types: `start`, `llm`, `http`, `branch`, `code`, `end`.
- Sidebar palette + properties panel.
- Auto-save graph ke API setiap 5 detik.

**Backend.**

- Route `POST /v1/workflows`, `PATCH /v1/workflows/:id` (validate Zod).
- Tabel `workflows(... graph jsonb ...)` — lihat Sec. 21 + Appendix A.

### 89.3 Minggu 7 — BullMQ + Worker + Run Viewer

```bash
pnpm --filter @teskel/queue add bullmq ioredis
```

```ts
// packages/queue/src/queues.ts
import { Queue } from 'bullmq';
const connection = { host: REDIS_HOST, port: REDIS_PORT };
export const workflowsQueue = new Queue('workflows', { connection });

// packages/queue/src/worker.ts
import { Worker } from 'bullmq';
import { runWorkflow } from '@teskel/runner';

new Worker('workflows', async (job) => {
  await runWorkflow(job.data);
}, { connection, concurrency: 8 });
```

- Endpoint `POST /v1/workflows/:id/runs` enqueue job.
- Status streaming via Server-Sent Events `/v1/runs/:id/stream`.
- Run viewer UI menampilkan node-by-node trace + cost.

### 89.4 Minggu 8 — Knowledge Base + pgvector

```sql
CREATE EXTENSION IF NOT EXISTS vector;
-- knowledge_chunks (lihat Appendix A)
```

- Service ingest: file upload → chunk (1000 tokens, overlap 100) → embed
  → upsert pgvector.
- Query: cosine top-k + filter `org_id = current_org()`.
- Rate limit ingest per org (10MB/menit free, 100MB/menit pro).

### 89.5 Minggu 9 — 3 Seed Templates

- Template 1: AI Lead Capture & Enrichment.
- Template 2: AI Customer Support Bot.
- Template 3: Content Engine Multi-Lang.

Setiap template ditulis sebagai folder di `seeds/templates/<slug>/`
mengikuti spec di Sec. 58 + Appendix C.

Loader di `packages/template-engine` parse manifest, validate, install ke
DB target org.

### 89.6 Minggu 10 — Marketplace Skeleton + Exit Gate

- Listing browse (`/marketplace`) — query templates `status=published`.
- Template detail page (`/templates/[slug]`) — README, screenshots,
  pricing, install button.
- Free install button → trigger loader.
- Tag rilis `v0.2.0`.

**Exit gate Phase 1:**

| Item | Verifikasi |
| --- | --- |
| AI workflow run | "Lead Gen" template bisa dieksekusi end-to-end |
| Logs trace | Langfuse menampilkan call AI dengan cost & latency |
| Run history | UI menampilkan riwayat 10 runs terakhir |
| KB ingest | File 5MB bisa di-ingest dan ditanya via prompt |
| Marketplace browse | 3 template muncul, install ke org sandbox |
| Synthetic monitor | Better Stack jalankan run hourly |

## 90. Phase 2 Walkthrough — Visual Builder + Sandbox (Minggu 11–16)

Tujuan: pengguna bisa generate app dari template, sandbox aman, SDK + CLI
tersedia.

### 90.1 Minggu 11 — Puck Integration

```bash
pnpm --filter web add @measured/puck
```

- Mount editor di `/projects/[id]/builder`.
- Persist data Puck ke tabel `pages(... data jsonb ...)`.
- Render output via `<Render data={data} config={config} />` di route
  publik project.

### 90.2 Minggu 12 — Block Library v1 + Theme Tokens

- Block: header, hero, feature-grid, list, form, table, chart, faq, cta,
  footer.
- Theme tokens dipakai: `accent`, `radius`, `density` dari org settings.
- AI command palette `/ask` — generate blok dari deskripsi natural.

### 90.3 Minggu 13 — E2B Sandbox + Workflow Code Node

```bash
pnpm --filter @teskel/runner add e2b @e2b/code-interpreter
```

- Wrapper `runInSandbox(code, { timeout, memory, egressAllowlist })`.
- Workflow node `code`: jalankan TS/Python di E2B.
- Logging stdout/stderr ke run trace.
- Policy enforcement (egress, max runtime, max memory) dari Sec. 32.4.

### 90.4 Minggu 14 — Project Export + Per-Project Preview

- `POST /v1/projects/:id/export` → zip + push ke repo Git buyer (jika
  set).
- Per-project preview environment via Coolify project API.
- Custom domain (CNAME) opsional Phase 3.

### 90.5 Minggu 15 — SDK + CLI v0

```bash
# packages/sdk
pnpm init -y
pnpm add hono
# expose typed RPC client (lihat Appendix B)

# packages/cli
pnpm add commander conf undici
# command: teskel login | init | dev | deploy | template validate
```

- Publish `@teskel/sdk` dan `teskel-cli` ke npm (private scope dulu).

### 90.6 Minggu 16 — AI Assistant Intra-App + Exit Gate

- Pengguna Pro+ punya copilot di workspace ("/ask" anywhere).
- Copilot bisa: jelaskan workflow, suggest next node, apply patch (PR-
  style).
- Tag rilis `v0.3.0`.

**Exit gate Phase 2:**

| Item | Verifikasi |
| --- | --- |
| Visual app | User bikin SaaS sederhana tanpa nulis kode > 1 callback |
| Sandbox | Code node sandbox lulus pen-test internal (no host access) |
| SDK install | `npm i @teskel/sdk` di proyek eksternal — type-safe |
| CLI deploy | `teskel deploy` selesai dalam ≤2 menit |
| Load test | 100 RPS workflow execution ≤500ms p95 |

## 91. Phase 3 Walkthrough — Marketplace + Creator Economy (Minggu 17–22)

Tujuan: creator bisa publish & monetize template; buyer beli & install
dengan refund/dispute pipeline.

### 91.1 Minggu 17 — Stripe Connect + KYC

```bash
pnpm --filter @teskel/billing add stripe
```

- Onboarding Connect Standard via redirect (lihat 57.1).
- Webhook handler: `account.updated`, `payout.paid`, `charge.dispute.created`.
- Tabel `creator_profiles(... stripe_account_id, kyc_status ...)`.

### 91.2 Minggu 18 — Listing Flow + Moderation

- Form publish (5 step wizard): metadata, screenshots, pricing,
  permissions, signing.
- Tabel `listings(... status ...)` enum: `draft`, `submitted`,
  `in_review`, `approved`, `rejected`, `live`.
- Moderation queue UI internal (`/__admin/moderation`).
- Pipeline otomatis (Sec. 59.2) dijalankan di queue.

### 91.3 Minggu 19 — Buy / Install Flow

- Pricing display: free / one-time / subscription.
- Stripe Checkout (one-time) atau Subscription untuk recurring.
- Webhook `checkout.session.completed` → buat `licenses` row → trigger
  install.

### 91.4 Minggu 20 — Refund + Reviews + Reports

- 7-hari refund window: API `POST /v1/orders/:id/refund` issue refund di
  Stripe + invalidate license.
- Reviews 1-5 stars, body teks.
- Report flag: spam, scam, harassment, broken — masuk queue moderasi.

### 91.5 Minggu 21 — Verified Badge + Creator Analytics

- Application form Verified Creator (manual review tim TESKEL).
- Dashboard analytics creator: views, install conv, revenue, refunds,
  rating.

### 91.6 Minggu 22 — SEO MVP + Exit Gate

- Programmatic landing per template + creator profile.
- Sitemap multi-file.
- Hreflang en/id.
- Tag rilis `v0.4.0`.

**Exit gate Phase 3:**

| Item | Verifikasi |
| --- | --- |
| 25 template live | Marketplace count |
| 5 creator paid out | Stripe Connect dashboard |
| GMV running | Stripe weekly report |
| Refund SLA | <2 menit dari klik tombol |
| Moderation SLA | First response 3 hari, decision 7 hari |

## 92. Phase 4 Walkthrough — Scale, SLA, Enterprise (Minggu 23–34)

Tujuan: siap kontrak Enterprise + SOC2 Type 1 + multi-region opsional.

### 92.1 Minggu 23–24 — SAML SSO + SCIM

- Aktifkan plugin SAML di Better Auth (`@better-auth/sso`).
- Per-org SSO config (Idp metadata URL, certificate).
- SCIM endpoint untuk auto-provisioning user.

### 92.2 Minggu 25–26 — Audit Log Eksternal + SIEM Export

- Tabel `audit_log(... actor, target, verb, payload, signed_hash ...)`.
- Hash chain (Merkle) supaya bisa dideteksi tamper.
- Export S3 hourly + relay ke SIEM customer (Splunk / Datadog).

### 92.3 Minggu 27–28 — Multi-Region Read Replica + Standby

- Provision Postgres standby di region kedua (US East / EU).
- Read replica untuk reporting / analytics.
- Failover runbook tested via GameDay.

### 92.4 Minggu 29–30 — DR Runbooks + GameDay

- Runbook: kehilangan primary DB, kehilangan Redis, region outage,
  sandbox provider outage, Stripe outage.
- GameDay quarter pertama: simulasikan kehilangan primary, ukur RTO
  aktual, perbaiki gap.

### 92.5 Minggu 31–34 — SOC2 Type 1 Audit Close

- Pilih auditor (BDO, A-LIGN, atau via Vanta/Drata).
- Kumpulkan evidence (logs, screenshots, policy docs).
- Remediation gap (akses kontrol, change management, etc.).
- Audit report tertanda → push ke security trust center.
- Tag rilis `v1.0.0` (GA Enterprise).

**Exit gate Phase 4:**

| Item | Verifikasi |
| --- | --- |
| SOC2 Type 1 | Report file |
| ≥3 customer Enterprise paid | Stripe / contracts |
| Failover RTO ≤30 menit | GameDay metric |
| SAML SSO live | 1 customer pakai |
| SCIM provisioning | 1 customer pakai |

## 93. Phase 5 Walkthrough — AI Native UX & Vertical Templates (Minggu 35–46)

Tujuan: AI Designer (NL → app) GA + 5 vertical bundle siap.

### 93.1 Minggu 35–36 — NL → App Planner

- Input: deskripsi natural.
- Output: multi-step plan JSON (entitas, halaman, workflow, prompts,
  data model).
- UI handoff: review plan → klik "Generate" → spawn project + workflow +
  blocks otomatis.

### 93.2 Minggu 37–38 — Inline Workflow Copilot

- Suggestion engine di canvas: "next node yang sering dipakai", "refactor
  branch ini".
- Apply patch reversible (history).

### 93.3 Minggu 39–43 — Vertical Packs

| Vertical | Komponen kunci |
| --- | --- |
| Healthcare-CRM | Pasien CRUD, SOAP notes, appointment, BPJS adapter |
| Education-LMS | Course, enrollment, quiz, gamifikasi |
| Retail-Loyalty | Member, point, redeem, kupon, integrasi POS |
| Logistics-Tracker | Tracking number, geo-event, ETA, PDF surat jalan |
| Finance-Reporter | Konsolidasi PnL, dashboard sheet, export PDF |

Setiap vertical pack = 1 template + 3-5 workflow + theme + content.

### 93.4 Minggu 44–46 — SOC2 Type 2 Audit Close

- 6-12 bulan window observation (mulai dari saat Type 1 close).
- Continuous control evidence (Vanta/Drata otomatis).
- Audit report Type 2 close.
- Tag rilis `v1.5.0`.

**Exit gate Phase 5:**

| Item | Verifikasi |
| --- | --- |
| AI Designer adoption | ≥50% Pro+ user pakai sekali |
| 5 vertical pack | Marketplace listed |
| SOC2 Type 2 | Report file |

## 94. Phase 6 Walkthrough — Enterprise & Compliance Plus (Minggu 47–60)

Tujuan: ISO 27001 + (opsional) HIPAA/GDPR posture + self-host package.

### 94.1 Minggu 47–50 — ISO 27001

- Gap analysis (sebagian besar overlap dengan SOC2).
- Implementasi kontrol baru (asset management, BCP, supplier mgmt).
- Stage 1 audit → remediation → Stage 2 audit.

### 94.2 Minggu 51–53 — HIPAA Add-on + BAA

- Tambah BAA template, sign per customer.
- Encryption at rest field-level untuk PHI.
- Audit log enhanced (per-record access).
- HIPAA training internal team.

### 94.3 Minggu 54–55 — Region Pinning

- `tenant.region` field; routing per request ke region yang sesuai.
- Data residency report untuk customer.

### 94.4 Minggu 56–57 — Self-Host Helm Chart

- Containerize semua service.
- Helm chart `teskel/teskel` di registry.
- Dokumentasi `docs.teskel.app/self-host`.

### 94.5 Minggu 58–60 — EU AI Act Mapping + GA Enterprise+

- Klasifikasi template per kategori risiko EU AI Act.
- Conformity assessment dokumen (high-risk only).
- Tag rilis `v2.0.0` (Enterprise Plus GA).

**Exit gate Phase 6:**

| Item | Verifikasi |
| --- | --- |
| ISO 27001 cert | Certificate file |
| ≥1 deal HIPAA | Signed BAA |
| Self-host customer | ≥1 customer running on-prem |
| EU AI Act mapping | All templates labeled |

## 95. Final Cutover & Public Launch Playbook

> Catatan: "Final" di sini = momen GA besar (mis. akhir Phase 1 Public
> Beta, akhir Phase 3 GA monetized, atau akhir Phase 4 Enterprise GA).
> Playbook ini berlaku setiap kali terjadi cutover skala besar.

### 95.1 T-30 Hari — Code Freeze & Audit

- [ ] Soft freeze: hanya P0 fix masuk `main`.
- [ ] Pen-test dijadwal & dimulai.
- [ ] Load test 5x peak diselesaikan (lihat Sec. 43.2).
- [ ] Backup + restore drill.
- [ ] DPA, Privacy Policy, ToS final review legal.
- [ ] Marketing copy + landing page A/B variant lock.

### 95.2 T-14 Hari — Final Pen-Test Close + Marketing Asset

- [ ] Pen-test report: P0/P1 close, P2 ditrack di Linear.
- [ ] Status page live di `status.teskel.app`.
- [ ] On-call rotation diumumkan ke tim.
- [ ] Press kit + screenshots ready.
- [ ] Email cadence di Resend: warm-up D-7, D-3, D-1, D-0.

### 95.3 T-7 Hari — Dress Rehearsal

- [ ] Run launch sequence di staging dengan timeline aktual.
- [ ] Simulasikan Sev1 (`break-glass`): rotate Stripe key, restore DB,
  switch sandbox provider.
- [ ] Test customer flow: signup → onboarding → first deploy → first
  workflow → first invoice.
- [ ] Latihan tim CS jawab 50 ticket sample.

### 95.4 T-1 Hari — GameDay + Go/No-Go

- [ ] GameDay: kill primary worker, drop standby DB, simulate AI provider
  outage. Pulih dalam SLA?
- [ ] Final go/no-go meeting (CEO + CTO + Eng Lead + PM + CS Lead).
- [ ] Lock prod: only on-call dapat deploy.
- [ ] Banner "scheduled launch" diumumkan di blog + email.

### 95.5 T-0 — Launch Sequence (Hour by Hour)

| Jam | Aksi | Owner | Verifikasi |
| --- | --- | --- | --- |
| H-2 | Deploy versi GA ke prod (canary 10%) | SRE | Healthcheck + Sentry |
| H-1 | Monitor canary; rollback siaga | SRE | Error rate <0.5% |
| H-0 | Promote canary 100% | SRE | All replicas serving |
| H+0 | Update status page → "operational" | IC | Public OK |
| H+5m | Publish blog post + tweet | Marketing | URL live |
| H+15m | Post Product Hunt + Hacker News + Indie Hackers | Marketing | Submission live |
| H+30m | Email blast warm list | Marketing | Resend delivery >95% |
| H+1h | Sync standup launch room | All | Issues triaged |
| H+2h | Webinar / live Q&A (opsional) | DevRel | Recording uploaded |
| H+4h | Daily review dashboard | PM | Funnel & error |
| H+12h | First retrospective notes | PM | Notion entry |

### 95.6 T+1..30 Hari — Post-Launch Operations

- Daily standup khusus "Launch Watch" 30 menit.
- Daily metrics review: signup, activation, error rate, AI cost burn,
  Stripe receipts.
- Hotfix lane: dedicated branch `hotfix/*` direct merge dengan 1
  reviewer.
- Update changelog setiap rilis (publik di `/changelog`).
- Customer success outreach 1:1 untuk top 20 paying orgs.
- Press follow-ups + podcast interview booking.

### 95.7 First Quarter Review (T+90 Hari)

- Health metrics vs target (Sec. 7).
- Postmortem tiap incident Sev1/Sev2 dipublik.
- Update risk register (Sec. 74).
- Cost review: AI burn, infra cost, gross margin per plan.
- Roadmap re-prioritization untuk fase berikutnya.

## 96. Day-2 Operations (Setelah Final / Steady State)

### 96.1 Cadence Release

- **Train release** Selasa & Kamis 10:00 WIB.
- Hotfix kapan saja jika Sev1/Sev2 active.
- Versi semver (`major.minor.patch`); breaking change membutuhkan major
  bump + 60 hari deprecation notice.
- Changelog dipublikasi otomatis dari Conventional Commits.

### 96.2 Cadence Security & Compliance

- Mingguan: dependency scan (Snyk/Dependabot), prompt eval suite.
- Bulanan: access review, secret rotation due-list, penetration smoke.
- Kuartalan: GameDay, BCP/DR drill, SOC2 control attestation refresh.
- Tahunan: pentest eksternal, ISO surveillance audit, SOC2 Type 2.

### 96.3 Cadence Finance / FinOps

- Mingguan: cost dashboard review (AI, infra, payout outflows).
- Bulanan: invoice / receivables, plan ratio, churn cohort.
- Kuartalan: pricing optimization (Sec. 60.6) berdasarkan A/B + cohort
  data.

### 96.4 Cadence Growth

- Mingguan: funnel & activation; A/B winners promote, losers kill.
- Bulanan: SEO ranking, content calendar, partnership pipeline.
- Kuartalan: NPS survey, customer advisory board, roadmap voting.

### 96.5 Continuous Improvement Loop

- Setiap incident → postmortem ≤5 hari kerja → action items dengan owner
  + due date di Linear (visibility ke seluruh tim).
- Setiap quarter "Engineering Excellence Week": dedicated time refactor,
  bayar tech debt, upgrade dependency major.
- "What we learned" newsletter monthly internal.

## 97. Common Pitfalls Per Phase + Mitigation

| Phase | Pitfall | Mitigasi cepat |
| --- | --- | --- |
| 0 | Build time CI >10 menit | Cache pnpm + Turborepo remote cache; split job |
| 0 | Drizzle migration drift | Wajib `drizzle-kit check` di CI; PR template ada DB checklist |
| 0 | Better Auth session tidak terbawa lintas subdomain | Set `cookieDomain: '.teskel.app'`, `secureCookies: true` |
| 1 | Workflow run hang | Tambah timeout absolut 60s (workflow level) + 10s (step) |
| 1 | LLM cost spike tak terkontrol | Per-org daily cap di gateway + alert >70% |
| 1 | KB ingest blocking event loop | Pindah ke worker khusus (`ingest` queue) |
| 2 | Puck data schema berubah breaking | Versi konfigurasi blok; migration up/down |
| 2 | Sandbox egress accidental | Default-deny + allowlist domain per template |
| 2 | Preview env naik biaya | Auto-suspend setelah 24 jam tidak diakses |
| 3 | Stripe webhook duplikat | Idempotency key: `event.id`; simpan `processed_events` |
| 3 | Listing review bottleneck | Auto-reject yang gagal validasi schema; tugaskan reviewer rotational |
| 3 | Refund + clawback creator wallet | Lock-up minimal 7 hari sebelum payout |
| 4 | SAML config customer salah → user terkunci | Test endpoint `/sso/test` + fallback magic-link admin |
| 4 | Audit log tabel >500GB | Partition + cold storage S3 90+ hari |
| 5 | NL → app halusinasi entitas | Plan harus dikonfirmasi user sebelum `Generate` |
| 5 | Vertical pack maintenance lupa | Owner per pack + quarterly review |
| 6 | ISO 27001 dokumen kadaluarsa | Tooling Vanta/Drata + DRI per control |
| Final | Status page lupa update | Otomatisasi via Better Stack alerts → Statuspage API |
| Final | Email blast spam-flagged | Warm-up domain + IP, batch <50k/jam, list hygiene |

## 98. Local Dev → Prod Promotion Checklist (Per Story)

Sebelum merge PR ke `main`:

- [ ] Lint + typecheck + test hijau di CI.
- [ ] Migrasi DB reversible (atau backfill plan tertulis).
- [ ] Feature flag added (default OFF) jika user-visible besar.
- [ ] Telemetry: event ditambah, dashboard updated.
- [ ] Docs: README/CHANGELOG/User docs diupdate.
- [ ] PR description: rollback plan + screenshots/recordings.
- [ ] Reviewer ≥1, owner code domain ack-ed.

Setelah merge:

- [ ] Auto-deploy ke staging hijau (synthetic test pass).
- [ ] Soft launch ke 10% via flag (jika applicable).
- [ ] Monitor 24 jam → ramp 100%.
- [ ] Hapus flag dalam 30 hari.

Sebelum cutover/release besar (Sec. 95):

- [ ] Lihat checklist di Sec. 85 + 95.

## 99. Senior-Eng Decision Cheatsheet (Quick Reference)

Daftar keputusan paling sering ditanya, supaya tim baru tidak debat:

| Pertanyaan | Jawaban Default | Catatan |
| --- | --- | --- |
| Pakai Server Component atau Client? | Server, kecuali butuh state interaktif | Bungkus Client di leaf |
| Fetch data di server atau pakai TanStack Query? | Initial render: Server. Real-time/refetch: TQ | Hindari double-fetch |
| Tulis raw SQL atau Drizzle query? | Drizzle. Raw SQL hanya untuk query analitik kompleks | Selalu pakai parameter binding |
| Kapan pakai Inngest, kapan BullMQ? | Cron / scheduled / step.ai → Inngest. Real-time short job, fan-out → BullMQ | Lihat Sec. 28 |
| Better Auth plugin custom atau pakai bawaan? | Bawaan dulu, custom hanya jika gap nyata | ADR wajib jika custom |
| Tulis langsung di DB atau outbox? | Outbox kalau ada side-effect pengguna luar (webhook/email) | Lihat 19.7 |
| Pakai Liveblocks atau Yjs raw? | Liveblocks (managed presence + storage) | Yjs hanya untuk dokumen besar atau self-host |
| Sandbox di E2B, Daytona, atau Modal? | E2B default. Daytona kalau butuh dev-env panjang. Modal kalau perlu GPU | Lihat 32 |
| Pakai pgvector atau Qdrant? | pgvector default. Qdrant per Enterprise opt-in | >50M chunk → Qdrant |
| Tulis kustom komponen UI atau pakai shadcn/ui? | shadcn/ui default; ekstend lewat composition | Hindari fork yang divergent |
| Tulis migration manual atau drizzle-kit generate? | `generate` selalu. Manual hanya untuk index/constraints khusus | Test up + down |
| Cache di Redis atau React Query saja? | TanStack di client; Redis untuk shared cross-instance | Hindari double cache invalidation |
| Tambah deps baru — kapan boleh? | Hanya kalau menggantikan ≥50 baris kode kita & lisensi MIT/Apache | ADR kalau >100KB bundle |
| Self-host atau pakai cloud provider service? | Cloud sampai $X/bulan, lalu evaluasi self-host | Hindari operasional 5 vendor sekaligus |
| Buat ADR atau RFC? | ADR untuk keputusan dengan konsekuensi tahunan; RFC untuk eksplorasi | Lihat Appendix G |

---

## Changelog v1 → v2 → v2.1 → v2.2 → v2.3 (FINAL)

### v2.3 (Final — Skills Library Expansion)

> Status: **FINAL**. Plan ini tetap *executable single source of
> truth*. Perubahan terhadap keputusan plan tetap wajib lewat jalur
> ADR/RFC (Sec. 71 + Appendix G).

- **Skills library di [`.agents/skills/`](./.agents/skills/) diperluas
  dari 5 → 19** untuk menutup operasi berulang lintas seluruh fase
  build:
  - **UI / UX**:
    [`add-ui-component`](./.agents/skills/add-ui-component/SKILL.md)
    (shadcn/ui-style component, design tokens, variants, states, a11y,
    Storybook, tests),
    [`add-page`](./.agents/skills/add-page/SKILL.md) (Next.js 15 App
    Router page, RSC default, RLS, four state files, Playwright smoke,
    perf budget),
    [`add-block`](./.agents/skills/add-block/SKILL.md) (Puck visual
    builder block, schema + render + palette + migrator + tests),
    [`design-review`](./.agents/skills/design-review/SKILL.md)
    (checklist 13 sumbu yang wajib dijalankan sebelum PR UI dibuka).
  - **Backend & API**:
    [`add-api-route`](./.agents/skills/add-api-route/SKILL.md) (Hono +
    Zod + auth + RBAC + idempotency + RLS + OpenAPI + SDK regen +
    integrationtest + observability).
  - **AI / LLM**:
    [`add-prompt-slot`](./.agents/skills/add-prompt-slot/SKILL.md)
    (versioned slot di registry, schema input/output, model
    routing/fallback, budget, Promptfoo eval ≥10 case, Langfuse trace,
    versioning rules).
  - **Operations & release**:
    [`add-feature-flag`](./.agents/skills/add-feature-flag/SKILL.md)
    (PostHog flag bertyped wrapper, rollout plan + kill criteria +
    removal SLA),
    [`add-runbook`](./.agents/skills/add-runbook/SKILL.md) (template
    runbook untuk setiap alert).
  - **Security & access**:
    [`add-rbac-role`](./.agents/skills/add-rbac-role/SKILL.md)
    (permission registry → role map → Better Auth → RLS → middleware
    → UI gate → audit log),
    [`rotate-secret`](./.agents/skills/rotate-secret/SKILL.md) (zero-
    downtime rotation 15 langkah dengan dual-window per tipe secret).
  - **Marketplace**:
    [`publish-template`](./.agents/skills/publish-template/SKILL.md)
    (manifest, license, pricing, demo URL, moderation, post-publish
    monitoring, versioning, takedown).
  - **Quality**:
    [`add-e2e-test`](./.agents/skills/add-e2e-test/SKILL.md)
    (Playwright fixtures, tenant isolation, network policy, a11y
    assertion, sharding, flake budget).
  - **Data**:
    [`data-backfill-job`](./.agents/skills/data-backfill-job/SKILL.md)
    (BullMQ-driven, batched, idempotent, throttled, RLS-aware,
    resumable backfill).
  - **Process**:
    [`kickoff-phase`](./.agents/skills/kickoff-phase/SKILL.md)
    (formal kickoff: update `current-phase.md`, brief, derive stories,
    register exit criteria, set up tracking, announce).
- **Update [`AGENTS.md` §19 (Skills Library)](./AGENTS.md#19-skills-library)**
  jadi indeks lengkap berbasis kategori plus tabel cepat
  "I'm about to..." → skill sehingga agent langsung tahu skill mana
  yang harus dijalankan.
- **Update [`README.md`](./README.md)** untuk menyebutkan ekspansi
  skills + link ke indeks.
- Tidak ada perubahan keputusan plan; semua skill mencerminkan
  konvensi yang sudah ada di plan dan di AGENTS.md.

### v2.2 (Final — AI Agent Operating Manual + Repo Scaffolding)

> Status: **FINAL**. Plan ini dianggap *executable single source of
> truth*. Perubahan selanjutnya melalui jalur ADR/RFC (lihat Sec. 71 +
> Appendix G).

- Tambah berkas operasional di repo root yang melengkapi plan ini supaya
  AI coding agent (Devin, Cursor, Claude Code, Copilot, Aider) bisa
  langsung eksekusi tanpa kebingungan:
  - [`AGENTS.md`](./AGENTS.md) — operating manual (21 bab): ringkasan
    cepat, identitas project, hierarki dokumen, repo map target, stack
    yang sudah diputuskan, command lokal, konvensi koding, hard
    constraint arsitektur, aturan keamanan/data, aturan AI/LLM, mirror
    Sec. 99, phase awareness, PR workflow, DoR/DoD, RFC→ADR flow,
    forbidden actions, escalation triggers, indeks lokasi, library
    skill, catatan per-agent, versioning.
  - [`CLAUDE.md`](./CLAUDE.md), [`.cursorrules`](./.cursorrules),
    [`.github/copilot-instructions.md`](./.github/copilot-instructions.md)
    — pointer file per agent, redirect ke `AGENTS.md`.
  - [`README.md`](./README.md) — public-facing intro (status,
    roadmap, stack at a glance, quick commands).
  - [`.agents/state/current-phase.md`](./.agents/state/current-phase.md)
    — state machine‑readable untuk membatasi scope agent ke fase aktif
    (default: Phase 0).
  - [`.agents/state/owners.md`](./.agents/state/owners.md) +
    [`.github/CODEOWNERS`](./.github/CODEOWNERS) — sumber tunggal
    routing review.
  - 5 SKILL.md di [`.agents/skills/`](./.agents/skills/):
    `add-package`, `add-table` (RLS-aware), `add-workflow-node`
    (full-stack), `release-canary`, `incident-sev1` — checklist agent
    untuk operasi berulang.
  - [`docs/adr/0000-template.md`](./docs/adr/0000-template.md) +
    [`docs/rfc/0000-template.md`](./docs/rfc/0000-template.md) —
    template formal untuk perubahan keputusan.
  - [`.github/pull_request_template.md`](./.github/pull_request_template.md)
    — template PR yang menegakkan DoD + architecture compliance check.
- Tegaskan jalur amandemen: untuk mengubah keputusan yang
  terdokumentasi di plan ini, agent/engineer **wajib** menulis RFC
  → promote ke ADR → update plan dalam PR yang sama. Plan ini
  *immutable kecuali via ADR* (lihat AGENTS.md §15).

### v2.1 (Phase X — Senior Fullstack Build Walkthrough)

- Tambah **Part X (Sec. 87–99)**: panduan eksekusi minggu-per-minggu dari
  Day-0 (akun & tooling), Phase 0 (foundation) sampai Phase 6 (enterprise
  & compliance plus), final cutover/public launch playbook (T-30 → T+90),
  day-2 ops (release/security/finance/growth cadence), pitfalls per fase,
  promotion checklist per-story, dan senior-eng decision cheatsheet.
- Tambah perintah konkret bash/SQL/TS untuk scaffold monorepo, Drizzle,
  Better Auth, BullMQ, E2B, Stripe Connect, sehingga PR pertama tiap fase
  punya starting point yang jelas.
- Tambah tabel akun & layanan eksternal yang harus disiapkan sebelum
  coding, lengkap dengan plan awal dan catatan provisioning.
- Tabel keputusan cepat (Sec. 99) supaya tim baru tidak debat ulang.

### v2 (Initial restructure)

- Restrukturisasi total ke 9 bagian (Strategy, Design, Architecture,
  Operations, Security, Growth, Build Plan, Risks, Reference).
- Tambah konvensi status (`DECIDED`/`DEFAULT`/`OPEN`/`LATER`) dan priority
  (`P0..P3`).
- Pilih *opinionated default* untuk semua *fork in the road* (auth=Better
  Auth, ORM=Drizzle dengan jalur Prisma, queue=BullMQ + Inngest jalur ke-2,
  sandbox=E2B, observability AI=Langfuse, dll.) — masing-masing diiringi
  ADR yang harus ditulis.
- Tambah modul yang sebelumnya hilang/lemah: SLO/SLA, runbooks, DR/backup,
  FinOps, threat model, RBAC matrix, RLS, marketplace moderation, template
  manifest spec, real-time/collab, sandbox layer, SDK + plugin + CLI,
  prompt registry & eval, AI guardrails, A/B + feature flags, customer
  support tooling.
- Tambah deliverables, DoR/DoD, quality gates per fase.
- Tambah appendix konkret (DDL, Hono RPC sample, template manifest,
  workflow JSON, env vars, launch checklist, ADR template).
- Roadmap diperluas dari 6 fase ke 7 (tambah Phase 6 — Enterprise &
  Compliance) dengan timeline ulang.
