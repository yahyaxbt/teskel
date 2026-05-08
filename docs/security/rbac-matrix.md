# RBAC matrix

Source-of-truth permissions × roles × surfaces map. Used by the
[`add-rbac-role`](../../.agents/skills/add-rbac-role/SKILL.md) skill
and enforced at three layers: Better Auth middleware, RLS policies,
UI gates.

> **Hard rule:** every permission in the codebase is registered here.
> A permission key referenced in code that's not in this file is a
> CI lint failure. The matrix is the contract; code conforms.

## Permission key format

`<resource>.<verb>[.<scope>]`

- `resource` — domain noun (`org`, `project`, `template`, `member`,
  `billing`, `audit_log`, `secrets`).
- `verb` — `read`, `create`, `update`, `delete`, `publish`, `invite`,
  `assign`, `export`, `restore`, `rotate`, `impersonate`.
- `scope` — optional sub-scope (`org.member.read.self` vs
  `org.member.read.all`).

## Roles

### System roles (built-in, immutable)

| Role | Description |
| --- | --- |
| `system.platform_admin` | TESKEL employee with full operational access. Auditable; high-friction MFA required. Limited count, listed in `docs/security/admins.md`. |
| `system.support_l1` | Customer support tier 1 — read-only across tenants for ticket triage. Cannot impersonate. |
| `system.support_l2` | Customer support tier 2 — can impersonate with customer consent token; subject to audit log every action. |
| `system.billing_admin` | Finance ops; access to billing tables only. |
| `system.security_lead` | Audit log access, RBAC modification, secret rotation orchestration. |
| `system.read_only` | Engineer with read-only prod access (debugging). |

### Tenant roles (configurable per org)

| Role | Description | Defaults |
| --- | --- | --- |
| `org.owner` | Created the org. Single user. Cannot be removed without ownership transfer. | All permissions in tenant. |
| `org.admin` | Full tenant control except billing change. | All except `billing.update`. |
| `org.member` | Default human contributor. | Project read/write; template read; no admin. |
| `org.viewer` | Read-only contributor. | Project read; template read. |
| `org.guest` | External collaborator on specific projects only. | Per-project assigned; no org-level read. |
| `org.api` | Service account / programmatic access. | Scoped permissions per API key. |
| `org.billing_contact` | Receives billing emails; can view invoices. | `billing.read`. |

Role definitions are stored in `org.role_definitions` table; system
roles are seed data, tenant roles are editable by `org.owner` /
`org.admin` within an allow-list of safe permissions.

## Permissions registry

> Populated as features ship. Each row is added by the PR that
> introduces the feature, via the
> [`add-rbac-role`](../../.agents/skills/add-rbac-role/SKILL.md)
> skill.

### org.*

| Permission | system.platform_admin | system.support_l2 | org.owner | org.admin | org.member | org.viewer | org.guest |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `org.read.self` | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| `org.update.self` | ✓ | — | ✓ | ✓ | — | — | — |
| `org.delete.self` | ✓ (with consent) | — | ✓ | — | — | — | — |
| `org.member.read` | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | — |
| `org.member.invite` | ✓ | — | ✓ | ✓ | — | — | — |
| `org.member.remove` | ✓ | — | ✓ | ✓ | — | — | — |
| `org.member.assign_role` | ✓ | — | ✓ | ✓ | — | — | — |

### project.*

| Permission | org.owner | org.admin | org.member | org.viewer | org.guest |
| --- | --- | --- | --- | --- | --- |
| `project.read` | ✓ | ✓ | ✓ | ✓ | per-assignment |
| `project.create` | ✓ | ✓ | ✓ | — | — |
| `project.update` | ✓ | ✓ | ✓ | — | — |
| `project.delete` | ✓ | ✓ | — | — | — |
| `project.publish` | ✓ | ✓ | ✓ | — | — |

### template.*

