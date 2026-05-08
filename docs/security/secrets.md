# Secrets registry

Source-of-truth list of every secret TESKEL holds. Used by the
[`rotate-secret`](../../.agents/skills/rotate-secret/SKILL.md) skill.

> **Hard rule:** if a secret isn't in this registry, it doesn't
> exist as far as ops is concerned. Adding a real secret without
> updating this file is a Sev3 compliance defect.

## Layout

Each secret has:

- **Slug** — env-var-style key (`STRIPE_SECRET_KEY`).
- **Owner** — who is paged when rotation breaks.
- **Type** — what category (see below).
- **Vault** — where the canonical copy lives (1Password vault path).
- **Distribution** — how it reaches each surface (Doppler project,
  Coolify env, GH Actions secret, etc.).
- **Dual-window?** — whether `<NAME>_NEXT` exists for zero-downtime
  rotation.
- **Rotation cadence** — target frequency.
- **Last rotated** — date of last successful rotation.
- **Audit log query** — saved query to confirm last rotation was
  audited.

## Categories

| Category | Examples | Default cadence | Dual-window? |
| --- | --- | --- | --- |
| Vendor API keys | `STRIPE_SECRET_KEY`, `RESEND_API_KEY`, `OPENROUTER_API_KEY` | 90 days | Yes |
| Webhook signing | `STRIPE_WEBHOOK_SECRET`, `GITHUB_WEBHOOK_SECRET` | 90 days | Yes |
| OAuth client secrets | `GOOGLE_OAUTH_CLIENT_SECRET`, `GITHUB_OAUTH_CLIENT_SECRET` | 180 days | Yes |
| JWT signing keys | `BETTER_AUTH_JWT_PRIVATE_KEY`, `INTERNAL_RPC_JWT_KEY` | 30 days | Yes (key ring) |
| DB passwords | `DATABASE_URL` (app role), `WAL_G_S3_*` | 90 days | Yes (dual-role) |
| Encryption keys | `PGENCRYPT_PRIVATE_KEY`, `R2_OBJECT_ENCRYPTION_KEY` | Annual + on incident | Yes (key ring) |
| SSH keys | Coolify deploy key, backup ssh | 180 days | No (rotate then update) |
| Internal tokens | `ADMIN_SUPPORT_BEARER`, `INTERNAL_CRON_TOKEN` | 30 days | Yes |
| Per-tenant API keys | Customer-issued via UI | Customer-controlled | n/a — customer rotates |

## Registry table

> Populated as services come online. Empty for now (Phase 0 has not
> begun yet). When Phase 0 ships, this table is populated by the
> first wave of secrets and updated on every rotation.

| Slug | Owner | Type | Vault path | Distribution | Dual-window | Cadence | Last rotated |
| --- | --- | --- | --- | --- | --- | --- | --- |
| _placeholder_ | | | | | | | |

## Rotation procedure

Always: [`rotate-secret`](../../.agents/skills/rotate-secret/SKILL.md).
Never: ad-hoc.

The skill enforces:

1. Two-person rule: requester + reviewer in 1Password approval.
2. Dual-window write: store new value in `<NAME>_NEXT`, deploy,
   rotate config to use `<NAME>` = old `<NAME>_NEXT`, redeploy,
   delete old.
3. Audit log entry per phase (`secret.rotation.requested`,
   `…approved`, `…distributed`, `…verified`, `…retired`).
4. Update this registry's `Last rotated` column in the same PR
   that closes the rotation.

## Special cases

### Better Auth JWT signing key

Operates as a **key ring** with multiple active public keys:

- `kid_2026_05` — current (signs new tokens).
- `kid_2026_02` — previous (still verifies old tokens until
  expiry).
- `kid_2025_11` — retired (kept for forensic verification, no
  longer accepted).

Rotation = add new `kid`, swap signing pointer, leave old in the
ring until all sessions issued under it expire.

### Postgres app password

Two roles: `app_v1` (current) and `app_v2` (next). Application
config switches by changing the `DATABASE_URL` username; old role
stays valid until all instances are confirmed switched, then
revoked.

### Webhook secrets (inbound)

Use multi-secret verifier — the
[`add-webhook-receiver`](../../.agents/skills/add-webhook-receiver/SKILL.md)
skill checks all configured secrets, allowing `<NAME>` and
`<NAME>_NEXT` to overlap during rotation.

### Customer-controlled API keys

For per-tenant API keys (e.g., a customer's API key into TESKEL),
TESKEL stores only the **prefix + bcrypt hash + last-4**. No raw
key. Rotation is initiated by the customer in their settings UI;
backend issues a new key and revokes the old per the
[`add-rbac-role`](../../.agents/skills/add-rbac-role/SKILL.md)
audit conventions.

## Compromise procedure

If a secret is suspected compromised (leaked in chat, posted in a
PR, found in a public log, employee offboard, vendor breach):

1. **Open Sev1 incident** (`incident-sev1` skill).
2. **Rotate immediately** via `rotate-secret`, mode `compromise`.
3. **Revoke old** at the source (vendor dashboard, DB role,
   Better Auth key ring) **before** rotation distribution
   completes — accept short downtime over continued exposure.
4. **Audit-log review:** every `*.<secret>` event since last
   rotation.
5. **Vendor incident:** notify the vendor's security team if
   their key was leaked from our side.
6. **Customer impact assessment:** if PII / financial / auth
   secrets were involved, file regulatory notice per legal review
   (GDPR Art. 33: 72h).

## Auditing

- Every secret has an `audit_events` query that returns its full
  history of rotation/use. Saved queries live in
  `ops/audit/secrets/<slug>.sql`.
- Quarterly: secrets registry review (this file). Verify cadence
  is being met; flag overdue rotations as Sev3.
- Annually: external auditor walks this file with the rotation
  audit log to verify no orphaned secrets and no rotation gaps.

## Related

- [`rotate-secret`](../../.agents/skills/rotate-secret/SKILL.md) —
  rotation procedure.
- [`add-integration`](../../.agents/skills/add-integration/SKILL.md) —
  how new vendor secrets enter the registry.
- [`add-rbac-role`](../../.agents/skills/add-rbac-role/SKILL.md) —
  how internal tokens are issued and revoked.
- Plan §47 (Secrets management) — design intent.
