# TESKEL — Fullstack Build Breakdown From Zero

**AI Startup Factory Platform**

Clean • Modern • Global-Class • AI-Native

---

## 0. Executive Summary

TESKEL adalah **AI Startup Factory + Infrastructure Platform** untuk membantu user membangun, meluncurkan, memonetisasi, dan menskalakan produk AI-native dengan cepat.

TESKEL bukan sekadar website builder, AI wrapper, atau marketplace biasa. TESKEL diposisikan sebagai **operating system untuk membangun AI-native businesses**:

- **Build**: AI SaaS, workflow, agent, dan aplikasi.
- **Deploy**: publish aplikasi, workflow, atau agent ke production.
- **Monetize**: subscription, usage-based billing, template sales, revenue sharing.
- **Scale**: analytics, infra, observability, marketplace, plugin ecosystem.

Target utama awal adalah membuat user bisa meluncurkan AI business pertama dalam waktu **kurang dari 10 menit** dari template siap pakai.

---

## 1. Product Direction Final

### 1.1 Core Positioning

TESKEL = **AI Startup Factory + Infrastructure Platform**.

User bisa:

- Build AI SaaS.
- Build AI workflows.
- Build AI agents.
- Deploy apps.
- Monetize.
- Scale.

Creator bisa:

- Jual template.
- Jual workflow.
- Jual AI systems.
- Jual plugins.
- Jual themes.
- Mendapat revenue share.

### 1.2 One-Line Positioning

**TESKEL helps founders, creators, and developers launch AI-native businesses from templates, workflows, agents, and deployable infrastructure.**

### 1.3 Product Category

TESKEL berada di perpotongan beberapa kategori:

| Category | TESKEL Angle |
|---|---|
| AI App Builder | Membuat produk AI siap pakai |
| Workflow Automation | Visual automation dan agentic workflow |
| SaaS Boilerplate Platform | Auth, billing, dashboard, deployment |
| Creator Marketplace | Template, agent, workflow, plugin economy |
| Deployment Platform | Publish app/workflow/agent ke production |
| AI Infrastructure Layer | Model routing, memory, RAG, observability |

### 1.4 ICP Initial

Prioritas awal:

1. **Solo founders** yang ingin launch AI SaaS cepat.
2. **AI builders** yang sering membuat tool dan automation.
3. **Agencies** yang membangun AI solution untuk client.
4. **Creators** yang ingin menjual template dan workflow.
5. **Developers** yang ingin starting point production-ready.

### 1.5 Core Promise

**From idea to deployed AI business in minutes, not months.**

---

## 2. Design Philosophy

TESKEL harus terasa:

| Trait | Style |
|---|---|
| Modern | Minimal, sharp, current |
| Fast | Low friction, instant feedback |
| Premium | Polished, thoughtful, subtle |
| Technical | Developer-native, clear data model |
| Clean | Uncluttered, readable |
| Global | SaaS-grade, not local-only aesthetic |
| AI-native | Futuristic but simple |

### 2.1 Design Keywords

- Calm interface.
- Precise layout.
- High whitespace.
- Small but useful animation.
- Clear hierarchy.
- No decorative noise.
- Strong empty states.
- Keyboard-first interactions.
- Command-driven UX.

### 2.2 Reference Products

Style reference:

- **Vercel**: clean developer platform, deployment-first UX.
- **Linear**: keyboard-first, fast interactions, polished details.
- **Framer**: visual builder quality and motion.
- **Supabase**: developer-native dashboard and database UX.
- **Notion**: modular blocks and simple creation flow.
- **Raycast**: command palette and action-first navigation.
- **Retool**: app/workflow builder patterns.
- **n8n**: workflow automation mental model.

---

## 3. UI/UX Direction

### 3.1 UI Principles

#### A. Clean

- Banyak whitespace.
- Simple hierarchy.
- Minimal noise.
- Fokus pada task utama.
- Gunakan border tipis, bukan shadow berat.

#### B. Fast

- Instant interactions.
- Optimistic UI.
- Smooth transitions.
- Skeleton loading.
- Keyboard shortcuts.
- Command palette untuk hampir semua action.

#### C. AI-Native

- AI assistant panels.
- Command palette.
- Workflow graph.
- Prompt editor.
- Model selector.
- Run history.
- AI usage visibility.

#### D. Modular

- Semua reusable.
- Feature-based components.
- UI primitives konsisten.
- Design tokens sejak awal.

### 3.2 Design System

#### Fonts

Recommended:

- **Geist** untuk default SaaS/developer feel.
- **Inter** sebagai alternatif paling aman.

Recommendation:

