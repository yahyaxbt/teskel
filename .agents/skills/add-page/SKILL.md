---
name: add-page
description: Add a new Next.js 15 App Router page in apps/web with Server Component default, metadata, layout, loading/error/not-found states, auth + RLS guards, and analytics. Use when adding any user-facing route.
---

# add-page — add a Next.js page

> Plan ref: [Sec. 17 (Frontend platform)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#17-frontend-platform-nextjs-15--rsc),
> [Sec. 13 (Empty/Error/Loading)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#13-empty--error--loading-patterns),
> [Sec. 14 (A11y)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#14-accessibility-wcag-22-aa),
> [Sec. 21 (Auth)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#21-authentication--better-auth),
> [Sec. 22 (RLS)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#22-multi-tenancy--rls).

Default rule: **Server Component first**. Add `'use client'` only when
you need state, refs, or browser APIs.

## Steps

1. **Decide the route.**
   - Public (marketing): `apps/web/src/app/(marketing)/<route>/page.tsx`
   - Authed app: `apps/web/src/app/(app)/<scope>/<route>/page.tsx`
   - Org-scoped: `apps/web/src/app/(app)/o/[orgSlug]/<route>/page.tsx`
   - Project-scoped: `.../o/[orgSlug]/p/[projectSlug]/<route>/page.tsx`

2. **Create the page** with metadata + Server Component fetch:

   ```tsx
   // apps/web/src/app/(app)/o/[orgSlug]/projects/page.tsx
   import { Suspense } from 'react';
   import type { Metadata } from 'next';
   import { redirect } from 'next/navigation';
   import { requireSession } from '@teskel/auth/server';
   import { db } from '@teskel/db';
   import { ProjectsList, ProjectsListSkeleton } from './projects-list';

   export const metadata: Metadata = {
     title: 'Projects · TESKEL',
     description: 'All projects in your organization.',
   };

   export default async function ProjectsPage(props: {
     params: Promise<{ orgSlug: string }>;
   }) {
     const { orgSlug } = await props.params;
     const { user, org } = await requireSession({ orgSlug });
     if (!org) redirect('/onboarding');

     return (
       <main className="container py-8">
         <h1 className="text-2xl font-semibold">Projects</h1>
         <Suspense fallback={<ProjectsListSkeleton />}>
           <ProjectsList orgId={org.id} />
         </Suspense>
       </main>
     );
   }
   ```

3. **Always use `db.withTenant(orgId)` for tenant queries** (RLS).

   ```tsx
   // apps/web/src/app/(app)/o/[orgSlug]/projects/projects-list.tsx
   import { db } from '@teskel/db';
   import { projects } from '@teskel/db/schema';

   export async function ProjectsList({ orgId }: { orgId: string }) {
     const rows = await db.withTenant(orgId).query.projects.findMany({
       orderBy: (t, { desc }) => desc(t.updatedAt),
       limit: 50,
     });
     if (rows.length === 0) return <ProjectsEmpty />;
     return <ul>{rows.map((p) => <li key={p.id}>{p.name}</li>)}</ul>;
   }
   ```

4. **Add the four required state files in the route folder.**

   - `loading.tsx` — uses skeleton primitives (no spinner-only).
   - `error.tsx` — Client Component, logs to Sentry, shows retry.
   - `not-found.tsx` — friendly empty state with a way back.
   - `page.tsx` — the page itself.

   ```tsx
   // error.tsx
   'use client';
   import { useEffect } from 'react';
   import * as Sentry from '@sentry/nextjs';
   export default function Error({ error, reset }: { error: Error & { digest?: string }; reset: () => void }) {
     useEffect(() => Sentry.captureException(error), [error]);
     return (
       <div role="alert" className="container py-8">
         <h2 className="text-xl font-semibold">Something went wrong</h2>
         <p className="text-muted-foreground">{error.digest}</p>
         <button className="btn mt-4" onClick={reset}>Try again</button>
       </div>
     );
   }
   ```

5. **Mutations via Server Actions or the API.**
   - Tiny mutations co-located with the page → Server Action with
     `'use server'`.
   - Anything reusable, complex, or invoked by SDK → call the Hono API
     via `@teskel/sdk`.
   - Both paths must:
     - Re-validate input with Zod (defense in depth).
     - Use `db.withTenant(orgId)` for RLS.
     - Return typed result (no `any`).
     - Accept idempotency-key for non-idempotent ops.

6. **Forms.**
   - React Hook Form + `@hookform/resolvers/zod`.
   - Client validation must match the server schema (re-export from
     `packages/shared/schemas/`).
   - Show inline errors with `aria-invalid` + `aria-describedby`.
   - On submit error, focus the first invalid field.

7. **A11y on the page level.**
   - One `<h1>` per page.
   - Logical heading hierarchy (no skipping).
   - Skip link if a complex nav is above the main content.
   - `<main>` landmark wraps the primary content.
   - Live regions (`aria-live="polite"`) for async toasts.
   - Lighthouse a11y ≥95.

8. **Performance budget.** See plan §43.
   - LCP ≤2.5s on 4G.
   - INP ≤200ms.
   - CLS ≤0.1.
   - JS shipped to client ≤180KB gz on first load.
   - `next/image` with `sizes` for above-the-fold images.

9. **Telemetry.**
   - Page view event auto-captured by PostHog provider — no manual
     call needed.
   - Custom events go through `track()` from
     `packages/shared/analytics/events.ts`. Define the event there
     first; do **not** inline event names.

10. **i18n.**
    - Strings must come from `packages/i18n` via `t('namespace.key')`.
    - No raw user-facing strings in JSX.
    - Add the new keys to `en.json` (canonical) + at least the
      placeholder for other locales.

11. **Tests.**
    - Playwright smoke test for the happy path:
      ```ts
      // apps/web/tests/e2e/projects.spec.ts
      test('lists projects for current org', async ({ page, login }) => {
        await login('owner@acme.test');
        await page.goto('/o/acme/projects');
        await expect(page.getByRole('heading', { name: 'Projects' })).toBeVisible();
      });
      ```
    - Vitest for any pure helpers used by the page.

12. **Manual checks before PR.**
    - 320px viewport — no horizontal scroll.
    - Keyboard-only — every interactive element reachable.
    - Slow 3G + CPU 6× throttle in DevTools — still usable.
    - Dark mode — verify.
    - With JS disabled — Server-rendered content visible.

## Pitfalls

- Marking a page `'use client'` because one button is interactive. Keep
  the page server, push the client island down to the leaf component.
- Reading `process.env.*` directly from a page — use
  `packages/shared/config.ts`.
- Calling Better Auth from a Client Component — auth runs on the
  server only.
- Forgetting `redirect()` after `requireSession` returns no org —
  causes a render of stale data.
- Using `useEffect` to fetch — fetch on the server with `await`.
- Letting heading hierarchy break across nested layouts.

## Done when

- Page renders with all four state files (`page` / `loading` / `error`
  / `not-found`).
- Auth + RLS guards in place.
- Lighthouse a11y ≥95, perf ≥90 on dev build.
- Playwright smoke test green.
- i18n keys added.
- Mobile + dark mode verified manually.