| Permission | org.owner | org.admin | org.member | org.viewer |
| --- | --- | --- | --- | --- |
| `template.read.installed` | ✓ | ✓ | ✓ | ✓ |
| `template.install` | ✓ | ✓ | ✓ | — |
| `template.create` | ✓ | ✓ | ✓ | — |
| `template.publish_to_marketplace` | ✓ | ✓ | — | — |
| `template.update.own` | ✓ | ✓ | ✓ | — |
| `template.delete.own` | ✓ | ✓ | ✓ | — |

### billing.*

| Permission | org.owner | org.admin | org.billing_contact | org.member |
| --- | --- | --- | --- | --- |
| `billing.read` | ✓ | ✓ | ✓ | — |
| `billing.update_payment_method` | ✓ | — | — | — |
| `billing.change_plan` | ✓ | — | — | — |
| `billing.invoice.download` | ✓ | ✓ | ✓ | — |

### audit_log.*

| Permission | system.security_lead | system.platform_admin | org.owner | org.admin |
| --- | --- | --- | --- | --- |
| `audit_log.read.platform` | ✓ | ✓ | — | — |
| `audit_log.read.tenant` | ✓ | ✓ | ✓ | ✓ |
| `audit_log.export.tenant` | ✓ | — | ✓ | — |

### secrets.*

| Permission | system.security_lead | system.platform_admin | org.owner |
| --- | --- | --- | --- |
| `secrets.rotate.platform` | ✓ | ✓ | — |
| `secrets.rotate.tenant_api_key` | — | — | ✓ |
| `secrets.read.metadata` | ✓ | ✓ | ✓ |

### support.* (system-only)

| Permission | system.support_l1 | system.support_l2 | system.platform_admin |
| --- | --- | --- | --- |
| `support.tenant.read.metadata` | ✓ | ✓ | ✓ |
| `support.tenant.read.content` | — | with consent | ✓ |
| `support.tenant.impersonate` | — | with consent | ✓ |

## Special markers

- `✓` — granted by default.
- `—` — not granted.
- `with consent` — requires customer-issued consent token (in-app
  approval), expires in 24h, every action audit-logged.
- `per-assignment` — granted only when explicitly assigned to a
  project.

## Enforcement layers

Every permission MUST be enforced at all applicable layers:

1. **Better Auth middleware** — coarse: "is this user authenticated +
   in the right org?".
2. **RBAC middleware** (`requirePermission(perm)`) — fine-grained
   permission check before handler runs.
3. **RLS policies** — DB-level: `org_id` matching `auth.org_id()`,
   role policies for cross-row visibility.
4. **UI gates** — surface-level: render gate only if the permission
   is granted; never security-critical (UI gates are UX, not auth).

Skipping any layer is a defect. The
[`add-rbac-role`](../../.agents/skills/add-rbac-role/SKILL.md) skill
walks all four.

## Audit semantics

All RBAC-relevant changes (role grant, permission flip, role
definition edit, consent issued, impersonation start/stop) emit
`audit_events` rows that are hash-chained per phase. Audit reads are
NOT chained but ARE logged (read-of-audit-log is itself audit-
logged).

## Adding a permission

Use the [`add-rbac-role`](../../.agents/skills/add-rbac-role/SKILL.md)
skill. The skill PR:

1. Adds the row to this matrix.
2. Adds the permission key to `packages/auth/permissions.ts`.
3. Updates the role-definition seed for any role that should get the
   permission by default.
4. Adds RLS policies if the permission gates row visibility.
5. Adds middleware enforcement.
6. Adds tests covering both grant and denial paths.

## Removing a permission

Treat as a breaking change for tenant role customizations. Steps:

1. Mark deprecated in this file (strikethrough + ADR reference).
2. Remove from default role grants (still in the registry until
   sunset).
3. After ≥1 release with deprecation warning logged, remove the
   permission key from code; CI lint catches dangling references.

## Related

- Plan §46 (Better Auth) — auth model.
- Plan §47 (RBAC) — matrix design intent.
- Plan §51 (Audit log) — chain integrity.
- [`add-rbac-role`](../../.agents/skills/add-rbac-role/SKILL.md) —
  skill that mutates this file.
- [`gdpr-data-request`](../../.agents/skills/gdpr-data-request/SKILL.md)
  — special "two-person rule" RBAC pattern.