- Gunakan **Geist Sans** untuk UI.
- Gunakan **Geist Mono** untuk code, logs, workflow payload, API keys.

#### Radius

Modern radius:

- `rounded-lg` untuk input kecil.
- `rounded-xl` untuk cards.
- `rounded-2xl` untuk panels dan modal besar.

#### Shadows

Gunakan soft subtle shadows:

- Default: border-only.
- Hover: subtle shadow.
- Floating panel: medium soft shadow.

#### Color Palette

Light mode priority.

| Area | Color |
|---|---|
| Background | white |
| Surface | zinc-50 |
| Elevated Surface | white |
| Border | zinc-200 |
| Muted Border | zinc-100 |
| Text Primary | zinc-950 |
| Text Secondary | zinc-600 |
| Text Muted | zinc-400 |
| Accent | blue/violet |
| Success | emerald |
| Warning | amber |
| Danger | red |

#### Dark Mode

Dark mode penting, tetapi bukan prioritas visual awal. Buat design token siap dark mode sejak awal agar tidak perlu refactor besar.

### 3.3 Core UX Flows

#### FLOW 1 — Create AI Startup

Steps:

1. Create project.
2. Choose template.
3. Configure AI.
4. Customize UI.
5. Connect billing.
6. Deploy.

Target:

- **Kurang dari 10 menit**.
- Wizard maksimal 5-6 step.
- Ada preview langsung.
- Ada default config yang aman.

#### FLOW 2 — Creator Marketplace

Steps:

1. Build template.
2. Package template.
3. Publish listing.
4. Add pricing.
5. Get installs.
6. Earn revenue.

Important UX:

- Creator dashboard jelas.
- Revenue analytics.
- Install funnel.
- Review dan rating.
- Versioning template.

#### FLOW 3 — AI Workflow Builder

Steps:

1. Drag nodes.
2. Connect logic.
3. Configure model/API/data.
4. Test run.
5. Inspect logs.
6. Deploy automation.

Important UX:

- Node config panel di kanan.
- Run/debug panel di bawah.
- Logs per node.
- Error state jelas.
- Version history.

---

## 4. Recommended System Architecture

### 4.1 Monorepo Recommendation

Gunakan monorepo sejak awal agar frontend, backend, package UI, config, dan shared types mudah dikelola.

Recommended structure:

```txt
apps/
  web/
  api/
  worker/
packages/
  ui/
  config/
  db/
  shared/
  ai/
  workflow-engine/
  template-engine/
  eslint-config/
  typescript-config/
```

### 4.2 Why Monorepo

- Shared types antara web, api, worker.
- Reusable UI components.
- Centralized validation schemas.
- Easier refactor.
- Cocok untuk AI-assisted coding karena konteks lebih rapi.
- Cocok dengan Turborepo atau pnpm workspaces.

### 4.3 Build Tooling

Recommended:

| Area | Tool |
|---|---|
| Package Manager | pnpm |
| Monorepo | Turborepo |
| Lint | ESLint |
| Format | Prettier |
| Git Hooks | Lefthook or Husky |
| Type Checking | TypeScript strict |
| Testing | Vitest, Playwright |
| Runtime scripts | Turbo pipelines |

---

## 5. Frontend Architecture

### 5.1 Recommended Stack

| Area | Technology |
|---|---|
| Framework | Next.js 15 |
| Language | TypeScript |
| UI Runtime | React 19 |
| Styling | Tailwind CSS |
| Components | shadcn/ui |
| Animations | Framer Motion or Motion |
| Icons | Lucide |
| Forms | React Hook Form |
| Validation | Zod |
| Client State | Zustand |
| Server State | TanStack Query |
| Tables | TanStack Table |
| Command Menu | cmdk |
| Charts | Recharts or Tremor-style custom charts |

### 5.2 Why Next.js

Next.js cocok karena:

- App Router.
- SSR dan streaming.
- Server Components.
- Edge support.
- Strong React ecosystem.
- AI tooling ecosystem besar.
- Mudah untuk dashboard, marketing pages, auth, dan marketplace.

### 5.3 Frontend Folder Structure

```txt
apps/web/
  src/
    app/
      (marketing)/
      (auth)/
      (dashboard)/
      api/
    components/
      ui/
      layout/
      shared/
    features/
      auth/
      dashboard/
      projects/
      templates/
      workflows/
      builder/
      ai/
      billing/
      marketplace/
      deployments/
    hooks/
    store/
    lib/
    styles/
```

### 5.4 Feature-Based Structure

Jangan struktur hanya berdasarkan tipe file. Gunakan feature-based structure.

Example:

```txt
features/workflows/
  components/
  hooks/
  api/
  stores/
  schemas/
  types/
  utils/
```

