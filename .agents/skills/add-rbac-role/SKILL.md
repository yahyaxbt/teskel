---
name: add-rbac-role
description: Add or modify a RBAC role/permission in the TESKEL authorization model. Updates the permission registry, role-permission map, Better Auth org plugin config, RLS policies (if needed), API rbac middleware, UI gating, audit log events, and tests. Use whenever a new capability needs gated access.
---

# add-rbac-role — modify the authorization model

> Plan ref: [Sec. 50 (RBAC matrix)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#50-rbac-matrix),
> [Sec. 21 (Auth)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#21-authentication--better-auth),
> [Sec. 22 (RLS)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#22-multi-tenancy--rls),
> [Sec. 51 (Audit log)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#51-audit-log).

> **Hard rule:** roles are coarse-grained; permissions are fine. Add
> new permissions to existing roles before inventing a new role.
> Inventing roles requires an ADR.

## Permission registry

Single source of truth: `packages/auth/src/permissions.ts`.

Permission ids: `<resource>:<action>`. Actions are CRUD-ish + a small
set of domain verbs (`approve`, `publish`, `invite`, `transfer`,
`impersonate`, `read:billing`).

```ts
// packages/auth/src/permissions.ts
export const permissions = {
  // org
  'org:read': 'Read organization profile and members',
  'org:update': 'Update organization profile',
  'org:delete': 'Delete the organization',
  'org:billing:read': 'Read billing portal and invoices',
  'org:billing:manage': 'Change plan, payment method, manage seats',

  // member
  'member:invite': 'Invite a new member to the org',
  'member:remove': 'Remove a member from the org',
  'member:role:assign': 'Change a member’s role',

  // project
  'project:read': 'Read project metadata and contents',
  'project:create': 'Create a new project',
  'project:update': 'Edit project metadata',
  'project:delete': 'Delete a project',

  // workflow
  'workflow:read': 'Read workflow definitions',
  'workflow:run': 'Trigger workflow execution',
  'workflow:edit': 'Edit workflow definitions',
  'workflow:publish': 'Publish workflow to production',

  // marketplace
  'marketplace.listing:read': 'Read marketplace listings',
  'marketplace.listing:publish': 'Publish a listing',
  'marketplace.listing:moderate': 'Approve/reject moderation queue',

  // billing
  'billing:refund': 'Issue refunds',
  'billing:metering:read': 'Read metering data',

  // platform admin (internal)
  'admin:impersonate': 'Impersonate any user',
  'admin:audit:read': 'Read full audit log',
} as const;

export type PermissionId = keyof typeof permissions;
```

## Role registry

```ts
// packages/auth/src/roles.ts
import type { PermissionId } from './permissions';

export const roles = {
  owner:   { label: 'Owner',   permissions: 'all' as const },
  admin:   { label: 'Admin',   permissions: [
    'org:read','org:update','member:invite','member:remove','member:role:assign',
    'project:read','project:create','project:update','project:delete',
    'workflow:read','workflow:run','workflow:edit','workflow:publish',
    'marketplace.listing:read','marketplace.listing:publish',
    'org:billing:read','org:billing:manage',
  ] satisfies PermissionId[] },
  developer: { label: 'Developer', permissions: [
    'org:read','project:read','project:create','project:update',
    'workflow:read','workflow:run','workflow:edit','workflow:publish',
    'marketplace.listing:read',
  ] satisfies PermissionId[] },
  member:    { label: 'Member', permissions: [
    'org:read','project:read','workflow:read','workflow:run',
    'marketplace.listing:read',
  ] satisfies PermissionId[] },
  billing:   { label: 'Billing', permissions: [
    'org:read','org:billing:read','org:billing:manage',
  ] satisfies PermissionId[] },
  guest:     { label: 'Guest', permissions: [
    'org:read','project:read','workflow:read',
  ] satisfies PermissionId[] },
} as const;
```

## Steps

1. **Decide: new permission, new role, or both?**
   - 90% of the time: add a new permission to existing roles.
   - New role: requires an ADR (`docs/adr/00xx-rbac-role-<name>.md`)
     because role count affects the UI, billing, support, and audit
     analysis.

2. **Add the permission** to `permissions.ts`. Keep alphabetical order
   inside each domain block.

3. **Map permission to roles** in `roles.ts`. Owner always has all
   permissions (special-cased). Admin has most. Developer/Member is
   the principle of least privilege.

4. **Sync Better Auth org plugin.** Better Auth's organization plugin
   takes a static role config; regenerate from our registry:

   ```bash
   pnpm --filter @teskel/auth gen:roles
   ```

   This emits `apps/api/src/auth/org-roles.generated.ts` consumed by
   Better Auth.

5. **Wire the API rbac middleware.** Already generic — new permissions
   are recognized automatically. In a route:

   ```ts
   .use(rbac({ permission: 'project:create' }))
   ```

   For composite checks:

   ```ts
   .use(rbac({ anyOf: ['project:update','workflow:edit'] }))
   ```

6. **Update RLS policies** if the permission affects DB access in
   ways not covered by tenant isolation alone.

   Example: `marketplace.listing:moderate` allows reading listings
   from other orgs in the moderation queue. Add a Postgres policy:

   ```sql
   CREATE POLICY "moderators read all listings"
     ON marketplace_listings
     FOR SELECT
     USING (
       current_setting('app.permissions', true) ILIKE '%marketplace.listing:moderate%'
     );
   ```

   See `add-table` skill for how `app.org_id` and
   `app.permissions` settings are populated by `db.withTenant()`.

7. **Gate UI components.** Add permission-aware components in
   `packages/ui/`:

   ```tsx
   <PermissionGate permission="workflow:publish" fallback={null}>
     <PublishButton />
   </PermissionGate>
   ```

   Server components use the helper `await canPerform('workflow:publish', ctx)`.

8. **Audit log.** Sensitive actions (anything ending in `:moderate`,
   `:delete`, `member:role:assign`, `billing:*`, `admin:*`) emit an
   audit entry:

   ```ts
   audit.log({ actor: user.id, orgId, action: 'member.role.assigned', target: targetUserId, metadata: { from: 'developer', to: 'admin' }});
   ```

   See `add-table` skill — audit table uses a hash chain.

9. **Tests.**
   - Unit: `hasPermission(role, perm)` returns expected booleans for
     every (role, permission) cell.
   - Integration: API routes return 403 when the role lacks the
     permission, 200 when it has it.
   - RLS: a developer cannot read a moderator-only row, a moderator
     can.
   - UI: `<PermissionGate>` hides children for unauthorized roles.

10. **Documentation.** Update the matrix doc
    `docs/security/rbac-matrix.md`. Re-run
    `pnpm --filter @teskel/auth gen:matrix-md` to regenerate the
    table from the registry. Commit the updated markdown.

11. **Migration considerations.**
    - If a permission previously implicit in a role is now split out:
      audit existing role assignments and migrate as needed.
    - If you remove a permission from a role: feature-flag the change
      and notify affected orgs.
    - Never silently downgrade a customer role; raise a
      `member.role.downgraded` audit event.

## Pitfalls

- Inventing roles instead of permissions — leads to role explosion and
  customer confusion. Add permissions first.
- Hard-coding permission strings in product code — always import from
  `permissions.ts`.
- Forgetting to update RLS policies when a permission affects
  cross-tenant or cross-row reads — leads to silent over-fetching at
  the API layer that RLS would have stopped.
- UI gates without API gates — anyone with a browser can bypass UI.
  Both layers required.
- Mutating `roles.ts` for one customer’s edge case — talk to product
  first; either it’s a permission for everyone or it’s a separate
  concept (ABAC, sharing, etc.).
- Omitting audit log for sensitive actions — fails compliance reviews.

## Done when

- Permission registered + mapped to appropriate roles.
- Better Auth roles regenerated.
- API rbac middleware wired.
- RLS policy added/updated if needed.
- UI gates in place.
- Audit log raised for sensitive actions.
- Tests cover happy + denied + cross-tenant cases.
- Matrix doc regenerated.
