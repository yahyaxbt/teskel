# API conventions

> **Status:** Stable. Required reading before opening a PR that
> adds or changes a public API route.
>
> **Governed by:** Plan §22 (API), §50 (data protection),
> [`AGENTS.md` §8](../../AGENTS.md#8-architecture-hard-constraints).
> Operationalized by `add-api-route` skill.

This document defines the conventions every public API route MUST
follow. Internal-only services have looser conventions but should
align where it doesn't cost anything.

---

## 1. URL shape

```text
https://api.teskel.app/{version}/{resource}[/{id}[/{sub-resource}[/{sub-id}]]]
```

- `{version}` — `v1`, `v2`, … (semver MAJOR; see
  [`docs/release/README.md` §3](../release/README.md#3-versioning)).
- `{resource}` — plural noun, kebab-case: `workflows`,
  `template-listings`, `audit-events`.
- `{id}` — UUID v7 unless documented otherwise.
- Sub-resources nest at most **two** levels. Deeper nesting → use a
  query parameter or a separate endpoint.

Examples:

- `GET  /v1/workflows`
- `POST /v1/workflows`
- `GET  /v1/workflows/{id}`
- `POST /v1/workflows/{id}/runs`
- `GET  /v1/workflows/{id}/runs/{runId}`

Forbidden:

- `/api/...` prefix (gateway already strips host).
- Verbs in the path (`/createWorkflow` → `POST /workflows`).
- Query strings to identify the resource (`/workflows?id=…`).
- Mixed case / snake_case in the URL.

---

## 2. Verbs

| Verb | Purpose | Idempotency | Body? |
| --- | --- | --- | --- |
| `GET` | Read | inherently | no |
| `HEAD` | Existence / metadata | inherently | no |
| `POST` | Create or trigger | **declare via `Idempotency-Key`** (see [`idempotency.md`](./idempotency.md)) | yes |
| `PUT` | Replace whole resource | yes | yes |
| `PATCH` | Partial update | declare via `Idempotency-Key` | yes |
| `DELETE` | Remove | yes | optional (rare) |

We do **not** use the "RPC" verb-on-path style (`/workflows/run`)
except for explicitly action-shaped sub-resources (`/{id}/cancel`,
`/{id}/promote`).

---

## 3. Request

### 3.1 Headers (required where listed)

| Header | When | Notes |
| --- | --- | --- |
| `Authorization: Bearer <jwt or api-key>` | Always for non-anonymous | API keys hashed at rest. |
| `Content-Type: application/json; charset=utf-8` | All bodies | reject other types with `415`. |
| `Accept: application/json` | All | `406` if unmet. |
| `Idempotency-Key: <ulid \| uuid>` | All non-`GET` mutations | See [`idempotency.md`](./idempotency.md). |
| `If-Match: "<etag>"` | Conditional updates | `412` on stale. |
| `Prefer: return=minimal` | Optional | response is `204` instead of full body. |
| `X-Tenant-Id: <uuid>` | When the caller is a member of multiple orgs and the route doesn't otherwise scope | server validates membership; mismatch ⇒ `403`. |

### 3.2 Body

- JSON only. UTF-8.
- Property names: `camelCase`.
- IDs: UUID v7 string.
- Money: integer minor units + currency code (`{ "amount": 1000, "currency": "usd" }` for $10.00).
- Time: RFC 3339 / ISO 8601 with `Z` (`"2026-05-08T02:13:00Z"`).
- Enums: lowercase string with hyphens (`"in-progress"`).
- Booleans: only as `true`/`false`; never `0`/`1`.

Validation by **Zod** at the handler boundary; reject with `400` on
schema violation.

### 3.3 Query parameters

| Parameter | Convention |
| --- | --- |
| `page[size]` / `page[after]` | cursor-based pagination (see §5). Use brackets to distinguish from filters. |
| `filter[<field>]` | exact-match filter; multiple → AND. |
| `sort` | comma-separated; prefix `-` for descending (`sort=-createdAt,name`). |
| `include` | comma-separated relations to include in response. |
| `fields` | comma-separated field allowlist (sparse fieldsets). |
| `q` | free-text search where supported. |

Other params are route-specific and documented in the OpenAPI spec.

---

## 4. Response

### 4.1 Success body

```json
{
  "data": { ... },
  "meta": { "pagination": { "next": "...", "total": 42 } },
  "links": { "self": "..." }
}
```

- `data`: the resource (object) or list (array).
- `meta`: pagination, counts, deprecation warnings.
- `links`: HATEOAS-style next/prev/self where it pays off.

For collection endpoints, `data` is an array; pagination metadata in
`meta.pagination`.

For empty collection: `data: []`, status `200`.

For `204 No Content`: empty body.

### 4.2 Error body — RFC 9457 Problem Details

```json
{
  "type": "https://errors.teskel.app/validation",
  "title": "Validation failed",
  "status": 400,
  "detail": "Field `name` must be 1–80 chars.",
  "instance": "/v1/workflows",
  "code": "VALIDATION_ERROR",
  "errors": [
    { "field": "name", "code": "TOO_SHORT", "minLength": 1 }
  ],
  "requestId": "req_01J..."
}
```

- `type` — stable URL identifying the error class.
- `code` — stable machine-readable code; **never reuse**.
- `errors` — array, each entry has `field` + `code` + structured args.
- `requestId` — present on all responses (success and error).

Error codes are listed in `apps/api/src/errors/codes.ts` and
documented in the OpenAPI spec.

### 4.3 Status code matrix

| Code | When |
| --- | --- |
| `200 OK` | Successful read or update with body. |
| `201 Created` | Successful create; `Location` header set. |
| `202 Accepted` | Async work started; `Location` to status endpoint. |
| `204 No Content` | Successful op, no body (delete, `Prefer: return=minimal`). |
| `400 Bad Request` | Schema violation; client must fix request. |
| `401 Unauthorized` | Missing or invalid auth. |
| `403 Forbidden` | Auth ok, lacks permission. |
| `404 Not Found` | Resource not found *or* not visible to this tenant (do not leak existence). |
| `405 Method Not Allowed` | Wrong verb on resource. |
| `406 Not Acceptable` | `Accept` header unmet. |
| `409 Conflict` | Idempotency key reused with different body, or optimistic concurrency. |
| `410 Gone` | Resource permanently deleted. |
| `412 Precondition Failed` | `If-Match` stale. |
| `415 Unsupported Media Type` | Wrong `Content-Type`. |
| `422 Unprocessable Entity` | Semantically invalid (rare; prefer `400`). |
| `429 Too Many Requests` | Rate limit; `Retry-After` header set. |
| `451 Unavailable For Legal Reasons` | DSAR / takedown / region restriction. |
| `500 Internal Server Error` | Server bug; `requestId` lets ops trace. |
| `502/503/504` | Upstream issue (LLM, DB) — clients should retry with backoff. |

---

## 5. Pagination

**Cursor-based**, always. Offset pagination is forbidden — it breaks
under concurrent writes and is O(N) on large tables.

Request:

```text
GET /v1/workflows?page[size]=50&page[after]=eyJpZCI6...
```

Response:

```json
{
  "data": [ ... ],
  "meta": {
    "pagination": {
      "next": "eyJpZCI6...",
      "size": 50
    }
  }
}
```

- `page[size]` — default 50, max 200.
- `page[after]` — opaque cursor; treat as encoding of `(sort_key,
  primary_key)` and never expose internals.
- `meta.pagination.next` is `null` on the last page.
- Cursor lifetime: **1 h** (encoded TTL); a client that pauses longer
  must restart.

---

## 6. Filtering, sorting, sparse fieldsets

```text
GET /v1/workflows?filter[status]=active&filter[ownerId]=...&sort=-updatedAt&fields=id,name,status
```

- Each filter is exact-match. For ranges/operators, expose explicit
  fields (`filter[createdAfter]=2026-01-01`).
- `sort` is multi-key; default per resource defined in the OpenAPI.
- `fields` reduces server work; the response still wraps in
  `{ data, meta }`.
- `include` for relation expansion (avoid N+1 calls). Cap depth at 2.

---

## 7. Auth

| Method | Where | Use case |
| --- | --- | --- |
| Better Auth session cookie | First-party web | App-served browser sessions. |
| Bearer JWT (Better Auth-issued) | First-party web / mobile | SPA outside cookie scope. |
| Bearer API key (`teskel_<env>_<rand>`) | Server-to-server / SDK | Workspace-scoped, hashed at rest. |
| OAuth2 (Phase 2+) | Third-party apps | Authorization Code + PKCE. |
| SAML / SCIM (Phase 4+) | Enterprise SSO | via Better Auth. |

API keys:

- Format: `teskel_<env>_<28-char base32>`. `env` ∈ `live`, `test`.
- Hashed with argon2id; the secret is shown **once** at creation.
- Scoped to a single workspace + a permission set.
- Revocable; deletion takes effect within 60 s globally.
- Last-used timestamp tracked.

---

## 8. Multi-tenancy

Every authenticated route resolves a **tenant context** before any
DB call. The handler **must** call `db.withTenant(tenantId, …)`. See
[`docs/architecture/multi-tenancy.md`](../architecture/multi-tenancy.md).

Cross-tenant operations are listed there explicitly. New cross-
tenant operation = **ADR**, not a route PR.

---

## 9. Rate limiting

| Tier | Default |
| --- | --- |
| Anonymous | 60 rpm per IP |
| Authenticated user | 600 rpm per user |
| API key (Starter) | 100 rps; 10k rpm |
| API key (Pro) | 500 rps; 50k rpm |
| API key (Business+) | 2k rps; 200k rpm |
| LLM-bearing routes | tighter — see slot budget |

Headers on every response:

- `X-RateLimit-Limit: <window-max>`
- `X-RateLimit-Remaining: <integer>`
- `X-RateLimit-Reset: <epoch-seconds>`
- `Retry-After: <seconds>` (only on `429`)

Per-tenant burst tokens stored in Redis token-bucket; backoff
follows AWS-style exponential with jitter.

---

## 10. Versioning & deprecation

- Public API versions live forever in parallel — minimum **12-month
  overlap** for any breaking change.
- Deprecated routes return `Deprecation: true` and `Sunset: <date>`
  headers (RFC 8594).
- The OpenAPI spec marks deprecated routes; SDK regenerates with
  warnings.
- A breaking change requires an ADR and a customer email.

---

## 11. Telemetry

Every handler emits:

- A span `api.handle` with attributes:
  `route` (template, not the URL), `tenant_id`, `user_id`,
  `idempotency_key`, `http_status`.
- A counter `api_requests_total{route, status}`.
- A histogram `api_request_seconds{route}`.
- An audit-log row for any non-`GET` (per
  [`docs/security/rbac-matrix.md`](../security/rbac-matrix.md)).

PII never lands in logs. The logger redacts `email`, `phone`,
`firstName`, `lastName`, raw bodies of webhooks, and prompt input.

---

## 12. OpenAPI

We **generate** the OpenAPI spec from Zod schemas (`zod-to-openapi`).
The spec is committed at `apps/api/openapi.yml` so reviewers see the
diff. The spec is the source for:

- The TypeScript SDK (`packages/sdk`).
- The CLI (`packages/cli`).
- The docs site reference page.
- Postman / Insomnia exports.

Hand-editing `openapi.yml` is forbidden — the diff in CI rejects it.

---

## 13. Webhooks (outbound)

When TESKEL **sends** a webhook to a customer-controlled URL:

- Payload is JSON, same shape as API responses (`data`, `meta`).
- Signed with HMAC-SHA-256 over the raw body using a per-tenant
  signing secret (rotated via `rotate-secret`).
- Headers: `X-Teskel-Event`, `X-Teskel-Delivery`,
  `X-Teskel-Signature`, `X-Teskel-Signature-Timestamp`.
- Retry policy: 0, 1m, 5m, 30m, 2h, 12h, then DLQ. Customer can
  manually replay from the dashboard.
- Customer endpoint must respond `2xx` within **5 s** to count as
  delivered.

Inbound webhooks (TESKEL receiving from a vendor) follow
[`add-webhook-receiver`](../../.agents/skills/add-webhook-receiver/SKILL.md).

---

## 14. Examples (illustrative)

### 14.1 Create a workflow

```http
POST /v1/workflows HTTP/1.1
Host: api.teskel.app
Authorization: Bearer teskel_live_abc...
Content-Type: application/json
Idempotency-Key: 01J9...

{
  "name": "Lead scorer",
  "description": "Scores inbound leads via LLM."
}
```

```http
HTTP/1.1 201 Created
Location: /v1/workflows/01J9X...
Content-Type: application/json; charset=utf-8

{
  "data": {
    "id": "01J9X...",
    "name": "Lead scorer",
    "status": "draft",
    "createdAt": "2026-05-08T09:14:00Z"
  },
  "links": { "self": "/v1/workflows/01J9X..." }
}
```

### 14.2 Validation error

```http
HTTP/1.1 400 Bad Request
Content-Type: application/problem+json; charset=utf-8

{
  "type": "https://errors.teskel.app/validation",
  "title": "Validation failed",
  "status": 400,
  "code": "VALIDATION_ERROR",
  "errors": [
    { "field": "name", "code": "TOO_SHORT", "minLength": 1 }
  ],
  "requestId": "req_01J..."
}
```

### 14.3 Idempotent replay

```http
POST /v1/workflows HTTP/1.1
Idempotency-Key: 01J9...
{ "name": "Lead scorer" }

→ 201 Created (first time)

POST /v1/workflows HTTP/1.1
Idempotency-Key: 01J9...     # same key, same body
{ "name": "Lead scorer" }

→ 200 OK (replay; same response body, header `Idempotent-Replay: true`)

POST /v1/workflows HTTP/1.1
Idempotency-Key: 01J9...     # same key, DIFFERENT body
{ "name": "Lead scorer v2" }

→ 409 Conflict
{ "code": "IDEMPOTENCY_KEY_CONFLICT", ... }
```

---

## 15. References

- [`idempotency.md`](./idempotency.md) — idempotency contract.
- [`docs/architecture/multi-tenancy.md`](../architecture/multi-tenancy.md).
- [`docs/security/rbac-matrix.md`](../security/rbac-matrix.md).
- `add-api-route` skill.
- Plan §22, §50.