Benefit:

- Mudah scale.
- Mudah dipahami AI coding assistant.
- Dependency lebih jelas.
- Feature bisa dipindah atau diisolasi.

---

## 6. Component System

### 6.1 Core UI Components

| Component | Purpose |
|---|---|
| Sidebar | Navigation utama |
| Topbar | Search, commands, account |
| Dashboard Cards | Analytics dan summary |
| AI Chat | AI interaction |
| Workflow Canvas | Automation builder |
| Template Cards | Marketplace display |
| Command Palette | Fast actions |
| Empty State | Guidance untuk user baru |
| Data Table | Template, deployment, logs |
| Activity Feed | Audit dan events |
| Settings Panel | Config per module |

### 6.2 Builder Components

| Component | Purpose |
|---|---|
| AI Node | LLM execution |
| Logic Node | Conditions |
| API Node | Integrations |
| Memory Node | AI state |
| Form Builder | Inputs |
| Data Source Node | DB or external data |
| Schedule Node | Cron trigger |
| Email Node | Notification |
| Webhook Node | External trigger |

### 6.3 Design System Packages

Recommended package split:

```txt
packages/ui/
  components/
  primitives/
  blocks/
  icons/
  hooks/
  tokens/
```

Use shadcn/ui as source baseline, but keep ownership of components inside repo.

---

## 7. Visual Builder Engine

### 7.1 Primary Recommendation

Best initial candidate:

- **Craft.js** for custom React page/app builder.

Alternative references:

- **Puck** for React visual editing with more structured config.
- **Builder.io** for commercial-grade visual CMS/editor concepts.

### 7.2 Why Craft.js

Craft.js cocok karena:

- React-native mental model.
- Component-based.
- Extensible.
- Cocok untuk SaaS builder.
- Dapat menyimpan tree sebagai JSON.
- Bisa dikontrol penuh oleh internal product logic.

Important caveat:

- Craft.js bukan ready-made editor penuh. TESKEL tetap harus membangun UI editor sendiri: layers panel, properties panel, inspector, responsive preview, theme editor.

### 7.3 Builder Storage Model

Basic representation:

```json
{
  "type": "Card",
  "props": {},
  "children": []
}
```

Recommended production model:

```json
{
  "version": "1.0.0",
  "root": "node_root",
  "nodes": {
    "node_root": {
      "type": "Page",
      "props": {
        "title": "AI CRM"
      },
      "children": ["node_1"]
    },
    "node_1": {
      "type": "HeroSection",
      "props": {
        "headline": "Launch your AI CRM"
      },
      "children": []
    }
  },
  "theme": {
    "font": "Geist",
    "radius": "xl",
    "accent": "violet"
  }
}
```

### 7.4 Builder Features

MVP:

- Drag and drop components.
- Properties panel.
- Layers panel.
- Preview mode.
- Save draft.
- Publish version.

Post-MVP:

- Resize.
- Responsive preview.
- Theme editor.
- Version history.
- Component marketplace.
- AI edit commands.
- Export to code.

---

## 8. Workflow Builder

### 8.1 Core Technology

Best initial candidate:

- **React Flow / xyflow**.

Why:

- Mature node-based UI library.
- Banyak contoh workflow builder.
- Custom nodes dan edges kuat.
- Cocok untuk automation, agent, data pipeline, dan visual programming.

### 8.2 Workflow Nodes

| Node | Purpose |
|---|---|
| Trigger | Manual, webhook, schedule |
| AI | LLM call |
| Memory | Read/write context |
| API | Fetch external services |
| Logic | If/else, switch |
| DB | Queries |
| Email | Notifications |
| Browser | Automation |
| Schedule | Cron |
| Transform | Map/filter/format data |
| Human Approval | Pause for approval |
| Output | Return final result |

### 8.3 Workflow Storage

Store as JSON graph:

```json
{
  "version": "1.0.0",
  "nodes": [
    {
      "id": "node_1",
      "type": "trigger",
      "position": { "x": 0, "y": 0 },
      "data": {}
    }
  ],
  "edges": [
    {
      "id": "edge_1",
      "source": "node_1",
      "target": "node_2"
    }
  ]
}
```

### 8.4 Workflow Runtime

Runtime harus terpisah dari UI.

Core responsibilities:

- Validate graph.
- Resolve execution order.
- Execute node handlers.
- Persist run state.
- Retry failed jobs.
- Emit logs and events.
- Track usage and billing.

### 8.5 Execution Strategy

MVP:

- Directed acyclic graph first.
- Manual trigger and webhook trigger.
- Sequential execution.
- Basic branching.
- Node-level logs.

