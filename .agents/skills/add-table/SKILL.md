---
name: add-table
description: Add a new Drizzle table with proper RLS policies, migration, and tenant scoping in TESKEL. Use whenever a feature requires a new persistent entity. Enforces multi-tenant safety and migration reversibility.
---

# add-table — add a Drizzle table with RLS

> Use this skill whenever a feature needs a new persistent entity. RLS
> is non-negotiable for tenant-owned tables — see
> [`AGENTS.md` §8](../../../AGENTS.md#8-architecture-hard-constraints)
> and [Plan Sec. 22](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#22-multi-tenancy--rls).

## Steps

1. **Decide ownership.** Is the table:
   - Tenant-owned (per-org data) — must enforce RLS via
     `org_id = current_setting('app.current_org')::uuid`. Most tables
     fall here.
   - Global / system (e.g., `prompts`, `feature_flags`) — no RLS, but
     restrict writes via API role.
   - User-personal (rare) — scope by `user_id` instead of `org_id`.

2. **Define columns.** Defaults to include on every tenant table:

   ```ts
   import { pgTable, uuid, timestamp } from 'drizzle-orm/pg-core';
   import { sql } from 'drizzle-orm';

   export const widgets = pgTable('widgets', {
     id: uuid('id').primaryKey().defaultRandom(),
     orgId: uuid('org_id').notNull().references(() => orgs.id, { onDelete: 'cascade' }),
     // ...your columns...
     createdAt: timestamp('created_at', { withTimezone: true }).notNull().defaultNow(),
     updatedAt: timestamp('updated_at', { withTimezone: true }).notNull().defaultNow(),
     deletedAt: timestamp('deleted_at', { withTimezone: true }),
   });
   ```

3. **Generate the migration.**

   ```bash
   pnpm --filter @teskel/db drizzle-kit generate
   ```

   Inspect the generated SQL. Edit if needed for indexes (next step).

4. **Add indexes.** Tenant-owned tables almost always need
   `(org_id, created_at desc)` for list queries. Add unique constraints
   that include `org_id`:

   ```sql
   CREATE INDEX widgets_org_created_idx ON widgets (org_id, created_at DESC);
   CREATE UNIQUE INDEX widgets_org_slug_idx ON widgets (org_id, slug);
   ```

5. **Add RLS policies (tenant-owned only).** In the migration:

   ```sql
   ALTER TABLE widgets ENABLE ROW LEVEL SECURITY;

   CREATE POLICY widgets_isolation ON widgets
     USING (org_id = current_setting('app.current_org', true)::uuid);

   CREATE POLICY widgets_insert ON widgets FOR INSERT
     WITH CHECK (org_id = current_setting('app.current_org', true)::uuid);
   ```

   For tables with admin override (e.g., `__admin` impersonation), add a
   second policy gated by a different setting (`app.admin_override`).

6. **Add a soft-delete trigger if the entity should be recoverable.**

   ```sql
   CREATE OR REPLACE FUNCTION soft_delete_widgets() RETURNS trigger AS $$
   BEGIN
     UPDATE widgets SET deleted_at = NOW() WHERE id = OLD.id;
     RETURN NULL;  -- block hard delete
   END $$ LANGUAGE plpgsql;
   ```

   Register a `BEFORE DELETE` trigger. Hard-delete is allowed only via
   a dedicated scheduled job that writes to `audit_log`.

7. **Write the down migration.**
   - `DROP POLICY ... ON widgets;`
   - `ALTER TABLE widgets DISABLE ROW LEVEL SECURITY;`
   - `DROP TABLE widgets;`

   If the table holds important data, the down migration should
   `INSERT INTO archive_widgets SELECT * FROM widgets` first.

8. **Add a tenant-isolation test.** In `packages/db/test/integration/`:

   ```ts
   it('isolates widgets between orgs', async () => {
     const orgA = await seedOrg();
     const orgB = await seedOrg();
     await db.withTenant(orgA.id).insert(widgets).values({ ... });
     const rows = await db.withTenant(orgB.id).select().from(widgets);
     expect(rows).toHaveLength(0);
   });
   ```

   This test must exist and pass for every new tenant-owned table.

9. **Update Drizzle relations** in `packages/db/src/schema/index.ts`
   if other tables FK to this one.

10. **PR conventions.**
    - Title: `feat(db): add widgets table`.
    - PR description includes: migration plan, rollback plan
      (apply down migration), data-loss risk assessment.
    - CODEOWNER for `packages/db/` must approve.

## Pitfalls

- Forgetting RLS on a tenant table — **caught by §8 reviewer**, but you
  should not get there. Default to RLS enabled.
- Using `drizzle-kit push` on prod — **never**. `push` is for dev only.
- Index on `(created_at)` without `org_id` — useless under RLS, slow
  under load.
- Foreign key without `ON DELETE` action — pick `cascade`, `set null`,
  or `restrict` deliberately.
- Forgetting to update `packages/db/src/schema/index.ts` exports.

## Done when

- Migration `up` and `down` both run cleanly on a fresh DB.
- Tenant-isolation test passes.
- `pnpm turbo run lint typecheck test --filter @teskel/db` is green.
- CODEOWNER for `db` approves.
