# Idempotency

> **Status:** Stable. Required reading before opening a PR that adds
> a non-`GET` route, a worker, an outbound integration, or any code
> that handles money.
>
> **Governed by:** Plan §22 (API), §27 (DB), §31 (queue).
> Hard rule in [`AGENTS.md` §8](../../AGENTS.md#8-architecture-hard-rules).

Idempotency is the property that retrying the same logical operation
produces the **same outcome** as performing it once. It is the
single biggest defense against partial-failure bugs in a distributed
system.

This document defines the **contract** every layer must implement.

---

## 1. The promise

If a client retries a request with the same `Idempotency-Key` within
the configured TTL:

1. The same **outcome** is produced (the underlying state is mutated
   at most once).
2. The same **response body** is returned.
3. The same **status code** is returned (modulo `Idempotent-Replay:
   true` header on replays).
4. Side effects (emails, charges, webhook deliveries) are emitted
   **at most once**.

If the client retries with the same key but a **different body**, the
server returns `409 Conflict` with code `IDEMPOTENCY_KEY_CONFLICT`.

---

## 2. Where it applies

| Layer | What is idempotent | Key |
| --- | --- | --- |
| Public API (non-`GET`) | One logical operation | `Idempotency-Key` header (caller supplies) |
| Workflow run step | One step execution | `(run_id, step_index, attempt)` (server-assigned) |
| BullMQ job | One job execution | `jobId` (caller supplies) |
| Outbox sender | One side effect | `(event_id, destination)` |
| Inbound webhook | One vendor event | `(provider, event_id)` |
| Outbound webhook | One delivery | `(subscription_id, delivery_id)` |
| Stripe webhook | One Stripe event | `Stripe-Signature` + `event.id` |
| Email send | One delivery | `(template_id, recipient_id, idempotency_key)` |

Every mutating handler that does not declare an idempotency key is a
**bug** and is rejected at PR review.

---

## 3. Public API contract

### 3.1 Header

```
Idempotency-Key: 01J9X3K2...        // ULID or UUID v7
```

- **Required** on `POST`, `PUT`, `PATCH`, `DELETE`.
- Caller-supplied; server does **not** generate.
- Recommended length 16–128 chars; printable ASCII.
- Server stores key for **24 hours** by default; longer for money-
  shaped routes (see §3.4).

### 3.2 Response semantics

| Caller | Server | Response |
| --- | --- | --- |
| First request | Process and persist | `2xx`, normal body |
| Retry, same body | Look up cached response | same `2xx`, same body, header `Idempotent-Replay: true` |
| Retry, different body | Conflict | `409`, code `IDEMPOTENCY_KEY_CONFLICT` |
| Retry while in-flight | Wait | hold up to 30 s for in-flight result, else `409`, code `IDEMPOTENCY_KEY_IN_FLIGHT` |
| Retry after TTL | Treat as new | new key required; old key gone |

### 3.3 Server implementation (per route)

The route handler:

1. Reads the `Idempotency-Key` header. If missing on a mutating
   route, returns `400` `IDEMPOTENCY_KEY_REQUIRED`.
2. Inserts a row in `idempotency_keys` with status `in_flight` —
   `INSERT … ON CONFLICT (tenant_id, key) DO NOTHING RETURNING …`.
3. If the insert returned a row → it's the first request; proceed.
4. If the insert returned no row → look up the existing row:
   - if `status = 'completed'` and request body hash matches →
     return cached response (set `Idempotent-Replay: true`).
   - if `status = 'completed'` and body hash differs → `409`
     `IDEMPOTENCY_KEY_CONFLICT`.
   - if `status = 'in_flight'` → poll up to 30 s; if still
     in-flight, return `409` `IDEMPOTENCY_KEY_IN_FLIGHT`.
5. After processing, update the row to `status = 'completed'` with
   the response body + status code, in the same transaction that
   commits the business change.
6. On exception, mark `status = 'failed'`. Failed keys are reusable
   only if the caller's logic confirms safe retry — by default the
   client should generate a new key on failure.

### 3.4 TTL by route shape

| Route shape | TTL |
| --- | --- |
| Default mutating | **24 h** |
| Payments / billing | **30 d** (Stripe charges, refunds, payouts) |
| Marketplace install / fork | **7 d** |
| Workflow run trigger | **24 h** |
| Cancel / abort | **24 h** |
| Bulk operations (e.g. import) | **7 d** |

---

## 4. Workflow steps

A workflow run is a sequence of steps. Each step is identified by
`(run_id, step_index)`. Each *attempt* of a step is identified by
`(run_id, step_index, attempt)`.

### 4.1 Worker contract

```ts
worker.process("workflow.step.run", async (job) => {
  const { runId, stepIndex, attempt, tenantId } = job.data;
  await db.withTenant(tenantId, async (tx) => {
    const step = await tx.query.workflowSteps.findFirst({
      where: and(
        eq(workflowSteps.runId, runId),
        eq(workflowSteps.index, stepIndex),
      ),
    });
    if (step.status === "succeeded") return;     // already done
    if (step.attempt > attempt) return;           // newer attempt is in flight

    try {
      const output = await runStep(step);
      await tx.update(workflowSteps)
        .set({ status: "succeeded", output, attempt })
        .where(eq(workflowSteps.id, step.id));
      await enqueueNextStep(tx, runId, stepIndex + 1);
    } catch (err) {
      // record failure, potentially retry, never exceed maxAttempts
    }
  });
});
```

### 4.2 Side effects in a step

A step may emit *exactly one* outbox row per attempt. The outbox row
carries the same `(run_id, step_index, attempt)` tuple, plus a
destination identifier. The outbox sender deduplicates downstream.

Forbidden:

- Emitting two outbox rows from the same step (split it into two
  steps instead).
- Calling an external API directly from a step without writing the
  outbox row first.
- Reusing the same `attempt` number after a crash without verifying
  the prior attempt's outcome.

---

## 5. Outbox + sender

The outbox pattern decouples *deciding* a side effect from *doing*
it. The decider writes a row in the same transaction as the business
state change. The sender is a separate worker that reads pending
rows and calls the integration.

### 5.1 Schema (illustrative)

```sql
create table outbox_events (
  id uuid primary key default gen_random_uuid(),
  tenant_id uuid not null,
  event_id uuid not null,                  -- correlates to source event
  destination text not null,               -- 'email', 'webhook:tenant_x', 'stripe', ...
  payload jsonb not null,
  status text not null default 'pending',  -- pending|sending|sent|failed
  attempts int not null default 0,
  next_attempt_at timestamptz not null default now(),
  created_at timestamptz not null default now(),
  sent_at timestamptz,
  failed_at timestamptz,
  last_error jsonb,
  unique (event_id, destination)           -- the dedup key
);
```

### 5.2 Sender invariants

1. Pull `status = 'pending' AND next_attempt_at <= now()` with
   `FOR UPDATE SKIP LOCKED`.
2. Mark `status = 'sending'` in the same transaction that locks.
3. Call the integration (typed client, retries inside the client).
4. On success, mark `status = 'sent'` with `sent_at = now()`.
5. On failure, increment `attempts`, set `next_attempt_at` per
   exponential backoff, mark `status = 'pending'` again. After max
   attempts → `status = 'failed'`, alert.
6. The integration call is itself idempotent — see §6.

### 5.3 Replay

A `failed` row can be retried by an operator (`audit-log-query`
skill). Replay regenerates the request to the integration with the
**same** dedup key, so duplicate delivery is avoided downstream.

---

## 6. Integration calls

Every typed client in `packages/integrations/<vendor>/` MUST:

1. Accept an idempotency key from the caller (use the outbox row id
   if not given).
2. Forward it to the vendor via the vendor-specific mechanism
   (Stripe: `Idempotency-Key` header; Resend: `idempotency_key`
   field; etc.).
3. Treat a vendor `409` / `idempotency_conflict` as a soft success
   (the operation already happened); log + mark sent.

If the vendor does not support idempotency keys natively, the client
implements **at-least-once + dedup at our side** (e.g. local cache
of `(operation, vendor_response_id)` for 24 h). If the vendor cannot
be deduplicated at our side either, document the limitation and
restrict the operation to non-money flows.

---

## 7. Inbound webhooks

Every inbound webhook handler:

1. Verifies the vendor signature.
2. Persists the raw payload + headers to `webhook_events` keyed by
   `(provider, event_id)` with `ON CONFLICT DO NOTHING`. Conflict =
   replay; respond `200` immediately.
3. Enqueues the work for async processing.
4. Responds `≤500 ms`.

See [`add-webhook-receiver` skill](../../.agents/skills/add-webhook-receiver/SKILL.md).

---

## 8. Money-shaped operations

For anything that touches Stripe (charges, refunds, payouts):

- TTL **30 d**.
- Idempotency key persisted forever in `audit_log` (so future
  reconciliation can correlate even after the cache TTL).
- The Stripe `Idempotency-Key` header is set to a derivative of our
  key, with a vendor-required prefix (`teskel_v1_`). The mapping is
  recorded so we can re-look up.
- Reconciliation job runs hourly; any drift between Stripe events
  and our mirror table pages on-call (per
  [`slo-sli.md`](../observability/slo-sli.md) §3.8).

---

## 9. Database operations (read-after-write)

Inside a single request:

- All mutations of a single resource happen in **one** transaction.
- Reads after writes within the same request use the same
  transaction (read-your-writes).
- Cross-resource workflows split into multiple steps (per §4) — each
  step is idempotent.

For multi-row updates that span tables, the canonical pattern is:

```ts
await db.withTenant(tenantId, async (tx) => {
  // 1. Read current state
  // 2. Validate
  // 3. Apply mutations
  // 4. Insert outbox rows for any side effects
  // 5. Insert audit_log row
});
```

If the transaction commits, all five steps happened. If it rolls
back, none did. There is no in-between.

---

## 10. Common mistakes (and how the system catches them)

| Mistake | Catch |
| --- | --- |
| Forgetting `Idempotency-Key` in client SDK. | SDK injects a ULID by default; opt-out requires explicit flag. |
| Reusing the same idempotency key for two different operations. | Server `409` on body mismatch. |
| Caching the response *outside* the transaction so business state succeeded but cache write failed. | Cache write is **inside** the transaction (same `tx`). |
| Sending email directly from a handler. | `add-api-route` skill forbids it; emails go through outbox. |
| Worker handler not destructuring `tenantId` from `job.data`. | Worker template scaffold + lint rule. |
| Stripe call without `Idempotency-Key`. | Typed Stripe client refuses to send without it. |
| Inbound webhook handler doing business logic synchronously. | `add-webhook-receiver` template enforces async; handler does verify+persist+enqueue only. |
| Two outbox rows per step. | DB unique on `(event_id, destination)`; second insert errors. |
| Replaying a "failed" outbox row that already partially succeeded downstream. | Vendor-side idempotency catches it; our `last_error` records the original outcome. |

---

## 11. Testing checklist

For any PR that introduces a new mutating route, worker, or
integration:

- [ ] Happy-path test: single call → success.
- [ ] Idempotency replay test: two calls with same key → one
      side-effect, two identical responses (second has
      `Idempotent-Replay: true`).
- [ ] Conflict test: same key, different body → `409`.
- [ ] In-flight test: concurrent calls with same key → one wins,
      other gets `IDEMPOTENCY_KEY_IN_FLIGHT`.
- [ ] Crash mid-way test: process killed after side-effect emitted
      but before commit → next retry produces no duplicate.
- [ ] Outbox dedup test: writing the same `(event_id, destination)`
      twice → second insert no-ops.
- [ ] Vendor 409 test: simulate vendor reporting "already processed"
      → handler treats as success.

---

## 12. References

- [`conventions.md`](./conventions.md) — API conventions including
  headers and response shape.
- [`docs/architecture/multi-tenancy.md`](../architecture/multi-tenancy.md).
- `add-api-route`, `add-workflow-node`, `add-webhook-receiver`,
  `add-integration`, `data-backfill-job`, `add-cron-job`,
  `add-email-template` skills.
- Plan §22, §27, §31, §49.