Later:

- Parallel branches.
- Loops.
- Human-in-the-loop.
- Long-running agent state.
- Sandbox execution.
- Versioned workflow deployments.

---

## 9. Backend Architecture

### 9.1 Recommended Stack

| Area | Technology |
|---|---|
| Runtime | Node.js initially, Bun optional later |
| Language | TypeScript |
| Framework | Hono |
| Validation | Zod |
| ORM | Prisma |
| API Style | REST initially |
| Database | PostgreSQL |
| Queue | BullMQ |
| Cache | Redis |
| Auth | Better Auth or Clerk |

### 9.2 Why Hono

Hono cocok karena:

- Modern.
- Fast.
- Lightweight.
- Edge-ready.
- TypeScript-friendly.
- API ergonomics sederhana.

### 9.3 Backend Structure

```txt
apps/api/
  src/
    modules/
      auth/
      ai/
      workflows/
      templates/
      billing/
      deployments/
      analytics/
      marketplace/
    services/
    lib/
    middleware/
    routes/
    config/
```

### 9.4 Domain Modules

| Module | Function |
|---|---|
| Auth | Login, sessions, org membership |
| AI | LLM orchestration, model routing |
| Workflow | Workflow graph and runtime API |
| Templates | Template management |
| Marketplace | Listing, install, rating |
| Billing | Stripe subscription and metering |
| Deploy | App deployment pipeline |
| Analytics | Events and product metrics |
| Usage | Quotas and rate tracking |
| Secrets | Encrypted user credentials |

### 9.5 API Style

Start dengan REST karena:

- Simpler.
- Faster to ship.
- Easy to debug.
- Cocok untuk public API awal.

Potential later:

- tRPC untuk internal type-safe APIs.
- GraphQL jika marketplace atau integrations sangat kompleks.
- Public REST API untuk developer ecosystem.

---

## 10. Database Architecture

### 10.1 Main Database

Recommended:

- **PostgreSQL**.

Why:

- Relational strong.
- Scalable.
- Mature.
- Supports JSONB.
- Supports pgvector.
- Strong ecosystem.

### 10.2 ORM

Recommended:

- **Prisma**.

Why:

- TypeScript ecosystem kuat.
- Migration workflow jelas.
- AI-assisted coding friendly.
- Banyak contoh SaaS production.

### 10.3 Core Tables

Core:

- `users`
- `organizations`
- `organization_members`
- `projects`
- `workflows`
- `workflow_versions`
- `workflow_runs`
- `workflow_run_logs`
- `templates`
- `template_versions`
- `template_installs`
- `deployments`
- `subscriptions`
- `usage_logs`
- `ai_requests`
- `api_keys`
- `secrets`
- `audit_logs`

Marketplace:

- `marketplace_listings`
- `reviews`
- `ratings`
- `creator_profiles`
- `creator_payouts`
- `licenses`

AI/RAG:

- `knowledge_bases`
- `documents`
- `document_chunks`
- `embeddings`
- `agent_memories`
- `prompt_templates`

### 10.4 Vector Search

Use:

- **pgvector**.

Untuk:

- Embeddings.
- RAG.
- Semantic search.
- Template discovery.
- Workflow recommendation.
- AI memory retrieval.

### 10.5 Multi-Tenancy

Recommended awal:

- Organization-based multi-tenancy.
- Semua core table punya `organization_id` jika relevan.
- Enforce authorization di service layer.
- Later bisa tambah PostgreSQL Row Level Security jika dibutuhkan.

---

## 11. AI Infrastructure

### 11.1 Multi-Model Layer

Recommended awal:

- **OpenRouter** sebagai model gateway.

Supported model families:

- GPT.
- Claude.
- Gemini.
- DeepSeek.
- Groq-hosted models.
- Open-source models.

### 11.2 Why OpenRouter

- Banyak model lewat satu API.
- Cocok untuk model switching.
- Cepat untuk MVP.
- Mengurangi integrasi provider satu per satu.

Caveat:

- Untuk enterprise atau scale besar, TESKEL perlu abstraction agar tidak lock-in.

### 11.3 AI Systems

| System | Purpose |
|---|---|
| Prompt Engine | Manage prompts and variables |
| Memory System | Store and retrieve context |
| Agent Runtime | Execute tools and agent steps |
| RAG Engine | Retrieval from documents |
| AI Router | Model switching and fallback |
| Usage Meter | Token/cost tracking |
| Safety Layer | Guardrails and policy checks |
| Eval Layer | Test prompt/workflow quality |

### 11.4 AI Orchestration Strategy

Awal:

