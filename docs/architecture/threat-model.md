# Threat model — STRIDE

> **Status:** Living. Reviewed every quarter and after every Sev1
> with security implications.
>
> **Governed by:** Plan §45 (STRIDE), §46 (auth), §47 (RBAC), §48
> (API key + signing), §51 (sandbox), §52 (AI guardrails).

This document enumerates threats *per surface* using STRIDE. It is
not exhaustive — it captures the threats we **explicitly designed
against**. Anything here labelled `OPEN` requires action (story or
ADR) before the relevant surface ships to production.

STRIDE = **S**poofing, **T**ampering, **R**epudiation,
**I**nformation disclosure, **D**enial of service, **E**levation of
privilege.

---

## 1. Surfaces

| # | Surface | Threats most relevant | Owner |
| --- | --- | --- | --- |
| 1 | Web app | S, T, I, E | Frontend + Identity |
| 2 | Public API | S, T, I, D, E | Backend |
| 3 | Workers / queue | T, R, E | Backend |
| 4 | Sandbox (E2B) | T, I, E (this is the sharpest knife) | Platform |
| 5 | AI gateway | I, D, E (prompt injection, exfil) | AI |
| 6 | Marketplace | T, I, E (template-as-attack-vector) | Marketplace |
| 7 | Identity / RBAC | S, E | Identity |
| 8 | Object storage (R2) | I, T | Backend |
| 9 | Outbound integrations | I, T | Integrations |
| 10 | Inbound webhooks | S, T, R | Integrations |
| 11 | Admin / support console | E, R, I | Identity |
| 12 | Customer apps (apps built on TESKEL) | full STRIDE — inherits TESKEL controls | Platform |

---

## 2. Per-surface STRIDE

### 2.1 Web app

| Threat | Mitigation |
| --- | --- |
| **S**: Stolen session cookie | HttpOnly + Secure + SameSite=Lax cookies; rotate on privilege change; Better Auth invalidates on password / email change. |
| **S**: CSRF on mutating actions | Server actions use Next.js `'use server'` + per-form CSRF token; API uses double-submit cookie + Origin/Referer check. |
| **T**: XSS via user content | React auto-escaping; CSP header (default `script-src 'self'`, no `unsafe-inline`); Markdown rendered through `rehype-sanitize` allowlist. |
| **T**: Open redirect | Allowlist on `next` params; reject absolute URLs that don't match own host. |
| **I**: PII leakage in client bundles | RSC by default; `'use client'` boundary lint; no DB calls in client code. |
| **E**: Privilege escalation via UI | Permissions checked server-side; UI gating is *cosmetic only*. |

### 2.2 Public API (Hono)

| Threat | Mitigation |
| --- | --- |
| **S**: Forged JWT / API key | Better Auth signs JWT with rotated key ring; API keys stored hashed (argon2id). |
| **T**: Body / param tampering | Zod schema at handler boundary; rejected requests fail-fast with typed error. |
| **T**: Race / TOCTOU on mutations | `Idempotency-Key` enforced for `POST/PUT/PATCH/DELETE`; row-level locking on conflict-prone updates. |
| **R**: Repudiated action | Mutating endpoints emit `audit_log` with hash-chain. |
| **I**: Cross-tenant read | RLS + `withTenant` (see [`multi-tenancy.md`](./multi-tenancy.md)). |
| **D**: Brute force on login / 2FA | Better Auth rate limit + exponential backoff; Cloudflare WAF rules. |
| **D**: Expensive query DoS | Cost budgets per tenant (LLM tokens, workflow runs, storage); query timeouts; pagination required. |
| **E**: Permission bypass | RBAC middleware required on every mutating handler; `add-api-route` skill enforces. |

### 2.3 Workers / queue

| Threat | Mitigation |
| --- | --- |
| **T**: Replay attack on outbox | Idempotency key on every outbound effect; sender deduplicates by `(event_id, destination)`. |
| **R**: Unidentifiable side-effects | Every outbox row references its triggering `event_id` + `tenant_id` + `actor`. |
| **E**: Worker crash leaks tenant context | `withTenant` is per-job; each retry resolves tenant fresh from `job.data`. Process never holds long-lived tenant state. |
| **D**: Poison message floods queue | Per-job retry cap; permanent failures move to DLQ; DLQ size triggers an alert + runbook. |

### 2.4 Sandbox (E2B)

