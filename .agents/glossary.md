# Glossary — TESKEL terminology

Definitions for terms an agent (or new engineer) sees across plan,
AGENTS.md, skills, and code. Sorted alphabetically. When two terms
mean almost-but-not-quite the same thing in TESKEL, the difference
is called out explicitly.

> If a term is missing, add it in your PR — even a one-liner. The
> glossary is append-only via PR, never deleted (rename via
> "deprecated → see X").

## A

**ADR** — Architecture Decision Record. Immutable record of a
chosen path. See [`docs/adr/`](../docs/adr/README.md). Skill:
[`write-adr`](./skills/write-adr/SKILL.md).

**AGENTS.md** — operating manual at the repo root. The first thing
any agent reads. Source of truth on hard rules.

**Audit chain / hash-chained log** — every `audit_events` row
includes a hash of the previous row's hash + its own contents,
forming a Merkle-ish chain. Tampering with one row invalidates
everything after it. See Plan §51.

## B

**Block** — a Puck visual builder unit. Has a Zod schema, a
React render component, and configurable props. See
[`add-block`](./skills/add-block/SKILL.md).

**Brief (phase brief)** — `docs/phases/<NN>-<slug>/brief.md`. The
human-readable companion to `current-phase.md`'s machine-readable
state. Written by [`kickoff-phase`](./skills/kickoff-phase/SKILL.md).

**BullMQ** — primary worker/queue runtime for short-lived jobs.
See ADR-0003 (planned). Inngest is the secondary jalur for
step-based workflows.

## C

**Canary release** — staged deploy where the new version receives
~5–10% of traffic for an observation window before full rollout.
See [`release-canary`](./skills/release-canary/SKILL.md).

**Coolify** — self-hosted PaaS we run our deploys on. See ADR-0010
(planned).

**Creator** — a person or org that publishes templates to the
TESKEL marketplace. See [`publish-template`](./skills/publish-template/SKILL.md).

**Current phase** — the active build phase, recorded in
[`.agents/state/current-phase.md`](./state/current-phase.md). Bound
by `allowed_scopes`.

## D

**DLQ** — dead-letter queue. Where BullMQ jobs go after exhausting
retries. Replayable from admin tooling.

**Drizzle** — TypeScript ORM we use for Postgres. See ADR-0002
(planned).

**DSAR** — Data Subject Access Request. GDPR/CCPA/PDP-Indonesia
right of access, deletion, etc. See
[`gdpr-data-request`](./skills/gdpr-data-request/SKILL.md).

## E

**E2B** — sandboxed code-execution runtime for workflow code nodes.
See ADR-0008 (planned).

**Exit criteria** — verifiable, binary checks that close a phase.
Listed in `current-phase.md` and the phase brief.

## F

**Feature flag** — runtime toggle (default: PostHog). Used for
rollout, kill-switch, A/B, and entitlement. See
[`add-feature-flag`](./skills/add-feature-flag/SKILL.md).

## G

**GameDay** — controlled chaos drill. Quarterly minimum. See
[`gameday-drill`](./skills/gameday-drill/SKILL.md).

**Goldenpath** — the ideal happy-path traversal of a flow.
TESKEL has six goldenpaths (Plan §15). E2E tests cover at least
these.

## H

**Hard rule** — a rule from `AGENTS.md §8` that is non-negotiable
without an RFC. Example: "every tenant-owned table has RLS".

**Hotfix** — emergency fix shipped outside the normal release train.
See [`release-hotfix`](./skills/release-hotfix/SKILL.md).

## I

**Idempotency** — a property: applying the operation twice has the
same effect as applying it once. Required for: workers, retries,
webhook handlers, payment ops.

**Inngest** — secondary workflow runtime for step-based,
durably-resumable functions. Complements BullMQ. See ADR-0003
(planned).

**Integration** — wrapped third-party service in
`packages/integrations/<vendor>/`. See
[`add-integration`](./skills/add-integration/SKILL.md).

## J

(no entries yet)

## K

**Kill-switch** — a feature flag whose only job is "turn this off
in production fast". Every integration and every cron has one.

## L

**Langfuse** — LLM observability platform. We trace every prompt
call, including model, latency, cost, and outcome. See ADR-0005
(planned).

## M

**Marketplace** — TESKEL's listing of templates. Buyers install,
creators publish. See [`publish-template`](./skills/publish-template/SKILL.md).

**Monorepo** — single repo holding all TESKEL packages, apps,
shared code. Managed with pnpm workspaces + Turborepo.

## N

**Next.js (App Router)** — frontend framework. Server Components
default; explicit Client Components only when interactive. See
ADR-0009 (planned).

## O