- Lightweight custom orchestration.
- Jangan terlalu tergantung LangChain.
- Buat abstraction sederhana: `AIProvider`, `ModelRouter`, `ToolRegistry`, `AgentRun`.

Later:

- Tambah support LangGraph atau Mastra jika workflow agent makin kompleks.
- Tambah eval harness.
- Tambah prompt/version testing.

### 11.5 AI-Native UX Features

- AI assistant per project.
- AI command palette.
- Generate app from prompt.
- Generate workflow from prompt.
- Explain workflow.
- Debug failed workflow run.
- Optimize prompt.
- Suggest monetization strategy.
- Recommend template.

---

## 12. Queue & Runtime

### 12.1 Core Stack

| Area | Technology |
|---|---|
| Queue | BullMQ |
| Cache | Redis |
| Jobs | BullMQ Workers |
| Scheduling | BullMQ repeatable jobs initially |
| Event Bus | Internal domain events initially |

### 12.2 Jobs

- AI generation.
- Embedding indexing.
- Deployment builds.
- Emails.
- Workflow execution.
- Webhook delivery.
- Usage aggregation.
- Marketplace payout calculation.

### 12.3 Worker Structure

```txt
apps/worker/
  src/
    jobs/
      ai-generation.ts
      workflow-run.ts
      deployment-build.ts
      email-send.ts
      embedding-index.ts
    queues/
    lib/
```

### 12.4 Runtime Rule

Jangan jalankan long-running execution di request/response API. Semua task berat harus masuk queue.

---

## 13. Deployment Infrastructure

### 13.1 Initial Stack

| Area | Technology |
|---|---|
| Containers | Docker |
| Platform | Coolify |
| Reverse Proxy | Traefik |
| SSL | Let's Encrypt via platform |
| Registry | GitHub Container Registry or Docker Hub |

### 13.2 Why Coolify

- Cepat setup.
- Self-hosted.
- Startup friendly.
- Docker-native.
- Biaya awal lebih rendah.
- Cocok untuk early product iteration.

### 13.3 Deployment Flow

1. Generate app.
2. Build Docker image.
3. Push container.
4. Provision domain.
5. Enable SSL.
6. Run health check.
7. Mark deployment active.

### 13.4 Long-Term Infra

Ketika scale:

- Kubernetes.
- Multi-region infra.
- Edge deployments.
- Dedicated worker pools.
- Isolated customer environments.

Important:

- Jangan mulai dengan Kubernetes.
- Fokus product-market fit dan launch velocity.

---

## 14. Storage Layer

### 14.1 Recommended

- **Cloudflare R2**.

Use cases:

- Uploads.
- Datasets.
- Generated assets.
- Template assets.
- Build artifacts.
- Export files.
- CDN delivery.

### 14.2 Why Cloudflare R2

- S3-compatible API.
- Cost-effective.
- Good global CDN story.
- Startup-friendly.

### 14.3 File Categories

- User uploads.
- Project assets.
- Template screenshots.
- Generated app bundles.
- Knowledge base documents.
- Workflow attachments.
- Exported logs.

---

## 15. Authentication & Authorization

### 15.1 Recommended Options

| Option | Best Use |
|---|---|
| Better Auth | Modern, flexible, self-owned auth |
| Clerk | Fastest production auth setup |
| Auth.js | Customizable, established ecosystem |

### 15.2 Recommendation

For fastest SaaS MVP:

- Use **Clerk** if speed and reliability are more important than full ownership.

For long-term platform ownership:

- Use **Better Auth** if TESKEL wants self-hosted auth and deeper control.

### 15.3 Required Features

- OAuth.
- Magic links.
- Organizations.
- RBAC.
- Team permissions.
- API keys.
- Session management.
- Audit logs.

### 15.4 RBAC Roles

Initial roles:

- Owner.
- Admin.
- Developer.
- Billing Manager.
- Viewer.
- Creator.

---

## 16. Billing Infrastructure

### 16.1 Recommended

- **Stripe**.

### 16.2 Features

- Subscriptions.
- Metered billing.
- Invoices.
- Creator payouts.
- Revenue sharing.
- Coupons.
- Trials.
- Usage-based AI billing.

### 16.3 Billing Model

Potential pricing dimensions:

- Seats.
- Projects.
- Deployments.
- AI tokens.
- Workflow runs.
- Storage.
- Premium templates.
- Marketplace commission.

### 16.4 Marketplace Revenue

Example:

- Creator gets 70-85%.
- TESKEL gets 15-30%.
- Optional platform subscription for creators.

---

## 17. Analytics Stack

### 17.1 Product Analytics

Recommended:

- **PostHog**.

### 17.2 Metrics

Track:

- Activation.
- Retention.
- Deployments.
- AI usage.
- Template installs.
- Workflow runs.
- Revenue.
- Churn.
- Funnel completion.

### 17.3 Critical Funnels

Create AI startup funnel:

1. Signup.
2. Create project.
3. Choose template.
4. Configure AI.
5. Deploy.
6. Connect billing.
7. First customer/payment.

Creator funnel:

1. Create creator profile.
2. Build template.
3. Publish listing.
4. First install.
5. First revenue.

---

## 18. Monitoring & Observability

### 18.1 Recommended Stack

| Need | Tool |
|---|---|
| Errors | Sentry |
| Metrics | Grafana |
| Logs | Loki |
| Tracing | OpenTelemetry |
| Uptime | Better Stack or Uptime Kuma |

### 18.2 Observability Requirements

- API error tracking.
- Worker job failures.
- Workflow run logs.
- AI request latency.
- Model cost tracking.
- Deployment build logs.
- User audit logs.

### 18.3 AI Observability

Track:

- Provider.
- Model.
- Prompt version.
- Tokens input/output.
- Latency.
- Cost.
- Error/fallback reason.
- User rating if available.

---

## 19. Security

### 19.1 Required

- RBAC.
- Rate limiting.
- API isolation.
- Encrypted secrets.
- Audit logs.
- Workflow sandboxing.
- Input validation.
- Webhook signature verification.
- Secret redaction in logs.

### 19.2 Secret Management

Secrets include:

- API keys.
- OAuth tokens.
- Database credentials.
- Deployment credentials.
- Webhook secrets.

Rules:

- Never store secrets in plaintext.
- Encrypt at rest.
- Redact logs.
- Scope secrets by organization/project.
- Rotate when needed.

### 19.3 Workflow Sandboxing

High-risk area.

Need protection from:

- Arbitrary code execution.
- Data exfiltration.
- Infinite loops.
- SSRF.
- Credential leakage.
- Malicious marketplace templates.

MVP approach:

- No arbitrary custom code execution.
- Only approved node types.
- Strict timeout.
- Strict network allowlist where possible.
- Rate limits per org.

---

## 20. Marketplace Architecture

### 20.1 Creator Can Sell

| Type | Example |
|---|---|
| AI Templates | CRM, chatbot SaaS, support tool |
| AI Workflows | SEO automation, lead enrichment |
| AI Agents | Support bot, research agent |
| Plugins | Gmail, Slack, Notion integration |
| Themes | Dashboard UI, landing page kit |
| Prompt Packs | Sales prompts, support prompts |

### 20.2 Marketplace Systems

Required:

- Listings.
- Reviews.
- Ratings.
- Licensing.
- Creator payouts.
- Search.
- Fraud detection.
- Versioning.
- Install analytics.
- Compatibility metadata.

### 20.3 Template Packaging

Template should include:

- App pages.
- Components.
- Workflow graph.
- AI prompts.
- Required environment variables.
- Required integrations.
- Seed data.
- Billing config.
- Documentation.

### 20.4 Template Versioning

Required fields:

- Version number.
- Changelog.
- Compatibility.
- Migration notes.
- Published status.

---

## 21. Developer Experience

### 21.1 DX Goals

TESKEL harus:

- Easy to extend.
- Modular.
- Fast local dev.
- AI-friendly codebase.
- Strict but not slow.
- Easy onboarding for contributors.

### 21.2 Coding Standards

- Strict TypeScript.
- Feature-based architecture.
- Modular services.
- Clean APIs.
- Zod schemas at boundaries.
- Shared types where useful.
- Avoid circular dependencies.
- Avoid premature abstraction.

### 21.3 Local Dev

Ideal local command:

```bash
pnpm dev
```

Should run:

- Web app.
- API server.
- Worker.
- Database containers.
- Redis.

### 21.4 Testing Strategy

| Layer | Tool |
|---|---|
| Unit | Vitest |
| Integration | Vitest + test DB |
| E2E | Playwright |
| API Contract | Zod schemas + integration tests |
| Workflow Engine | Deterministic test fixtures |

---

## 22. AI-Assisted Development Workflow

### 22.1 Tools

| Tool | Purpose |
|---|---|
| Claude Opus | Architecture and complex coding |
| GPT-5.x | Rapid implementation |
| Cursor | AI IDE workflows |
| Windsurf | Agentic coding |
| GitHub Copilot | Inline acceleration |

### 22.2 Workflow

1. Define module.
2. Create specs and schemas.
3. AI generate scaffold.
4. Human review.
5. Add tests.
6. Refactor.
7. Ship fast.

### 22.3 AI-Friendly Repo Rules

