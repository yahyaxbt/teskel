---
name: rotate-secret
description: Rotate a secret (API key, DB password, JWT signing key, OAuth client secret, webhook secret, vendor token) safely with zero downtime. Covers planning, dual-key window, propagation, audit, and post-rotation verification. Use on schedule (90 days) or on suspected compromise.
---

# rotate-secret — rotate a secret with zero downtime

> Plan ref: [Sec. 47 (Secrets management)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#47-secrets-management),
> [Sec. 51 (Audit log)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#51-audit-log),
> [Sec. 76 (Failure modes)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#76-failure-modes--rollback).
>
> **Hard rule:** never paste secrets into chat, PR, or commits. Use the
> 1Password vault and the secrets manager (Doppler/Infisical/AWS SM).

## Decide rotation type

| Type | Strategy |
| --- | --- |
| Symmetric API key (Stripe, Resend, OpenRouter) | Provision new key → dual-config window → cut over → revoke old |
| Database password | Add 2nd login role → swap connection string → drop old role |
| JWT signing key (Better Auth) | Add new key as `next` → wait 1 token TTL → promote `next` to `current`, demote `current` to `previous` (verify-only) → drop `previous` after 1 token TTL |
| Webhook secret (inbound) | Provider must support multi-secret; otherwise stagger by routing through dual-listener |
| OAuth client secret | Provider-specific; usually dual-secret window 24–72h |
| SSH key | Add new key alongside; remove old after deploy verifies |
| Vendor SDK token | Same as API key |

## Generic playbook (15 steps)

1. **Pre-flight.**
   - Confirm rotation reason: scheduled / suspected compromise /
     departing team member / vendor breach.
   - If compromise: declare a Sev1 (see `incident-sev1` skill); this
     skill becomes part of remediation.
   - Identify owner + reviewer. Block any deploys not strictly part of
     the rotation.

2. **Inventory consumers.** Find every place the secret is used.

   ```bash
   pnpm grep "STRIPE_SECRET_KEY"   # find all references
   ```

   List: services (api, worker, scheduler), CI, scripts, edge functions,
   third-party tools.

3. **Generate the new secret.**
   - Vendor portal: create the new key with same scope as old.
   - For DB: `CREATE ROLE app_v2 LOGIN PASSWORD '<random32>'` with
     identical grants.
   - For JWT signing: `openssl rand -base64 64`.

4. **Store in vault.** 1Password → "TESKEL · Secrets". Add a new entry
   `STRIPE_SECRET_KEY (next)` with metadata: created, owner, reason,
   linked PR.

5. **Push to secrets manager** (Doppler/Infisical/AWS SM):
   - Add as a **new** key first (`STRIPE_SECRET_KEY_NEXT`), do not
     overwrite the active one.
   - Sync to **staging** first.

6. **Roll out dual-config to staging.**
   - Code reads both:
     ```ts
     const keys = [config.STRIPE_SECRET_KEY, config.STRIPE_SECRET_KEY_NEXT].filter(Boolean);
     ```
   - For verifying signed payloads, accept any of the configured
     secrets.
   - For outbound calls, prefer `_NEXT` if set; fall back to current.
   - Deploy to staging.
   - Run smoke tests.

7. **Verify in staging.**
   - Synthetic checks pass against the new key.
   - Logs show no secret values (regex `(secret|api_key|bearer)` on
     last 1h logs).

8. **Cut over to production (canary).**
   - Push `STRIPE_SECRET_KEY_NEXT` to prod secrets manager.
   - Roll one canary container/instance with the new key as primary.
   - Hold for at least 30 minutes; watch error rate, signature
     failures, 401s.

9. **Promote to 100%.**
   - Roll fleet with new key as primary, old key as fallback.
   - Continue to accept signatures from old key for the dual-window
     duration.

10. **Wait the dual-window.**
    | Secret type | Dual window |
    | --- | --- |
    | JWT signing | ≥1 access-token TTL (1h) |
    | Webhook | ≥7 days (vendor retry tail) |
    | API key | 24–72h depending on traffic |
    | DB role | until all PgBouncer pools have rotated |

11. **Revoke the old secret.**
    - Vendor portal: delete or disable.
    - Doppler/SM: remove `STRIPE_SECRET_KEY` (old).
    - Drop old DB role (`DROP ROLE app_v1`).
    - Update code to read only the new key (or just the canonical
      `STRIPE_SECRET_KEY` env var); remove `_NEXT` shim.
    - Deploy.

12. **Post-rotation verification.**
    - Synthetic checks still green.
    - Stripe dashboard: no recent calls failed with the old key.
    - Audit log entry recorded:
      ```
      audit.log({ action: 'secret.rotated', target: 'STRIPE_SECRET_KEY', actor: user.id, metadata: { reason, previousFingerprint } })
      ```

13. **Update the rotation registry.** `docs/security/secrets.md`:
    - Last rotated: today.
    - Next due: today + 90 days.
    - Owner unchanged.

14. **Clean up.**
    - Remove the dual-read code path (or convert to a generic
      multi-key loader if rotations happen often).
    - Confirm no lingering references to the old fingerprint in logs
      or commit history.

15. **Postmortem (only if rotation was due to compromise).**
    - 5-business-day write-up.
    - Identify how the secret leaked.
    - Add detection (CI secret-scanning, log scrubber, leakage canary).

## Special cases

- **Better Auth JWT signing rotation:** Better Auth supports a key
  ring; expose two keys via env (`AUTH_JWT_KEY`, `AUTH_JWT_KEY_NEXT`)
  and configure in `apps/api/src/auth/config.ts` to use both for
  verify, only `current` for sign. After dual-window, demote.

- **Postgres password rotation:**
  ```sql
  CREATE ROLE app_v2 LOGIN PASSWORD '...' IN ROLE app_role;
  -- Update PgBouncer userlist.txt or AWS RDS IAM auth as appropriate.
  -- Cut traffic via app config to use app_v2.
  -- Wait one full deploy cycle.
  DROP ROLE app_v1;
  ```

- **Webhook outbound (we sign and a partner verifies):**
  - Send `signature_v1` AND `signature_v2` headers during dual-window.
  - Coordinate with partner; track in
    `docs/integrations/<partner>.md`.

- **Compromise (key leaked publicly):** skip dual-window entirely;
  generate new key, deploy hotfix, revoke old immediately, accept
  short outage on inflight calls. Open Sev1 incident in parallel.

## Pitfalls

- Pasting secret into a Slack DM "for the deploy" — never. Use the
  vault.
- Deleting old key before fleet has fully rotated — outage.
- Forgetting CI secrets — deploys break next morning.
- Forgetting cron jobs / scheduled functions — they may use stale env.
- Missing audit log entry — compliance review failure.
- Logging the old key fingerprint in plaintext — leaks rotation
  signal.

## Done when

- New secret active in prod, old revoked.
- Dual-window observed.
- Audit log entry recorded.
- Rotation registry updated.
- No errors above baseline for ≥24h.
- Postmortem (if compromise) filed.
