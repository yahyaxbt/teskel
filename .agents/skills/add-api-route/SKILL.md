---
name: add-api-route
description: Add a Hono API route in apps/api with Zod input/output schemas, auth + RBAC, RLS-aware DB access, idempotency, structured logs, OpenTelemetry traces, and OpenAPI docs. Use whenever the SDK, Studio, or external integrations need a new endpoint.
---

# add-api-route — add a Hono API route

> Plan ref: [Sec. 18 (API)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#18-api-platform--hono--rpc),
> [Sec. 21 (Auth)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#21-authentication--better-auth),
> [Sec. 22 (RLS)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#22-multi-tenancy--rls),
> [Sec. 26 (Idempotency)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#26-idempotency-retry-and-dlq),
> [Sec. 39 (Observability)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#39-observability-stack),
> [Sec. 50 (RBAC)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#50-rbac-matrix).

## Decide the route surface

| Surface | Path prefix | Auth |
| --- | --- | --- |
| Web UI (RSC + Server Action) | (no API needed unless reusable) | session |
| Public REST | `apps/api/src/routes/v1/...` | API key or session |
| Internal RPC | `apps/api/src/routes/internal/...` | service token |
| Webhooks (inbound) | `apps/api/src/routes/webhooks/...` | signature verify |

Default: REST under `v1` with versioning. Use RPC only when called by
our own SDK and a stable JSON shape is unnecessary.

## Steps

1. **Pick verb + path.** Resource-oriented (`POST /v1/orgs/:orgId/projects`,
   not `POST /v1/createProject`). Plural collection nouns. No verbs in
   path except for explicit actions (`POST /v1/.../actions/cancel`).

2. **Define Zod schemas** in `packages/shared/schemas/projects.ts`:

   ```ts
   import { z } from 'zod';

   export const CreateProjectInput = z.object({
     name: z.string().min(1).max(80),
     description: z.string().max(500).optional(),
     visibility: z.enum(['private', 'workspace']).default('private'),
   });
   export type CreateProjectInput = z.infer<typeof CreateProjectInput>;

   export const ProjectOutput = z.object({
     id: z.string().uuid(),
     orgId: z.string().uuid(),
     name: z.string(),
     description: z.string().nullable(),
     visibility: z.enum(['private', 'workspace']),
     createdAt: z.string().datetime(),
     updatedAt: z.string().datetime(),
   });
   export type ProjectOutput = z.infer<typeof ProjectOutput>;
   ```

3. **Implement the route** with the standard middleware stack
   (`auth` → `rbac` → `idempotency` → handler). Use `zValidator` from
   `@hono/zod-validator` and `zod-openapi` for spec generation.

   ```ts
   // apps/api/src/routes/v1/projects.ts
   import { OpenAPIHono, createRoute } from '@hono/zod-openapi';
   import { auth } from '../../middleware/auth';
   import { rbac } from '../../middleware/rbac';
   import { idempotency } from '../../middleware/idempotency';
   import { db, schema } from '@teskel/db';
   import { CreateProjectInput, ProjectOutput } from '@teskel/shared/schemas/projects';
   import { logger } from '@teskel/shared/logger';
   import { trace } from '@opentelemetry/api';

   export const projectsRouter = new OpenAPIHono();

   const createProjectRoute = createRoute({
     method: 'post',
     path: '/orgs/{orgId}/projects',
     summary: 'Create a project in an organization',
     tags: ['projects'],
     request: {
       params: z.object({ orgId: z.string().uuid() }),
       body: { content: { 'application/json': { schema: CreateProjectInput } } },
     },
     responses: {
       201: { description: 'Created', content: { 'application/json': { schema: ProjectOutput } } },
       400: { description: 'Invalid input' },
       401: { description: 'Unauthenticated' },
       403: { description: 'Forbidden' },
       409: { description: 'Duplicate (idempotent replay with conflicting payload)' },
       422: { description: 'Validation error' },
     },
   });

   projectsRouter.openapi(createProjectRoute,
     auth(),
     rbac({ permission: 'project:create' }),
     idempotency({ ttlSeconds: 24 * 60 * 60 }),
     async (c) => {
       const tracer = trace.getTracer('api');
       return tracer.startActiveSpan('projects.create', async (span) => {
         try {
           const { orgId } = c.req.valid('param');
           const input = c.req.valid('json');
           const { user } = c.get('session');

           const created = await db.withTenant(orgId).transaction(async (tx) => {
             const [row] = await tx.insert(schema.projects).values({
               orgId, name: input.name, description: input.description ?? null,
               visibility: input.visibility, createdBy: user.id,
             }).returning();

             await tx.insert(schema.outboxEvents).values({
               topic: 'project.created',
               payload: { projectId: row.id, orgId, by: user.id },
             });
             return row;
           });

           logger.info({ orgId, projectId: created.id, userId: user.id }, 'project_created');
           span.setAttributes({ 'org.id': orgId, 'project.id': created.id });
           return c.json(ProjectOutput.parse({ ...created, createdAt: created.createdAt.toISOString(), updatedAt: created.updatedAt.toISOString() }), 201);
         } finally {
           span.end();
         }
       });
     },
   );
   ```

4. **Mount the router.**

   ```ts
   // apps/api/src/app.ts
   import { projectsRouter } from './routes/v1/projects';
   app.route('/v1', projectsRouter);
   ```

5. **Idempotency.** All non-idempotent (POST/PATCH/DELETE) routes must
   accept `Idempotency-Key` header. The middleware persists
   `(orgId, route, idemKey, requestHashSha256)` in `idempotency_keys`
   table; replays return the stored response. Conflicting payload
   under the same key returns 409.

6. **Pagination.** Lists use cursor-based pagination
   (`?cursor=opaque&limit=50`, max 200). Never `OFFSET`. Return
   `{ data, nextCursor }`.

7. **Errors.** Standardize on RFC 7807 problem detail JSON:

   ```json
   { "type": "https://teskel.dev/errors/invalid-input", "title": "Invalid input", "status": 400, "detail": "name: too short", "code": "invalid_input", "traceId": "..." }
   ```

   Helper: `apiError(c, { status, code, detail })`. Always include
   `traceId` in response so users can attach it to support tickets.

8. **Logs.** Structured (`pino`) with at minimum
   `traceId`, `userId`, `orgId`, `route`, `latencyMs`, `outcome`.
   Never log secrets or full request bodies.

9. **Traces.** OTel auto-instrumentation handles HTTP + Postgres; add
   custom spans only for internal segments worth measuring (LLM call,
   sandbox exec, third-party API). Tag with `org.id`, `project.id`,
   `user.id` (no PII beyond IDs).

10. **Tests.**
    - Unit: handler with mocked db + middleware (Vitest).
    - Integration: full app with `Testcontainers` Postgres + supertest:
      - Happy path 201.
      - Missing auth → 401.
      - Wrong RBAC → 403.
      - Idempotent replay → same 201, no duplicate row.
      - RLS: user from another org cannot read/write.
    - Contract: snapshot the OpenAPI spec
      (`pnpm --filter @teskel/api openapi:check`) so PRs that change
      the public surface require explicit approval.

11. **SDK update.** Run `pnpm --filter @teskel/sdk generate` to
    regenerate the typed client from OpenAPI. Commit the regenerated
    client in the same PR.

12. **Docs.** API reference is auto-generated from OpenAPI; add a
    short prose page under `apps/docs/content/api/<resource>.mdx` if
    the resource is new.

13. **Telemetry events.** If the route triggers a meaningful business
    event, add it to `packages/shared/analytics/events.ts` and emit
    via PostHog (server-side) or via the outbox event (preferred).

14. **Rate limit.** Default `100/min` per user-IP-route. Tune in
    `apps/api/src/middleware/ratelimit.ts` only with justification;
    add an entry to `docs/runbooks/rate-limits.md`.

## Pitfalls

- Direct `db` access without `withTenant(orgId)` — RLS will block in
  prod, but tests on a superuser connection won't catch it. Always
  go through `withTenant`.
- Returning Drizzle row objects directly without `Output.parse()` —
  leaks columns, drifts from the contract.
- Mutating + side-effect (email/webhook/payment) in one transaction
  inline — use the outbox pattern (insert event row in same tx; worker
  emits).
- Hard-coded HTTP status numbers — use `c.json(body, 201)` etc., never
  string status text.
- Logging the full request body — likely contains PII or secrets.
- Forgetting to mount the router in `app.ts` — endpoint silently 404s.
- Bumping a route shape without versioning — break the contract under
  `/v2` instead of `/v1`.

## Done when

- Schemas defined in `packages/shared/schemas/`.
- Route registered with `auth + rbac + idempotency + handler` stack.
- OpenAPI spec updated; SDK regenerated.
- Unit + integration tests green; RLS test included.
- Logs + traces verified in dev (Grafana + Tempo).
- PR template's "Architecture compliance" checklist all checked.