- Keep features isolated.
- Use clear file names.
- Keep schemas near feature.
- Add README per major package later.
- Avoid giant files.
- Prefer explicit types.

---

## 23. Recommended Build Order

### PHASE 0 — Product & Technical Foundation

Duration: 2-3 weeks.

Build:

- Monorepo setup.
- Design system base.
- Landing page.
- Auth decision.
- Database schema v0.
- Project model.
- Basic dashboard shell.

Success criteria:

- User can sign up.
- User can enter dashboard.
- User can create organization/project.
- Design quality baseline established.

### PHASE 1 — Foundation MVP

Duration: 0-2 months.

Build:

- Auth.
- Dashboard.
- Projects.
- AI router.
- Basic AI chat/test console.
- Deployment stub or manual deployment integration.
- Billing base.

Success criteria:

- User can create project.
- User can call AI model.
- Usage is tracked.
- Basic subscription works.

### PHASE 2 — Template System

Duration: 2-4 months.

Build:

- Template cloning.
- Template metadata.
- Creator dashboard.
- Marketplace listing.
- Install flow.
- Template versioning.

Success criteria:

- Creator can publish a template.
- User can install template into project.
- Template can be deployed or previewed.

### PHASE 3 — Workflow System

Duration: 4-8 months.

Build:

- React Flow builder.
- Workflow JSON schema.
- Runtime engine.
- Node logs.
- AI nodes.
- API nodes.
- Schedule/webhook trigger.
- Workflow deployment.

Success criteria:

- User can build and run useful workflow.
- Workflow runs reliably in worker.
- Logs and errors are understandable.

### PHASE 4 — Visual App Builder

Duration: 6-10 months.

Build:

- Component builder.
- Properties panel.
- Theme editor.
- Responsive preview.
- App generation from template.
- AI-assisted editing.

Success criteria:

- User can customize template UI without coding.
- Changes are publishable.

### PHASE 5 — Ecosystem

Duration: 8-12 months.

Build:

- Plugins.
- SDK.
- Public APIs.
- Integrations.
- Advanced marketplace.
- Creator payouts.
- App review/moderation.

Success criteria:

- Third parties can build on TESKEL.
- Marketplace grows beyond internal templates.

---

## 24. MVP Scope Recommendation

### 24.1 Do First

MVP should focus on:

- Auth.
- Dashboard.
- Project creation.
- Template install.
- AI configuration.
- Basic deployment.
- Billing.
- Usage tracking.

### 24.2 Avoid in MVP

Avoid early:

- Kubernetes.
- Full plugin ecosystem.
- Full visual builder.
- Complex agent runtime.
- Arbitrary code execution.
- Enterprise SSO.
- Multi-region infra.
- Overly complex marketplace moderation.

### 24.3 Ideal MVP User Story

A user signs up, chooses “AI Support Bot SaaS”, configures model/provider, edits branding, connects Stripe, clicks deploy, and gets a working AI SaaS app.

---

## 25. Biggest Mistakes to Avoid

### A. Overengineering

Jangan langsung microservices. Start modular monolith with clear boundaries.

### B. Building Infra Too Early

Jangan Kubernetes awal. Use Docker + Coolify or managed platform until real scale pain appears.

### C. Bad UX

Builder rumit = gagal. Make default path easy and advanced features progressive.

### D. Too Many Features

Fokus awal:

- Launch AI business cepat.
- Template install.
- AI config.
- Deploy.
- Monetize.

### E. Weak Template Quality

Marketplace akan gagal jika template tidak production-grade. Internal templates pertama harus sangat bagus.

### F. No Usage/Cost Visibility

AI cost bisa membunuh margin. Track token and model cost from day one.

### G. Unsafe Workflow Runtime

Workflow engine tanpa sandbox dan limit bisa berbahaya. Mulai dengan node yang terkontrol.

---

## 26. Suggested First Templates

### 26.1 AI SaaS Templates

- AI Customer Support Bot.
- AI CRM Assistant.
- AI Content Repurposer.
- AI Resume Reviewer.
- AI Research Assistant.
- AI Meeting Notes SaaS.
- AI Lead Enrichment Tool.

### 26.2 Workflow Templates

- Website SEO audit automation.
- Lead scrape and enrich flow.
- Support ticket triage.
- Blog to social posts pipeline.
- Invoice extraction workflow.
- Daily competitor monitoring.

### 26.3 Agent Templates

- Research agent.
- Customer support agent.
- Sales qualification agent.
- Product feedback analyst.
- Code review assistant.

---

## 27. Initial Database Entity Map

High-level relationships:

```txt
User
  -> OrganizationMember
  -> Organization
      -> Project
          -> Workflow
          -> Deployment
          -> AIRequest
          -> UsageLog
      -> Subscription
      -> Secrets

CreatorProfile
  -> Template
      -> TemplateVersion
      -> MarketplaceListing
      -> TemplateInstall
      -> Review
```

Key rule:

- `Organization` is the tenant boundary.
- `Project` is the product/workspace boundary.
- `Template` is reusable package.
- `WorkflowVersion` and `TemplateVersion` are immutable after publish.

---

## 28. Technical Risk Areas

### 28.1 Deployment Automation

Risks:

- Build failures.
- Secret injection.
- Domain provisioning.
- SSL setup.
- Rollback.

Mitigation:

- Start with limited template runtime.
- Use standardized Dockerfile.
- Keep deployment logs visible.
- Provide one-click rollback later.

### 28.2 Workflow Runtime

Risks:

- Infinite loops.
- Failed retries.
- Partial execution.
- External API errors.
- Cost explosions.

Mitigation:

- Max execution time.
- Max node count.
- Retry policy.
- Idempotency keys.
- Usage caps.

### 28.3 Marketplace Trust

Risks:

- Low-quality templates.
- Malicious templates.
- Fake reviews.
- License abuse.

Mitigation:

- Review process.
- Verified creators.
- Security scan.
- Version approval.
- Refund policy.

### 28.4 AI Cost Control

Risks:

- Expensive model defaults.
- User abuse.
- Runaway workflows.

Mitigation:

- Daily/monthly limits.
- Model routing rules.
- Provider fallback.
- Cost dashboard.

---

## 29. Strategic Positioning

TESKEL bukan:

- Website builder biasa.
- AI wrapper tipis.
- Marketplace template biasa.
- n8n clone.
- Vercel clone.

TESKEL adalah:

**Infrastructure + ecosystem untuk melahirkan AI-native businesses.**

Strategic moat:

- Template ecosystem.
- Workflow runtime.
- AI-native deployment path.
- Creator economy.
- Usage/billing infrastructure.
- Fast launch experience.

---

## 30. Reference Repositories & Docs

### Frontend, Monorepo, UI

- shadcn/ui monorepo docs: `https://ui.shadcn.com/docs/monorepo`
- Turborepo: `https://turbo.build/repo`
- Next.js App Router: `https://nextjs.org/docs/app`
- SaaS Boilerplate by ixartz: `https://github.com/ixartz/SaaS-Boilerplate`

### Workflow Builder

- React Flow / xyflow: `https://github.com/xyflow/xyflow`
- React Flow showcase: `https://reactflow.dev/showcase`
- Automation workflow examples: `https://github.com/Azim-Ahmed/Automation-workflow`

### Visual Builder

- Craft.js: `https://github.com/prevwong/craft.js`
- Craft.js docs: `https://craft.js.org/`
- Puck visual editor: `https://github.com/puckeditor/puck`
- Builder.io open source repo: `https://github.com/BuilderIO/builder`

### Backend & Database

- Hono: `https://hono.dev/`
- Prisma examples: `https://github.com/prisma/prisma-examples`
- PostgreSQL: `https://www.postgresql.org/`
- pgvector: `https://github.com/pgvector/pgvector`

### Infra, Queue, Analytics

- BullMQ: `https://bullmq.io/`
- Redis: `https://redis.io/`
- Coolify: `https://coolify.io/`
- Cloudflare R2: `https://developers.cloudflare.com/r2/`
- PostHog: `https://posthog.com/`
- Sentry: `https://sentry.io/`
- OpenTelemetry: `https://opentelemetry.io/`

### AI Layer

- OpenRouter: `https://openrouter.ai/`
- Vercel AI SDK: `https://sdk.vercel.ai/`
- LangGraph: `https://github.com/langchain-ai/langgraph`
- Mastra: `https://mastra.ai/`

---

## 31. Final Recommendation

Build TESKEL as a **modular monorepo SaaS platform** with this sequence:

1. **Foundation**: auth, dashboard, projects, database, design system.
2. **AI Core**: model router, prompt engine, usage tracking.
3. **Template MVP**: installable AI SaaS templates.
4. **Deployment MVP**: standardized Docker/Coolify pipeline.
5. **Billing**: subscriptions and usage metering.
6. **Workflow Builder**: React Flow UI and worker runtime.
7. **Marketplace**: creators, listings, installs, revenue.
8. **Visual Builder**: Craft.js/Puck-style app customization.
9. **Ecosystem**: SDK, plugins, public APIs.

Most important early product metric:

**How fast can a new user go from signup to deployed AI product?**

If TESKEL can consistently deliver that in under 10 minutes, the product direction is strong.