This is the **sharpest** surface. Code in a `code` workflow node is
*untrusted* by definition. Treat every sandbox as compromised.

| Threat | Mitigation |
| --- | --- |
| **T**: Container escape | E2B firecracker isolation; no privileged containers; kernel pinned to vendor's hardened build. |
| **T**: Egress to attacker-controlled host | Default-deny egress; allowlist proxy (only documented integrations); DNS resolver locked. |
| **I**: Read host filesystem | No host mounts; sandbox FS is ephemeral, unmounted on shutdown. |
| **I**: Read other tenants' sandboxes | Per-job sandbox; pre-warmed pool gated by tenant; sandbox process killed at job end. |
| **D**: Fork-bomb / mem-bomb | CPU + RSS + pid + nofile cgroups; wall-clock cap (default 60s, max 5 min); disk quota. |
| **D**: Outbound DDoS | Egress rate-limited per tenant; only HTTPS to allowlisted hosts. |
| **E**: Privilege escalation via syscall | seccomp profile (allowlist); SUID binaries removed from base image. |

Hardening details and the egress allowlist live in (planned)
`docs/architecture/sandbox.md`. Until that exists, the above is the
contract.

### 2.5 AI gateway

| Threat | Mitigation |
| --- | --- |
| **I**: Prompt injection exfiltrating PII | Output content filter checks for known data patterns (emails, card numbers); slots that include PII in inputs are reviewed by privacy. |
| **I**: Tool-use exfiltration | LLM tool calls run inside `withTenant`; each tool requires explicit slot-level allowlist. |
| **I**: Training data leakage | Default model selection prefers no-train providers; per-tenant data **never** sent to fine-tuning unless tenant opted in. |
| **D**: Unbounded LLM spend | Per-tenant + per-slot token budget enforced **before** call; circuit breaker per provider. |
| **E**: Privilege confusion via LLM "agent" | Agent runs as the tenant's session, never as `system.platform_admin`; impersonation forbidden. |

### 2.6 Marketplace

A template is **untrusted code + config**. Even after moderation we
treat installed templates as if they could be malicious.

| Threat | Mitigation |
| --- | --- |
| **T**: Template manifest manipulation post-publish | Manifest signed with creator's key; install verifies signature against pinned creator identity. |
| **I**: Template exfils buyer data | Workflow nodes inside an installed template run in buyer's tenant context; HTTP node egress allowlist applies; LLM calls use buyer's budget. |
| **E**: Template requests elevated permissions silently | Manifest declares required scopes; install UI shows scopes; user must explicitly approve. |
| **R**: Reviews / ratings sybil | Reviews tied to verified purchases; rate-limit + abuse heuristics. |

### 2.7 Identity / RBAC

| Threat | Mitigation |
| --- | --- |
| **S**: Account takeover | 2FA / passkey enforced for `org.owner`, `system.*` roles; magic-link cooldown + IP heuristics; recent-IP rule for sensitive actions. |
| **E**: Role explosion / shadow admin | Permissions registry is exhaustive; new permission requires PR + reviewer; role assignments emit `audit_log`. |
| **E**: SSO IdP compromise (Phase 4) | SCIM deprovision on IdP user disable; "break glass" tenant-owner credentials stored in vault, last-resort only. |

### 2.8 Object storage (R2)

| Threat | Mitigation |
| --- | --- |
| **I**: Direct object access bypassing app | Bucket private; only signed URLs; signing helper enforces tenant prefix. |
| **T**: Tampering with stored object | Server-side write only; client uploads go via signed PUT with content-length + content-type pinned; checksum recorded. |
| **D**: Storage exhaustion | Quota per tenant; soft-warn at 80%, hard-stop at 100%. |

### 2.9 Outbound integrations

| Threat | Mitigation |
| --- | --- |
| **I**: Secret leaked in URL or logs | Secrets only in headers / body; logger redacts known sensitive headers. |
| **T**: Mid-session credential rotation breaks state | Dual-window rotation per [`rotate-secret`](../../.agents/skills/rotate-secret/SKILL.md). |
| **D**: Vendor outage cascades | Circuit breaker per integration; documented kill-switch via flag. |

### 2.10 Inbound webhooks

| Threat | Mitigation |
| --- | --- |
| **S**: Forged webhook | Signature verify required; secret per integration; raw body preserved before parsing. |
| **T**: Replay | Persist `(provider, event_id)` and reject duplicates; audit table holds raw payload for forensic. |
| **R**: Inability to reconstruct what happened | Raw payload + headers + timing logged; replayable from DLQ. |

