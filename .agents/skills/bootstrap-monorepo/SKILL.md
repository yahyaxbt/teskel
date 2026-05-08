---
name: bootstrap-monorepo
description: Scaffold the empty TESKEL monorepo (pnpm workspaces + Turborepo + tsconfig + lint/format + Docker base + GitHub Actions skeleton + Husky pre-commit). The very first PR of Phase 0 W1. Once accepted, the repo is ready for `add-package` / `add-table` skills to drop real packages into the structure.
---

# bootstrap-monorepo — first PR of Phase 0

> Plan ref: [Sec. 88 (Phase 0 / Foundation)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#88-phase-0--foundation--minggu-14),
> [Sec. 16 (C4-Light)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#16-c4-light--konteks--container--component),
> [Sec. 17 (Repo map)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#17-repo-map-target).
>
> This skill runs **once**. Subsequent packages use
> [`add-package`](../add-package/SKILL.md).

> **Hard rule:** the bootstrap PR adds **no business logic**. No
> auth, no DB schema, no UI components. Its job is to give every
> following PR a place to land.

## Pre-conditions

- [`AGENTS.md`](../../../AGENTS.md) merged (already true after PR #2).
- Plan v2.3+ on `main`.
- `current-phase.md` set to `phase: 0` with `phase_status:
  not_started`.
- ADR-0001..0010 are open as drafts via [`write-adr`](../write-adr/SKILL.md)
  (they don't have to be merged yet — but the placeholders should
  exist so the bootstrap PR can reference them).

## Steps

1. **Open the kickoff story.** File `docs/stories/P0-001-bootstrap-monorepo.md`
   from the [template](../../../docs/stories/0000-template.md). Acceptance
   criteria below should map to it 1:1.

2. **Branch.** `devin/<ts>-bootstrap-monorepo` or `feat/bootstrap-monorepo`.

3. **Lock the toolchain with `mise`.** Add `.mise.toml`:

   ```toml
   [tools]
   node = "20.18.0"
   pnpm = "9.12.3"
   python = "3.12.6"   # for tooling like pre-commit
   "go:github.com/wal-g/wal-g" = "3.0.3"   # used by ops
   ```

   Document install instructions in `README.md` (`curl https://mise.run | sh`).

4. **`package.json` at root.**

   ```json
   {
     "name": "teskel",
     "private": true,
     "packageManager": "pnpm@9.12.3",
     "engines": { "node": ">=20.18 <21", "pnpm": ">=9.12 <10" },
     "scripts": {
       "build": "turbo run build",
       "dev": "turbo run dev --parallel",
       "lint": "turbo run lint",
       "lint:fix": "turbo run lint -- --fix",
       "typecheck": "turbo run typecheck",
       "test": "turbo run test",
       "test:e2e": "turbo run test:e2e",
       "format": "prettier --write .",
       "format:check": "prettier --check .",
       "clean": "turbo run clean && rm -rf node_modules .turbo"
     },
     "devDependencies": {
       "turbo": "^2.1.3",
       "prettier": "^3.3.3",
       "typescript": "^5.6.2",
       "husky": "^9.1.6",
       "lint-staged": "^15.2.10",
       "@commitlint/cli": "^19.5.0",
       "@commitlint/config-conventional": "^19.5.0"
     }
   }
   ```

5. **`pnpm-workspace.yaml`.**

   ```yaml
   packages:
     - apps/*
     - packages/*
     - tooling/*
   ```

6. **`turbo.json`.**

   ```json
   {
     "$schema": "https://turbo.build/schema.json",
     "globalDependencies": ["**/.env.*local", ".mise.toml"],
     "globalEnv": ["NODE_ENV"],
     "tasks": {
       "build": { "dependsOn": ["^build"], "outputs": ["dist/**", ".next/**"] },
       "dev":   { "cache": false, "persistent": true },
       "lint":  {},
       "typecheck": { "outputs": [] },
       "test":  { "dependsOn": ["^build"], "outputs": ["coverage/**"] },
       "test:e2e": { "dependsOn": ["^build"], "outputs": ["playwright-report/**"], "cache": false },
       "clean": { "cache": false }
     }
   }
   ```

7. **TypeScript base config** at `tooling/tsconfig/base.json`:

   ```json
   {
     "$schema": "https://json.schemastore.org/tsconfig",
     "compilerOptions": {
       "target": "ES2022",
       "lib": ["ES2022"],
       "module": "ESNext",
       "moduleResolution": "Bundler",
       "strict": true,
       "noUncheckedIndexedAccess": true,
       "noImplicitOverride": true,
       "exactOptionalPropertyTypes": true,
       "esModuleInterop": true,
       "isolatedModules": true,
       "skipLibCheck": true,
       "forceConsistentCasingInFileNames": true,
       "verbatimModuleSyntax": true,
       "resolveJsonModule": true,
       "incremental": true
     }
   }
   ```

   Each package's `tsconfig.json` extends this:

   ```json
   { "extends": "@teskel/tsconfig/base.json", "include": ["src/**/*.ts"] }
   ```

8. **ESLint at the root** (`eslint.config.mjs`, flat config):

   ```js
   import js from '@eslint/js';
   import tseslint from 'typescript-eslint';
   import importPlugin from 'eslint-plugin-import';
   import unusedImports from 'eslint-plugin-unused-imports';

   export default tseslint.config(
     { ignores: ['**/dist/**', '**/.next/**', '**/.turbo/**', '**/node_modules/**'] },
     js.configs.recommended,
     ...tseslint.configs.recommendedTypeChecked,
     {
       plugins: { import: importPlugin, 'unused-imports': unusedImports },
       languageOptions: { parserOptions: { project: true } },
       rules: {
         'no-console': ['error', { allow: ['warn', 'error'] }],
         'unused-imports/no-unused-imports': 'error',
         'import/order': ['warn', { 'newlines-between': 'always', alphabetize: { order: 'asc' } }],
       },
     },
   );
   ```

9. **Prettier config** at root:

   ```json
   {
     "printWidth": 100,
     "singleQuote": true,
     "trailingComma": "all",
     "semi": true,
     "arrowParens": "always",
     "plugins": ["prettier-plugin-packagejson", "prettier-plugin-tailwindcss"]
   }
   ```

10. **`.editorconfig`**:

    ```ini
    root = true
    [*]
    charset = utf-8
    end_of_line = lf
    insert_final_newline = true
    trim_trailing_whitespace = true
    indent_style = space
    indent_size = 2
    [*.md]
    trim_trailing_whitespace = false
    ```

11. **`.gitattributes`**:

    ```
    * text=auto eol=lf
    *.png binary
    *.jpg binary
    *.svg text
    pnpm-lock.yaml -diff
    *.lock -diff
    ```

12. **`.gitignore`** (curated, not the noisy default):

    ```
    node_modules
    .next
    dist
    .turbo
    coverage
    .env
    .env.*
    !.env.example
    .DS_Store
    *.log
    .vscode/*
    !.vscode/extensions.json
    !.vscode/settings.json
    ```

13. **Husky + lint-staged + commitlint:**

    ```bash
    pnpm dlx husky-init && pnpm install
    ```

    Replace `.husky/pre-commit`:

    ```sh
    #!/usr/bin/env sh
    pnpm exec lint-staged
    ```

    Add `.husky/commit-msg`:

    ```sh
    #!/usr/bin/env sh
    pnpm exec commitlint --edit "$1"
    ```

    `lint-staged.config.js`:

    ```js
    export default {
      '*.{ts,tsx,js,mjs,cjs}': ['eslint --fix', 'prettier --write'],
      '*.{json,md,yml,yaml}': ['prettier --write'],
      '*.sql': ['prettier --write --plugin=prettier-plugin-sql'],
    };
    ```

    `commitlint.config.cjs`:

    ```js
    module.exports = { extends: ['@commitlint/config-conventional'] };
    ```

14. **Folder structure** (empty placeholders + READMEs so the
    structure is committed):

    ```
    apps/
      web/                 # Next.js (App Router)
      api/                 # Hono
      worker/              # BullMQ workers
      studio/              # Workflow studio (Phase 1)
      builder/             # Visual builder (Phase 2)
    packages/
      auth/                # Better Auth wiring
      db/                  # Drizzle schema + migrations
      shared/              # config, logger, errors, retry
      ui/                  # design system + components
      blocks/              # Puck blocks (Phase 2)
      ai/                  # AI gateway, prompt registry
      integrations/
        stripe/
        resend/
        openrouter/
      jobs/                # BullMQ job + worker definitions
      notifications/
        email/             # Resend templates
      sdk/                 # TS SDK (Phase 2+)
      cli/                 # `teskel` CLI (Phase 2+)
      queue/               # BullMQ + Inngest wrappers
      observability/       # OTel + Langfuse + PostHog server
    tooling/
      tsconfig/
      eslint-config/       # extracted in Phase 1 if shared rules grow
    ```

    Each folder gets a `README.md` (≤10 lines) and an empty
    `package.json` only if the folder is a real workspace member.
    The placeholder packages export nothing — they are scaffolds.

15. **Dockerfile base.** Single `Dockerfile` per app, multi-stage,
    distroless final stage, BuildKit-cache-aware.

    Skeleton at `apps/api/Dockerfile`:

    ```dockerfile
    FROM node:20.18.0-bookworm-slim AS deps
    RUN corepack enable && corepack prepare pnpm@9.12.3 --activate
    WORKDIR /app
    COPY pnpm-workspace.yaml package.json pnpm-lock.yaml ./
    COPY apps/api/package.json apps/api/
    COPY packages packages/
    RUN --mount=type=cache,id=pnpm,target=/pnpm/store pnpm install --frozen-lockfile

    FROM deps AS build
    COPY . .
    RUN pnpm --filter @teskel/api build

    FROM gcr.io/distroless/nodejs20-debian12 AS runtime
    WORKDIR /app
    COPY --from=build /app/apps/api/dist ./dist
    COPY --from=build /app/node_modules ./node_modules
    USER nonroot:nonroot
    EXPOSE 8080
    CMD ["dist/server.js"]
    ```

    Same shape for `apps/web` and `apps/worker`.

16. **GitHub Actions skeleton**, `.github/workflows/ci.yml`:

    ```yaml
    name: ci
    on:
      push: { branches: [main] }
      pull_request: { branches: [main] }
    concurrency: { group: ci-${{ github.ref }}, cancel-in-progress: true }
    jobs:
      lint-typecheck-test:
        runs-on: ubuntu-24.04
        steps:
          - uses: actions/checkout@v4
          - uses: pnpm/action-setup@v4
            with: { version: 9.12.3 }
          - uses: actions/setup-node@v4
            with: { node-version: 20.18.0, cache: pnpm }
          - run: pnpm install --frozen-lockfile
          - run: pnpm format:check
          - run: pnpm lint
          - run: pnpm typecheck
          - run: pnpm test --filter=...[origin/main]
    ```

    Add `release.yml` skeleton (no actual release yet) and
    `e2e.yml` placeholder (disabled until Phase 0 W3).

17. **`.env.example`** at root + per-app:

    ```
    # ---- shared ----
    NODE_ENV=development
    LOG_LEVEL=debug

    # ---- DB ----
    DATABASE_URL=postgres://teskel:teskel@localhost:5432/teskel_dev
    REDIS_URL=redis://localhost:6379

    # ---- auth ----
    BETTER_AUTH_SECRET=change_me_dev_only

    # ---- storage ----
    R2_ENDPOINT=
    R2_ACCESS_KEY_ID=
    R2_SECRET_ACCESS_KEY=

    # ---- ai ----
    OPENROUTER_API_KEY=

    # ---- observability ----
    OTEL_EXPORTER_OTLP_ENDPOINT=
    SENTRY_DSN=
    POSTHOG_HOST=
    POSTHOG_KEY=
    LANGFUSE_HOST=
    LANGFUSE_PUBLIC_KEY=
    LANGFUSE_SECRET_KEY=
    ```

    Real values live in Doppler / 1Password — never in the repo.

18. **Local dev compose** at `docker-compose.dev.yml` for Postgres
    + Redis + Mailhog + Minio (R2 stand-in):

    ```yaml
    services:
      postgres:
        image: postgres:16
        environment:
          POSTGRES_USER: teskel
          POSTGRES_PASSWORD: teskel
          POSTGRES_DB: teskel_dev
        ports: ['5432:5432']
        volumes: ['pg:/var/lib/postgresql/data']
      redis:
        image: redis:7-alpine
        ports: ['6379:6379']
      mailhog:
        image: mailhog/mailhog
        ports: ['8025:8025', '1025:1025']
      minio:
        image: minio/minio
        command: server /data --console-address ":9001"
        environment:
          MINIO_ROOT_USER: teskel
          MINIO_ROOT_PASSWORD: teskel123
        ports: ['9000:9000', '9001:9001']
        volumes: ['minio:/data']
    volumes: { pg: {}, minio: {} }
    ```

19. **CODEOWNERS** already exists (PR #2). Verify ownership is set
    for the new top-level folders (`apps/`, `packages/`, `tooling/`,
    `.github/`).

20. **Smoke check.** From a clean clone:

    ```bash
    pnpm install
    pnpm format:check
    pnpm lint
    pnpm typecheck
    pnpm test
    ```

    All four MUST pass on the bootstrap PR. They will be no-ops at
    first (no real code), but the wiring must work.

21. **README.md update.** Bump the `Quick start` section to match
    the new commands. Mention `mise` first.

22. **PR title:** `chore(repo): bootstrap monorepo (P0-001)`. PR
    description references plan §88 W1 + the story file.

23. **Acceptance.** Reviewer (platform owner) verifies:

    - `pnpm i` from a fresh clone works.
    - `pnpm format:check && pnpm lint && pnpm typecheck && pnpm test`
      all green.
    - Folder structure matches the layout doc.
    - Husky hook fires (try a commit with non-conventional message
      — should be rejected).
    - GitHub Actions runs and goes green.

24. **Close the story.** Update P0-001 to Done. Mark exit criterion
    `repo_scaffolded` in `current-phase.md` as `verified: true` (if
    that criterion exists).

## What this PR does NOT do

- Add a real DB schema. That's the next story (P0-002, uses
  [`add-table`](../add-table/SKILL.md)).
- Add Better Auth. That's P0-003.
- Add a real Hono route. That's part of P0-002+.
- Set up Coolify production deploy. That's P0-004.
- Wire Sentry / OTel / PostHog real DSNs. That's P0-005.

Each downstream story uses the [`add-package`](../add-package/SKILL.md)
skill to land its workspace package and the relevant skill for
its concern.

## Pitfalls

- Adding a "small bit of business logic" to the bootstrap PR.
  Reject in review — keep this PR boring.
- Pinning versions loosely (`^` ranges in `engines`) — bootstrap
  pins exact, downstream packages can use ranges.
- Forgetting `.husky` permissions — `chmod +x .husky/*`.
- Skipping `pnpm-lock.yaml` commit — non-determinism in CI.
- Bringing every dev dependency in at once — bootstrap is minimal;
  add deps as packages need them via `add-package`.
- Creating ESLint rules nobody can pass yet — start with the
  default recommended set; tighten in Phase 1 review.
- Not writing the kickoff story — leaves no rationale trail.
- Adding `.env` (real) instead of `.env.example` — security issue.

## Done when

- Branch merged, CI green, no warnings.
- Fresh-clone smoke (`install → format:check → lint → typecheck →
  test`) passes.
- README quick start matches reality.
- P0-001 story marked Done.
- Exit criterion `repo_scaffolded` flipped if applicable.
- The next story (P0-002) is unblocked.