**Org** — short for "organization". The tenant boundary. Every
multi-tenant table has `org_id`. Aliased with **tenant** in this
codebase but `org_id` is the column.

**OpenRouter** — default AI gateway router. Multi-model routing,
fallback, cost guardrails. See ADR-0005 (planned).

**Outbox** — pattern: writes that need reliable side-effects
(emails, webhooks, third-party calls) go through an `outbox` table
that workers drain. Eliminates lost messages on transaction
rollback.

**Owner** (CODEOWNERS) — the engineer or team with merge authority
for an area. Mirror in [`state/owners.md`](./state/owners.md).

## P

**Phase** — one of seven build phases (0–6). Each phase has a
brief, `allowed_scopes`, and exit criteria. See
[`docs/phases/`](../docs/phases/README.md).

**pgvector** — Postgres extension we use for KB embeddings. See
ADR-0006 (planned).

**Plan** — [`TESKEL_FULLSTACK_BUILD_BREAKDOWN.md`](../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md).
The strategic document.

**PostHog** — analytics + feature flags. Self-hosted in Phase 1+.

**Prompt slot** — versioned prompt template registered in the
prompt registry. See [`add-prompt-slot`](./skills/add-prompt-slot/SKILL.md).

**Puck** — visual builder library used in TESKEL. Craft.js is the
fallback for cases Puck can't model. See ADR-0007 (planned).

## Q

(no entries yet)

## R

**RBAC** — Role-Based Access Control. See
[`docs/security/rbac-matrix.md`](../docs/security/rbac-matrix.md)
and [`add-rbac-role`](./skills/add-rbac-role/SKILL.md).

**Resend** — transactional email provider. See
[`add-email-template`](./skills/add-email-template/SKILL.md).

**RFC** — Request for Comments. Exploratory design doc that
converges on a recommendation; usually followed by an ADR. See
[`docs/rfc/`](../docs/rfc/README.md).

**RLS** — Postgres Row-Level Security. The **only** tenant
boundary in TESKEL (planned ADR-0004). Every tenant-owned table
has policies enforcing `org_id = auth.org_id()`.

**Runbook** — operational playbook attached to an alert. See
[`docs/runbooks/`](../docs/runbooks/README.md) and
[`add-runbook`](./skills/add-runbook/SKILL.md).

**RPO / RTO** — Recovery Point / Time Objectives. TESKEL targets
RPO ≤5 min, RTO ≤2 h. See
[`db-restore-pitr`](./skills/db-restore-pitr/SKILL.md).

## S

**Sandbox** — runtime for executing untrusted user code. Default:
E2B. See Plan §31.

**Sev1 / Sev2 / Sev3 / Sev4** — incident severity. See
[`incident-sev1`](./skills/incident-sev1/SKILL.md) and
[`incident-sev2`](./skills/incident-sev2/SKILL.md).

**Skill** — `.agents/skills/<slug>/SKILL.md`. Reusable checklist
for a recurring operation.

**Slot** — abstract: the place where a customizable thing plugs in.
TESKEL uses "prompt slot" specifically (a versioned prompt
template).

**Story** — unit of trackable work. Filed under
[`docs/stories/`](../docs/stories/README.md). Phase brief decomposes
into stories.

## T

**Template** — packaged workflow + UI + assets that creators
publish to the marketplace and customers install. See
[`publish-template`](./skills/publish-template/SKILL.md).

**Tenant** — colloquial for "org". The isolation boundary.

**Turborepo** — monorepo task orchestrator (caching + remote
caching). Pairs with pnpm workspaces.

## U

(no entries yet)

## V

**Vault** — 1Password (canonical) → Doppler/Infisical
(distribution). Secrets never live in code, env files, or chat.
See [`docs/security/secrets.md`](../docs/security/secrets.md).

## W

**Watch window** — the period after a deploy when extra observation
is required. Default: 24h after a phase release; 1h after a normal
release.

**Webhook (inbound)** — vendor-pushed event we receive. See
[`add-webhook-receiver`](./skills/add-webhook-receiver/SKILL.md).

**Webhook (outbound)** — TESKEL-pushed event we send to a partner.
Always via the outbox.

**WSDAP** — TESKEL's North Star: "**W**orld's most **S**ynthesizing,
**D**emocratized **A**I **P**latform" (Plan §10). The single
sentence we use to litmus-test every decision.

## X

(no entries yet)

## Y

(no entries yet)

## Z

**Zod** — schema validation library. Used at every boundary
(HTTP request body, vendor response, prompt I/O, queue payload).

---

## Adding terms

When you find yourself defining a term twice in PR comments, add
it here. One sentence is fine; one paragraph is plenty. Cross-link
to the relevant skill / ADR / plan section.