### 2.11 Admin / support console

| Threat | Mitigation |
| --- | --- |
| **E**: Support agent abusing access | All actions emit `audit_log` with operator identity + reason; high-risk actions require dual-control (Sec. 5.2 below). |
| **R**: "It wasn't me" | Audit log is hash-chained and exported daily to immutable storage. |
| **I**: Bulk export of customer data | Bulk export limited to GDPR DSAR pipeline; ad-hoc bulk export blocked. |

### 2.12 Customer apps (apps **built on** TESKEL)

Customer apps inherit *all* TESKEL controls. We do **not** provide an
escape hatch for customers to disable RLS, signed URLs, audit log,
or budget guards in their own apps.

---

## 3. Cross-cutting controls

### 3.1 Encryption

- TLS 1.2+ everywhere; HSTS preload (planned at GA).
- At rest: Postgres encrypted volumes; R2 SSE; Redis encrypted at
  rest where supported.
- Field-level encryption (envelope) for: 2FA seeds, OAuth refresh
  tokens, customer-controlled API keys (Phase 1+).

### 3.2 Logging

- Every request gets `request_id`, `tenant_id`, `user_id`,
  `idempotency_key` (where present).
- PII is **not** logged. Lint rule rejects logging of `email`,
  `phone`, raw payloads of webhooks, and prompt input.
- Audit log is separate from app log; hash-chained; exported daily.

### 3.3 Secrets

- Sourced via env at boundary; validated by Zod once at boot.
- Stored encrypted in Coolify; mirrored to 1Password Vault for
  ops use only.
- Rotation per [`rotate-secret`](../../.agents/skills/rotate-secret/SKILL.md).
- Compromise procedure in [`docs/security/secrets.md`](../security/secrets.md).

### 3.4 Build supply chain

- pnpm with frozen lockfile; allow-list of approved packages
  enforced via `pnpm audit` + manual review for new top-level deps.
- CI runs SCA (Trivy) on container images.
- Dockerfile uses minimal base (`node:20-bookworm-slim`); rootless
  user; read-only filesystem at runtime.
- Image signing with cosign at GA.

### 3.5 Dual-control actions (require two operators)

- Disabling RLS on any table.
- Mass deletion of audit log rows (in fact: never; only retention
  job deletes per policy).
- Issuing a `system.platform_admin` role.
- Changing JWT signing key ring.
- Publishing a hotfix that touches auth.

---

## 4. Threats we **do not** fully mitigate (yet)

| Threat | Status | Plan |
| --- | --- | --- |
| Supply-chain attack via transitive npm dep | Partial | OPEN: enable Sigstore + provenance attestation by Phase 4. |
| Side-channel timing attack on token comparison | Mitigated for password / API key (constant-time compare). | OPEN: audit all secret comparison paths during Phase 4 SOC2 prep. |
| Insider threat at hosting provider (Coolify host VM) | Mitigated by encrypted disks + per-region keys; defense in depth. | OPEN: ADR for region-pinning + customer-managed keys (Phase 6). |
| Adversarial fine-tune model output | Out of scope for v1; we trust the LLM provider's safety layer. | OPEN: per-slot output filter, see Plan §52. |
| Quantum-safe TLS | Not addressed. | OPEN: revisit at Phase 6. |

`OPEN` items become stories in the appropriate phase; SOC2 / ISO
audit gates depend on them.

---

## 5. Update protocol

Update this file when:

1. A new surface is added.
2. A new threat is realized (post-incident retro).
3. A control changes (e.g. moving from JWT to PASETO).
4. Quarterly review per Plan §45.

Each update goes in the same PR as the change that motivates it,
linking the relevant ADR or incident.

---

## 6. References

- [`AGENTS.md` §9](../../AGENTS.md#9-securitydata-protection-rules) —
  security rules.
- Plan §45–§55 — security & compliance.
- [`multi-tenancy.md`](./multi-tenancy.md) — tenant isolation.
- [`docs/security/rbac-matrix.md`](../security/rbac-matrix.md).
- [`docs/security/secrets.md`](../security/secrets.md).
- `incident-sev1`, `incident-sev2`, `gameday-drill`, `rotate-secret`,
  `gdpr-data-request` skills.
