---
name: add-package
description: Scaffold a new internal package under packages/ in the TESKEL monorepo. Use when the work cannot fit into an existing package and a new bounded context is needed (e.g., a new domain like packages/billing/ or packages/notifications/).
---

# add-package — scaffold a new internal package

> Use this skill when the requested change is large enough to deserve a
> dedicated package (own dependencies, own tests, distinct bounded
> context). When in doubt, **prefer extending an existing package** —
> creating new packages adds maintenance overhead.

## When to use

- A new domain that does not fit any existing `packages/*` (auth, db,
  ai, queue, runner, ui, shared, sdk, cli, template-engine, billing,
  testing).
- A package that will be **published externally** (only `sdk` and `cli`
  are public today).

## When NOT to use

- The code naturally belongs to an existing package — extend that one.
- The package is "utilities for app X" — those go in
  `apps/<x>/src/lib/`.

## Steps

1. **Confirm the need.** Re-read [`AGENTS.md` §4](../../../AGENTS.md#4-repository-map-target-structure).
   If the new package is not in the planned structure and is not a
   trivial extraction, **open an RFC first** at `docs/rfc/`.

2. **Pick the name.**
   - npm name: `@teskel/<short-name>` (kebab-case, ≤2 words).
   - Folder: `packages/<short-name>/`.

3. **Scaffold files.**

   ```bash
   mkdir -p packages/<short-name>/{src,test}
   cd packages/<short-name>
   ```

   `package.json`:

   ```json
   {
     "name": "@teskel/<short-name>",
     "version": "0.0.0",
     "private": true,
     "type": "module",
     "main": "./src/index.ts",
     "types": "./src/index.ts",
     "scripts": {
       "lint": "eslint .",
       "typecheck": "tsc --noEmit",
       "test": "vitest run"
     },
     "dependencies": {},
     "devDependencies": {
       "@teskel/testing": "workspace:*",
       "typescript": "*",
       "vitest": "*"
     }
   }
   ```

   `tsconfig.json`:

   ```json
   {
     "extends": "../../tsconfig.base.json",
     "include": ["src/**/*", "test/**/*"]
   }
   ```

   `src/index.ts` — public surface only. Keep it small and intentional.

4. **Wire up workspace.**
   - Already covered by `pnpm-workspace.yaml` glob `packages/*`.
   - Run `pnpm install` at the repo root to refresh links.

5. **Add CODEOWNERS line.**

   ```text
   /packages/<short-name>/  @<owner-handle>
   ```

6. **Add a README.md** in the package describing:
   - Purpose (1 sentence).
   - Public API (re-exports from `src/index.ts`).
   - Internal layout.
   - When NOT to use it.

7. **Add at least one passing test** in `test/` so CI proves the
   scaffold works.

8. **Update**:
   - [`AGENTS.md` §4](../../../AGENTS.md#4-repository-map-target-structure)
     repository map.
   - [`TESKEL_FULLSTACK_BUILD_BREAKDOWN.md`](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md)
     Sec. 17 (monorepo) if the package adds a new top-level concern.

9. **Commit & PR.**
   - Branch: `<author>/scaffold-<short-name>-package`.
   - Title: `feat(<short-name>): scaffold package`.
   - PR template filled (rollback plan: `rm -rf packages/<short-name>`).

## Pitfalls

- Do not import `@teskel/db` from `@teskel/ui` (UI must remain
  data-agnostic).
- Do not couple to `@teskel/auth` from packages other than `app/web`,
  `app/api`, and route handlers.
- Avoid circular deps — Turborepo will yell.
- Do not bump shared deps in this PR; keep the diff focused.

## Done when

- `pnpm turbo run lint typecheck test --filter @teskel/<short-name>`
  passes.
- Package appears in `pnpm ls --depth -1` from the repo root.
- README.md present.
- CODEOWNERS updated.
