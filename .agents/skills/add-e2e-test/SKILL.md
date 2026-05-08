---
name: add-e2e-test
description: Add a Playwright end-to-end test for a critical user flow in apps/web. Covers fixtures, isolated tenant setup, deterministic data, network mocking policy, accessibility assertions, and CI sharding. Use whenever a new golden flow ships or a regression escapes lower-level tests.
---

# add-e2e-test — add a Playwright E2E test

> Plan ref: [Sec. 12 (Golden flows)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#12-golden-user-flows),
> [Sec. 75 (Test pyramid)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#75-test-pyramid--quality).

> **Hard rule:** E2E is the smallest layer of the pyramid. If logic
> can be tested at unit (Vitest) or integration (Vitest +
> Testcontainers) level, do it there. E2E is for **user-visible flows**
> that span >2 services.

## When to add E2E

Yes:
- Sign up → onboarding → first-project flow.
- Editor save → publish → live preview.
- Marketplace install → run → uninstall.
- Stripe checkout → subscription active → entitlements unlocked.
- Critical RBAC denials (member can't reach admin pages).

No:
- A button label change.
- A pure utility function.
- Anything that needs a third-party flaky external (test against a
  recorded fixture instead).

## Test layout

```
apps/web/tests/e2e/
  fixtures/
    auth.fixture.ts        # custom Playwright fixture that logs in
    tenant.fixture.ts      # creates an isolated org per worker
    seed.fixture.ts        # seed deterministic data
  flows/
    onboarding.spec.ts
    publish-page.spec.ts
    marketplace-install.spec.ts
  utils/
    selectors.ts           # `data-testid` constants
    pageObjects/
      ProjectsPage.ts
      EditorPage.ts
  playwright.config.ts
```

## Steps

1. **Pick the flow.** One scenario per spec file. File name describes
   the user goal: `onboarding.spec.ts`, not `test1.spec.ts`.

2. **Use Page Objects** for any interaction reused in 2+ specs.

   ```ts
   // apps/web/tests/e2e/utils/pageObjects/ProjectsPage.ts
   import type { Page } from '@playwright/test';

   export class ProjectsPage {
     constructor(private page: Page) {}
     async goto(orgSlug: string) {
       await this.page.goto(`/o/${orgSlug}/projects`);
     }
     async createProject(name: string) {
       await this.page.getByRole('button', { name: 'New project' }).click();
       await this.page.getByLabel('Project name').fill(name);
       await this.page.getByRole('button', { name: 'Create' }).click();
     }
     async expectVisible(name: string) {
       await this.page.getByRole('link', { name }).waitFor();
     }
   }
   ```

3. **Use the auth + tenant fixtures.** Each test gets an isolated org
   so parallel runs don't collide.

   ```ts
   // apps/web/tests/e2e/fixtures/auth.fixture.ts
   import { test as base, expect } from '@playwright/test';
   import { createTestOrg, deleteTestOrg } from '@teskel/test-helpers';

   export const test = base.extend<{
     orgSlug: string;
     login: (email?: string) => Promise<void>;
   }>({
     orgSlug: async ({}, use) => {
       const { slug } = await createTestOrg();
       await use(slug);
       await deleteTestOrg(slug);
     },
     login: async ({ page }, use) => {
       await use(async (email = 'owner@e2e.test') => {
         await page.goto('/api/auth/test-login?email=' + email);
         await page.waitForURL('/');
       });
     },
   });
   export { expect };
   ```

   Helper `createTestOrg()` calls a guarded internal endpoint
   (`/api/internal/test/org`) that's only enabled when
   `NODE_ENV !== 'production'` and `INTERNAL_TEST_TOKEN` matches.

4. **Locator strategy.** Prefer roles + accessible names; use
   `data-testid` only when ambiguity forces it.

   ```ts
   page.getByRole('button', { name: 'Publish' });          // best
   page.getByLabel('Project name');                        // good
   page.getByText('All projects', { exact: true });        // ok
   page.getByTestId('editor-save-button');                 // last resort
   ```

   Forbidden: CSS selectors that depend on classnames or DOM
   structure (`.btn.btn-primary > span`).

5. **Wait properly. No `page.waitForTimeout`.**
   - Use auto-waiting locators (`.click()` waits for actionable).
   - Use `page.waitForURL` after navigation.
   - Use `expect(locator).toBeVisible({ timeout })` for assertions.

6. **Network policy.**
   - **First-party APIs:** test against the real backend running in
     the e2e environment. Don't mock our own APIs in E2E.
   - **Third parties (Stripe, OpenAI, Resend):** intercept with
     `page.route()` and return fixture responses. Do NOT call real
     vendors in E2E.

   ```ts
   await page.route('**/api.openai.com/**', (route) =>
     route.fulfill({ status: 200, body: JSON.stringify(fixtureLLMResponse) }));
   ```

7. **Deterministic data.** Seed the org with known data via the
   `seed.fixture.ts` helper. Never rely on "whatever is in the DB".

8. **Snapshots.** For visual diffs, use Playwright's
   `expect(page).toHaveScreenshot()` with `mask` on volatile regions
   (timestamps, user-generated avatars). Update snapshots only via
   `pnpm e2e:update --grep "name"`, with a reviewer.

9. **Accessibility assertion in the same test.** For any page-level
   spec:

   ```ts
   import AxeBuilder from '@axe-core/playwright';
   test('projects page is accessible', async ({ page, login, orgSlug }) => {
     await login();
     await page.goto(`/o/${orgSlug}/projects`);
     const results = await new AxeBuilder({ page }).analyze();
     expect(results.violations).toEqual([]);
   });
   ```

10. **Tracing.** `playwright.config.ts` enables `trace: 'on-first-retry'`
    so failures upload a trace bundle to CI artifacts.

11. **Sharding.** CI runs 4 shards in parallel:
    `--shard=1/4 ... --shard=4/4`. Tests must be **independent** and
    **idempotent** — never depend on order, never share global state.

12. **Flake budget.** A test that fails ≥1 in 50 runs is flaky.
    Quarantine it (`test.fixme`), open an issue, fix or delete within
    1 sprint. Do not retry it indefinitely in CI.

13. **Run locally**:
    ```bash
    pnpm --filter @teskel/web e2e --headed --grep "publish a page"
    ```

14. **CI integration.** Add to `apps/web/playwright.config.ts` if a
    new project (e.g., new browser); otherwise the new spec is picked
    up automatically.

## Pitfalls

- Calling real Stripe / OpenAI / Resend in E2E — slow, flaky, costs
  money, leaks PII.
- Reading another test's data (cross-test coupling) — one slow CI
  run reorders execution and the suite breaks.
- Asserting on classnames or implementation details — refactors
  break tests with no behavior change.
- Hard-coded timeouts (`waitForTimeout(2000)`) — flake on slow CI.
- One spec testing five flows — split, name each by user goal.
- Using `globalSetup` to "log in once" — breaks parallelism and
  isolation; always log in per worker.

## Done when

- Spec is in `apps/web/tests/e2e/flows/<flow>.spec.ts`.
- Uses fixtures (auth + tenant + seed) for isolation.
- Locators are role/label-based.
- Third parties mocked via `page.route()`.
- A11y assertion included.
- Passes on a clean run + on retry — no flake.
- Reviewed by another agent or human.
