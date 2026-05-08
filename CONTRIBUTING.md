# Contributing to TESKEL

Thanks for contributing. TESKEL is a heavily skill-driven repo: most
recurring operations have a documented checklist under
`.agents/skills/`. **Use them.** They exist so reviewers don't
re-litigate the same tradeoffs every PR.

If you are an AI coding agent, your full operating manual is
[`AGENTS.md`](./AGENTS.md). The rest of this document is the human-
readable summary; AGENTS.md is the source of truth on conflicts.

## Before you start

1. **Read [`AGENTS.md`](./AGENTS.md)** end-to-end — at least once.
   Skim periodically. The **hard rules** in §8 are non-negotiable.
2. **Check the active phase** in
   [`.agents/state/current-phase.md`](./.agents/state/current-phase.md)
   — it constrains scope. Out-of-phase work needs an [RFC](./docs/rfc/README.md).
3. **Find the relevant skill** in
   [`.agents/skills/`](./.agents/skills/). Skills are categorized in
   [`AGENTS.md` §19](./AGENTS.md#19-skills-library). If your task isn't
   covered, consider whether it merits a new skill (open a PR adding
   one) or a one-off RFC.

## How we ship

```
plan / RFC → ADR → skill (if recurring) → PR → review → CI → merge → release
```

- **Plan** ([`TESKEL_FULLSTACK_BUILD_BREAKDOWN.md`](./TESKEL_FULLSTACK_BUILD_BREAKDOWN.md))
  is the strategic document. It rarely changes; when it does it's
  through a deliberate PR.
- **RFCs** ([`docs/rfc/`](./docs/rfc/README.md)) explore non-trivial
  design.
- **ADRs** ([`docs/adr/`](./docs/adr/README.md)) record decisions.
  Immutable once accepted.
- **Skills** ([`.agents/skills/`](./.agents/skills/)) are the
  step-by-step procedures for recurring operations.
- **PRs** are the unit of change.

## Branch + commit conventions

- Branch naming: `<author>/<short-slug>` for humans,
  `devin/<timestamp>-<slug>` for AI.
- Single-concept PRs. If your PR title needs "and", split it.
- Keep PRs ≤500 LoC where reasonable. Bigger PRs need a strong
  reason (large generated changes, full-package introduction).
- Conventional Commits style for messages:
  `feat(scope): summary`, `fix(scope): summary`, `chore(scope):
  summary`, `docs(scope): summary`, `refactor(scope): summary`.
  Scope ≈ package or surface (`api`, `web`, `db`, `auth`, `ai`,
  `marketplace`, `ops`, `docs`).

## Pull request expectations (DoR / DoD)

Definition of Ready (before opening):

- [ ] Linked to a tracker item (or RFC / ADR if foundational).
- [ ] Stays in scope of the active phase or an approved RFC.
- [ ] Locally green: lint, typecheck, unit tests, relevant
      integration tests.
- [ ] Touched skill(s) read end-to-end and followed.

Definition of Done (before merge):

- [ ] PR template's compliance checks all box-checked or N/A with
      explanation.
- [ ] At least one Owner per affected area approved (see
      [`CODEOWNERS`](./.github/CODEOWNERS)).
- [ ] CI green.
- [ ] Docs updated where applicable (skill, ADR, plan, runbook,
      registry).
- [ ] Telemetry & alerts updated where applicable.

## Code style

- Languages: TypeScript (strict), SQL (Postgres 16+), bash.
- Formatter: Prettier (configured at repo root). Run
  `pnpm format`.
- Linter: ESLint (configured at repo root). Run `pnpm lint`.
- Imports at the top of files. No nested-in-function imports.
- No `any`, no `getattr`/`setattr`-style escape hatches. If you
  reach for one, you don't understand the type yet.
- Comments: terse and infrequent. Don't comment the diff. Don't
  paraphrase the code.

See [`AGENTS.md` §7](./AGENTS.md#7-coding-conventions) for the full
list.

## Testing

- Unit-test what's logic-y; integration-test what crosses
  boundaries (DB, queue, vendor); E2E-test only critical user
  flows.
- E2E: see [`add-e2e-test`](./.agents/skills/add-e2e-test/SKILL.md).
- Tenant isolation tests required for any new tenant-owned table
  (see [`add-table`](./.agents/skills/add-table/SKILL.md)).
- Fixtures + factories live with their package, not in a global
  `tests/` graveyard.

## Security

- Never commit secrets. CI scans for leaked keys; pre-commit hook
  blocks common patterns.
- Don't import vendor SDKs ad-hoc — use
  [`packages/integrations/`](./packages/integrations/) wrappers.
  See [`add-integration`](./.agents/skills/add-integration/SKILL.md).
- All tenant-owned tables enforce RLS. See
  [`add-table`](./.agents/skills/add-table/SKILL.md).
- Report security issues per [`SECURITY.md`](./SECURITY.md), **not**
  via public issues.

## Filing an issue

We use GitHub Issues for tracking. Templates:

- **Bug** — describe expected vs actual; include reproduction.
- **Feature** — link to RFC if non-trivial; otherwise short proposal
  + use case.
- **Operational** — alert + runbook link (per
  [`docs/runbooks/README.md`](./docs/runbooks/README.md)).

For incident-class issues, follow
[`incident-sev1`](./.agents/skills/incident-sev1/SKILL.md) /
[`incident-sev2`](./.agents/skills/incident-sev2/SKILL.md) skills —
not a generic issue.

## Code review

Review your own PR first. The PR template's checklist is the
reviewer's first pass; don't make a reviewer chase missing
checks.

When reviewing:

- Comment per concern, not per line. Drive-by nits are fine in
  separate threads.
- Block (request changes) for hard-rule violations only. For
  preferences, suggest.
- Approve when "I'd ship this" — not "this is perfect".

## Release & deploy

Releases follow [`release-canary`](./.agents/skills/release-canary/SKILL.md).
Emergency fixes follow [`release-hotfix`](./.agents/skills/release-hotfix/SKILL.md).

## License & copyright

TESKEL source is **proprietary** during the Phase 0–4 build window.
Phase 5+ may open-source select packages (CLI, SDK) under MIT.
Until then, do not republish. See [`README.md`](./README.md) for
current license status.

By contributing, you certify that:
- You wrote the code, or have rights to contribute it.
- You agree to license your contribution under the project's
  license.

## Questions?

- Architecture: open an issue tagged `architecture` or write an
  RFC.
- Day-to-day: ask in `#engineering` Slack channel.
- Security disclosure: see [`SECURITY.md`](./SECURITY.md).
