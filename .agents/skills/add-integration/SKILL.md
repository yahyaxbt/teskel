---
name: add-integration
description: Add a new third-party integration (Stripe, Resend, Slack, OAuth provider, vendor SDK) with secrets, typed client, retry/backoff, circuit breaker, observability, and a kill-switch. Use when bringing any new external service into TESKEL.
---

# add-integration — add a third-party integration

> Plan ref: [Sec. 30 (Integration patterns)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#30-integrations--external-apis),
> [Sec. 47 (Secrets)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#47-secrets-management),
> [Sec. 26 (Idempotency, retry, DLQ)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#26-idempotency-retry-and-dlq).

> **Hard rule:** every external network call goes through a typed
> client in `packages/integrations/<vendor>/`. No vendor SDK is
> imported directly from product code.

## Decide the integration type

| Type | Examples | Notes |
| --- | --- | --- |
| Server-side SDK | Stripe, Resend, OpenRouter, Posthog server | Typed client wraps SDK |
| OAuth provider | Google, GitHub, Microsoft | Use Better Auth's social plugin |
| REST-only API | Internal vendor with no SDK | Build client with `ofetch` + Zod |
| Webhook (inbound) | Stripe events, GitHub events | See `add-webhook-receiver` skill |
| Webhook (outbound) | Send to partner | Use outbox + signed delivery |
| Browser SDK | PostHog client, Sentry client | Lazy-loaded, consent-gated |

## Steps

1. **Vendor due diligence (≤30 min).** Before importing anything:
   - Pricing & rate limits.
   - Data residency (where does the vendor process data?).
   - DPA / SOC2 / ISO available?
   - Auth model: API key, OAuth, mTLS, signed JWT?
   - SDK quality: maintained? typed? bundle size if browser?
   - Sandbox / test mode availability.
   - Status page + historical uptime.
   - Document findings in `docs/integrations/<vendor>.md`.

2. **Get the secret(s).** Provision vendor credentials:
   - **Test/sandbox** keys for dev + CI.
   - **Production** keys via 1Password vault → Doppler/Infisical.
   - Naming: `STRIPE_SECRET_KEY`, `STRIPE_WEBHOOK_SECRET`,
     `STRIPE_PUBLISHABLE_KEY` (no abbreviations, no app prefix).
   - Add to `packages/shared/config.ts` as a typed `z.string().min(1)`.
   - Never read `process.env` outside that file.

3. **Scaffold the package.**

   ```
   packages/integrations/stripe/
     src/
       index.ts        # public exports
       client.ts       # typed client (constructor, lazy init)
       types.ts        # vendor types re-exported / Zod-validated
       errors.ts       # vendor errors → TESKEL errors
       circuit.ts      # circuit breaker config
     package.json
     README.md
     vendor.test.ts    # contract tests against sandbox
   ```

   Run `add-package` skill if needed.

4. **Build the typed client.**

   ```ts
   // packages/integrations/stripe/src/client.ts
   import Stripe from 'stripe';
   import { config } from '@teskel/shared/config';
   import { logger } from '@teskel/shared/logger';
   import { trace } from '@opentelemetry/api';
   import { CircuitBreaker } from './circuit';
   import { mapStripeError } from './errors';

   let _client: Stripe | null = null;

   export function getStripe(): Stripe {
     if (!_client) {
       _client = new Stripe(config.STRIPE_SECRET_KEY, {
         apiVersion: '2024-12-18.acacia',
         maxNetworkRetries: 0,    // we manage retry
         timeout: 10_000,
       });
     }
     return _client;
   }

   const breaker = new CircuitBreaker({
     name: 'stripe',
     failureThreshold: 5,
     resetTimeoutMs: 30_000,
   });

   export async function callStripe<T>(
     op: string,
     fn: (s: Stripe) => Promise<T>,
   ): Promise<T> {
     const tracer = trace.getTracer('integration.stripe');
     return tracer.startActiveSpan(`stripe.${op}`, async (span) => {
       try {
         return await breaker.exec(() => fn(getStripe()));
       } catch (err) {
         throw mapStripeError(err);
       } finally {
         span.end();
       }
     });
   }
   ```

5. **Validate vendor responses with Zod.** Don't trust vendor types.

   ```ts
   import { z } from 'zod';
   const SubscriptionResp = z.object({
     id: z.string().startsWith('sub_'),
     status: z.enum(['active','past_due','canceled','incomplete','incomplete_expired','trialing','unpaid','paused']),
     current_period_end: z.number().int(),
   });
   const data = SubscriptionResp.parse(await callStripe('subscriptions.retrieve', s => s.subscriptions.retrieve(id)));
   ```

6. **Errors.** Map vendor errors to typed TESKEL errors so callers
   can branch:

   ```ts
   // packages/integrations/stripe/src/errors.ts
   export class StripeRateLimitError extends Error { code = 'stripe_rate_limit' as const; retryable = true; }
   export class StripeCardError extends Error { code = 'stripe_card' as const; retryable = false; }
   export class StripeUnknownError extends Error { code = 'stripe_unknown' as const; retryable = false; }
   export function mapStripeError(err: unknown): Error { /* ... */ }
   ```

7. **Retry policy.** Only retry idempotent operations. Use the
   helper:

   ```ts
   import { retry } from '@teskel/shared/retry';
   const sub = await retry(
     () => callStripe('subscriptions.retrieve', s => s.subscriptions.retrieve(id)),
     { attempts: 3, baseMs: 200, jitter: true, retryOn: (e) => 'retryable' in e && e.retryable === true },
   );
   ```

8. **Outbound webhooks (we send).** Use the outbox pattern. Don't
   `await fetch()` from the request handler.

9. **Inbound webhooks (vendor sends us).** See `add-webhook-receiver`
   skill — separate concern.

10. **Circuit breaker.** Threshold per vendor, default
    `5 failures / 30s window`. When open, fail fast for
    `resetTimeoutMs`; emit metric. Tune via vendor SLA.

11. **Kill-switch.** Wrap the integration in a feature flag
    (`flag.integration.<vendor>`). When flipped OFF, all calls
    short-circuit with a typed `IntegrationDisabledError`. Use during
    vendor outages to take pressure off retries.

12. **Observability.**
    - OTel span per call (`<vendor>.<operation>`).
    - Counter `integration_calls_total{vendor, op, outcome}`.
    - Histogram `integration_call_duration_seconds`.
    - Grafana dashboard `Integrations · <vendor>` (auto-generated from
      template).
    - Alert: error rate >5% over 5m; circuit-breaker open >5m.

13. **PII handling.**
    - Don't log request/response bodies in plaintext.
    - Vendor IDs (e.g., `cus_xyz`) are OK in logs.
    - PII fields (email, name) hashed/redacted in logs.
    - DPA filed; vendor listed in `/legal/subprocessors.md` (auto-
      generated from a registry).

14. **Tests.**
    - Unit: error mapping covers all vendor error classes.
    - Contract (against sandbox in CI): happy-path, rate-limit,
      auth-failure responses.
    - Smoke: real call to sandbox with synthetic identity, runs hourly
      in prod monitoring.

15. **Docs.** `docs/integrations/<vendor>.md` covers:
    - What it does.
    - Auth + secrets.
    - Endpoints we use + rate limits.
    - Failure modes and our mitigation.
    - Monitoring dashboards + runbooks.
    - DPA / data sharing.
    - Cost model + observed monthly cost.

16. **Subprocessor list.** If the vendor processes user data, add to
    `legal/subprocessors.md` (date, vendor, purpose, region).

## Pitfalls

- Importing `stripe` directly from product code — bypasses retry,
  breaker, telemetry, and kill-switch.
- Using `fetch` without a typed client — silent type drift.
- Logging full request/response bodies — leaks PII / secrets.
- Trusting vendor types as runtime contracts — vendors change shapes
  unannounced; always Zod-parse.
- Retrying non-idempotent ops (`charges.create`) — duplicate side
  effects. Use vendor's idempotency key instead, then optionally
  retry.
- Skipping the kill-switch — blast radius during outage is huge.
- Silently changing the API version mid-flight — pin it; bump in a
  PR with regression tests.

## Done when

- `packages/integrations/<vendor>` exists with typed client + tests.
- Secrets registered in config.ts and Doppler.
- Circuit breaker + retry helper used.
- Kill-switch flag exists.
- OTel + dashboards + alerts wired.
- `docs/integrations/<vendor>.md` written.
- Subprocessor list updated if applicable.
- Sandbox smoke test runs hourly in prod.
