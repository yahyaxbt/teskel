---
name: add-webhook-receiver
description: Add an inbound webhook endpoint that verifies vendor signatures, dedups events, persists raw payload to an audit table, queues processing async, and replays from DLQ. Use for any vendor-pushed event (Stripe, GitHub, Resend, Slack, OAuth providers).
---

# add-webhook-receiver — handle an inbound webhook

> Plan ref: [Sec. 30 (Integrations)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#30-integrations--external-apis),
> [Sec. 26 (Idempotency, retry, DLQ)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#26-idempotency-retry-and-dlq),
> [Sec. 51 (Audit log)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#51-audit-log).

> **Hard rule:** the HTTP handler **only** verifies and persists. All
> business logic runs in a worker. A handler that takes >500ms is a
> bug.

## Pattern at a glance

```
Vendor → POST /webhooks/<vendor>
   |     1. parse raw body (no JSON middleware before signature check)
   |     2. verify HMAC/Ed25519 with current + previous secret
   |     3. INSERT INTO webhook_events (raw, signature, vendor, ...)
   |        ON CONFLICT (vendor, vendor_event_id) DO NOTHING  -- dedup
   |     4. ENQUEUE webhook.process { id }
   |     5. respond 2xx fast (≤500ms)
   v
Worker (BullMQ)
   1. SELECT raw FROM webhook_events WHERE id = ?
   2. parse + Zod-validate body
   3. FOR EACH handler subscribed to (vendor, type) → run
   4. on success: mark processed_at; on failure: retry; after N: DLQ
```

## Steps

1. **Get vendor signing secret.** Stripe `whsec_...`, GitHub
   `X-Hub-Signature-256`, Slack `v0=...`. Add to config:

   ```ts
   STRIPE_WEBHOOK_SECRET: z.string().startsWith('whsec_'),
   STRIPE_WEBHOOK_SECRET_NEXT: z.string().startsWith('whsec_').optional(),
   ```

   `_NEXT` slot exists so [`rotate-secret`](../rotate-secret/SKILL.md)
   has a dual-window.

2. **Add the route — preserve the raw body.** Hono parses body lazily;
   read raw before JSON.

   ```ts
   // apps/api/src/routes/webhooks/stripe.ts
   import { Hono } from 'hono';
   import { db, schema } from '@teskel/db';
   import { config } from '@teskel/shared/config';
   import { logger } from '@teskel/shared/logger';
   import { queue } from '@teskel/queue';
   import { verifyStripe } from '@teskel/integrations/stripe/webhook';

   export const stripeWebhook = new Hono();

   stripeWebhook.post('/', async (c) => {
     const raw = await c.req.text();
     const sig = c.req.header('stripe-signature') ?? '';

     const verified = verifyStripe(raw, sig, [
       config.STRIPE_WEBHOOK_SECRET,
       config.STRIPE_WEBHOOK_SECRET_NEXT,
     ].filter(Boolean) as string[]);
     if (!verified) {
       logger.warn({ vendor: 'stripe' }, 'webhook_invalid_signature');
       return c.json({ error: 'invalid_signature' }, 400);
     }

     const eventId = verified.id;
     const eventType = verified.type;

     const [row] = await db.insert(schema.webhookEvents).values({
       vendor: 'stripe',
       vendorEventId: eventId,
       eventType,
       raw,
       signatureHeader: sig,
       receivedAt: new Date(),
     }).onConflictDoNothing({ target: [schema.webhookEvents.vendor, schema.webhookEvents.vendorEventId] }).returning();

     if (!row) {
       // Duplicate — vendor retry. Already enqueued.
       return c.json({ ok: true, dedup: true });
     }

     await queue.add('webhook.process', { id: row.id, vendor: 'stripe' }, {
       attempts: 5,
       backoff: { type: 'exponential', delay: 1_000 },
       removeOnComplete: { age: 3600, count: 1000 },
       removeOnFail: false, // keep failed jobs in DLQ
     });

     return c.json({ ok: true });
   });
   ```

3. **Signature verification.** Per-vendor helper in
   `packages/integrations/<vendor>/webhook.ts`. Always:
   - Use **constant-time** comparison (`crypto.timingSafeEqual`).
   - Validate timestamp window (replay protection, ≤5 min drift).
   - Verify against ALL configured secrets (current + previous), not
     just the active one.

   ```ts
   // packages/integrations/stripe/webhook.ts
   import Stripe from 'stripe';
   const stripe = new Stripe('dummy', { apiVersion: '2024-12-18.acacia' });

   export function verifyStripe(rawBody: string, signature: string, secrets: string[]) {
     for (const s of secrets) {
       try {
         return stripe.webhooks.constructEvent(rawBody, signature, s);
       } catch { /* try next */ }
     }
     return null;
   }
   ```

4. **`webhook_events` table.**

   ```sql
   CREATE TABLE webhook_events (
     id            uuid PRIMARY KEY DEFAULT gen_random_uuid(),
     vendor        text NOT NULL,
     vendor_event_id text NOT NULL,
     event_type    text NOT NULL,
     raw           text NOT NULL,
     signature_header text NOT NULL,
     received_at   timestamptz NOT NULL,
     processed_at  timestamptz,
     last_error    text,
     attempts      int NOT NULL DEFAULT 0,
     UNIQUE (vendor, vendor_event_id)
   );
   CREATE INDEX webhook_events_unprocessed_idx ON webhook_events (vendor, received_at) WHERE processed_at IS NULL;
   ```

   This is **not** tenant-owned (no `org_id`); webhooks are
   platform-level. Access from app surfaces is admin-only and audit-
   logged.

5. **Worker.** One worker per vendor; one handler registry per
   `event_type`.

   ```ts
   // packages/jobs/workers/webhook-stripe.ts
   import { worker } from '@teskel/queue';
   import { db, schema, sql } from '@teskel/db';
   import { stripeHandlers } from './handlers/stripe';

   worker('webhook.process', async (job) => {
     if (job.data.vendor !== 'stripe') return;

     const [row] = await db.select().from(schema.webhookEvents).where(sql`id = ${job.data.id}`);
     if (!row || row.processed_at) return;

     const handler = stripeHandlers[row.event_type];
     if (!handler) {
       // Unknown event type: mark processed-noop, log for analysis.
       await db.update(schema.webhookEvents)
         .set({ processed_at: new Date(), last_error: 'no_handler' })
         .where(sql`id = ${row.id}`);
       return;
     }

     try {
       await handler(JSON.parse(row.raw));
       await db.update(schema.webhookEvents)
         .set({ processed_at: new Date() })
         .where(sql`id = ${row.id}`);
     } catch (err) {
       await db.update(schema.webhookEvents)
         .set({ attempts: sql`attempts + 1`, last_error: String(err) })
         .where(sql`id = ${row.id}`);
       throw err;  // BullMQ will retry / DLQ
     }
   });
   ```

6. **Handler registry.** One file per `event_type`, kept tiny.

   ```ts
   // packages/jobs/workers/handlers/stripe.ts
   import { z } from 'zod';
   const SubUpdated = z.object({
     id: z.string(),
     data: z.object({ object: z.object({ id: z.string(), status: z.string() }) }),
   });

   export const stripeHandlers = {
     'customer.subscription.updated': async (raw: unknown) => {
       const evt = SubUpdated.parse(raw);
       // Update local subscription mirror; emit domain event.
     },
     'invoice.payment_failed': async (raw: unknown) => { /* ... */ },
     'charge.refunded':         async (raw: unknown) => { /* ... */ },
   } as const;
   ```

7. **Dedup.** Two layers:
   - **DB unique** on `(vendor, vendor_event_id)` prevents duplicate
     rows.
   - **Handler-level** idempotency: every handler must be safe to run
     twice. Use natural keys (vendor IDs) when writing to the DB.

8. **DLQ + replay.**
   - BullMQ stores failed jobs.
   - Admin UI at `/admin/webhooks/dlq` lets ops view + replay (or
     mark abandoned with reason).
   - CLI: `pnpm teskel webhooks replay --vendor=stripe --since=2026-05-01`.

9. **Observability.**
   - Counter `webhook_received_total{vendor, event_type, outcome}`.
   - Histogram `webhook_handler_duration_seconds{vendor, event_type}`.
   - Alert: signature failure rate >0.1% (rotation issue) → page.
   - Alert: DLQ size >100 → page.
   - Dashboard `Webhooks · <vendor>` shows received vs processed vs
     failed.

10. **Vendor admin URL.** Document the vendor's webhook config URL +
    selected event types in `docs/integrations/<vendor>.md`. Adding a
    new event type to subscribe to is itself a PR (so reviewers know
    new payloads are coming).

11. **Tests.**
    - Unit: signature verifier accepts valid + rejects tampered/replay/
      future-dated/wrong-secret.
    - Integration: POST with vendor fixture → `webhook_events` row +
      job enqueued.
    - Replay: dup POST → 200, no second job, no second handler call.
    - Worker: handler error → row attempts++ → DLQ after retries.

12. **Tenant impact.** Most webhooks ultimately mutate tenant data
    (e.g., subscription belongs to an org). The handler resolves
    `vendorEventId → orgId` via a mirror table (e.g.,
    `stripe_customers (stripe_customer_id, org_id)`) and uses
    `db.withTenant(orgId)` for writes.

## Pitfalls

- Calling `c.req.json()` before signature verification — most vendor
  signatures cover the **raw** body; reformatting breaks verification.
- Doing business logic in the handler — slow handler causes vendor
  retries → duplicates → harder dedup.
- Forgetting timestamp/replay window — old captured signatures could
  be replayed.
- Single secret slot — rotation breaks the integration.
- Logging the raw body — likely contains PII or financial data;
  log only `(vendor, event_type, vendor_event_id)`.
- Returning 500 on signature mismatch — return 400, vendor will stop
  retrying garbage.
- Returning 4xx on processing failure — vendor stops retrying. Always
  ack with 2xx after persisting; let the worker retry.

## Done when

- Route registered, signature verified with multi-secret support.
- `webhook_events` row inserted with dedup constraint.
- Worker + handler registry in place.
- DLQ + replay working in dev.
- Dashboards + alerts wired.
- Tests cover sig verify, dedup, retry, DLQ.
- Vendor admin URL + subscribed events documented.
