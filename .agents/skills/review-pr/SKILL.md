---
name: review-pr
description: Review a pull request as a senior engineer or AI reviewer — applies the TESKEL hard-rules checklist, security review, performance review, and DoD verification. Use this skill before approving any PR.
---

# Skill — Review a PR

> **Plan refs:** §69 (DoR/DoD), §70 (PR workflow), §97
> (pitfalls per phase).
> **Related skills:** every skill that opens a PR (i.e. all of them).
> **Hard rules:** [`AGENTS.md` §8](../../../AGENTS.md#8-architecture-hard-constraints),
> [`AGENTS.md` §13](../../../AGENTS.md#13-pr-workflow-branch-commit-review).

A PR review at TESKEL is a **two-axis** check:

1. **Correctness** — does the code do what the PR description
   claims?
2. **Compliance** — does the code respect the platform's hard rules
   (RLS, idempotency, outbox, prompts via registry, etc.)?

A reviewer who only checks the first is missing half the job. The
second axis is what keeps a fast-moving repo from drifting into a
slow-moving one.

This skill is for **both** human reviewers and AI reviewers
(Cursor, Copilot, Devin). Skim once before every review you do.

---

## When to invoke

- You are CODEOWNER on a PR.
- You're an AI reviewer leaving an inline review.
- You're a release captain pre-flighting the train (per
  [`release-canary`](../release-canary/SKILL.md) Step 2).
- You're an interviewer simulating a code review for a candidate.

This is **not** for triaging customer bug reports — see
[`triage-bug`](../triage-bug/SKILL.md).

---

## Steps

### 1. Read the PR description first

Before opening any file:

- The **title** should follow Conventional Commits / repo norms
  (`feat:`, `fix:`, `chore:`, etc.). Reject otherwise.
- The PR template's checkboxes should be filled. Empty boxes are a
  red flag, not a request to fill them on the reviewer's behalf.
- The **scope** should match the **branch name** and the **current
  phase** (per [`.agents/state/current-phase.md`](../../state/current-phase.md)).
- A linked **story** / **ADR** / **RFC** belongs in the description
  for any non-trivial change.

If any of the above is missing, leave a single comment requesting
the fix and **stop**. Do not deep-review a PR with a bad description
— it wastes everyone's time.

### 2. Map the diff

Run `git diff --stat` (or your tool's equivalent). Note:

- Which packages / apps are touched.
- Which directories appear that didn't before.
- File-count + line-count. > 800 LoC? Push back unless the diff is
  generated (lockfile, snapshot, OpenAPI).
- Any **forbidden** path edits: `apps/api/openapi.yml`,
  `pnpm-lock.yaml` for unrelated changes, generated migration files,
  `docs/adr/<merged>.md`. See §6.

### 3. Hard-rules checklist

Walk this list in order. If any answer is "No", leave an inline
comment and block.

| # | Question | If yes / no |
| --- | --- | --- |
| 1 | If the PR adds a table → is RLS enabled + cross-tenant negative test added? | Required (per [`add-table`](../add-table/SKILL.md)). |
| 2 | If the PR adds a mutating API route → is `Idempotency-Key` enforced? Zod-validated body? `withTenant` scoped? RBAC checked? | Required (per [`add-api-route`](../add-api-route/SKILL.md), [`docs/api/conventions.md`](../../../docs/api/conventions.md), [`docs/api/idempotency.md`](../../../docs/api/idempotency.md)). |
| 3 | If the PR sends an email / posts a webhook / charges a card → does it go through the **outbox**? | Required. Direct vendor call from a handler is a block. |
| 4 | If the PR calls an LLM → does it go through the **AI gateway** with a **prompt slot** + **eval set**? | Required (per [`add-prompt-slot`](../add-prompt-slot/SKILL.md)). |
| 5 | If the PR adds an integration → is the vendor SDK isolated to `packages/integrations/<vendor>/`? Typed client? Kill-switch flag? Circuit breaker? | Required (per [`add-integration`](../add-integration/SKILL.md)). |
| 6 | If the PR runs user code → does it go through the sandbox broker (E2B)? | Required. In-process execution is a Sev1 risk. |
| 7 | If the PR adds a worker → is the consumer idempotent on `(jobId)`? Does it read `tenantId` from `job.data` on every retry? | Required. |
| 8 | If the PR adds a migration → is it expand-then-contract? Backfill via [`data-backfill-job`](../data-backfill-job/SKILL.md) skill, not inline? | Required. |
| 9 | If the PR adds a feature → is it behind a feature flag (per [`add-feature-flag`](../add-feature-flag/SKILL.md))? | Required for risky / cross-cutting changes. |
| 10 | If the PR adds a permission / role → is it in the registry + audit-logged? | Required (per [`add-rbac-role`](../add-rbac-role/SKILL.md)). |
| 11 | If the PR adds a runbook / dashboard → does the corresponding alert link to it? | Required (per [`add-runbook`](../add-runbook/SKILL.md)). |
| 12 | If the PR introduces a new top-level npm dep → is it justified in the description? | Reviewer may push back if the dep is borderline. |
| 13 | If the PR is a hotfix → does it follow [`release-hotfix`](../release-hotfix/SKILL.md) (no migrations, no refactors)? | Required. |

If 13 questions feels like a lot — yes. The platform deliberately
trades reviewer effort for runtime safety.

### 4. Architecture review

Read the diff with these lenses:

- **Boundaries:** Server vs Client component split? Zod-validated
  inputs at the boundary? Types flow inward, never outward?
- **Dependencies:** New imports across package boundaries that
  shouldn't be there (e.g. `apps/web` directly importing
  `apps/workers`)? PR should split into shared `packages/*`.
- **Concerns:** Is the change in the right place? UI logic in a
  Hono handler is a smell; DB logic in a React component is a
  block.
- **Naming:** Verbs match conventions
  ([`docs/observability/README.md` §5](../../../docs/observability/README.md));
  IDs are UUID v7; money is integer minor + currency.
- **Tests:** Unit + integration + (where relevant) cross-tenant
  negative + idempotency replay + e2e if a user-visible flow.

### 5. Security review (always; even tiny PRs)

- Any new external surface (route, webhook, SDK method)? Walk
  through [`docs/architecture/threat-model.md`](../../../docs/architecture/threat-model.md)
  for the matching surface.
- Any new secret / credential / signing key? Confirm rotation path
  per [`rotate-secret`](../rotate-secret/SKILL.md).
- Any PII flowing somewhere new? Confirm logging + retention class
  + DSAR cascade.
- Any new permission added? Confirm it's in the matrix +
  audit-logged.
- Any change to auth / session / token verification? **Two
  reviewers required.**
- Any input that affects an SQL string? Confirm parameterized.
- Any redirect or open-href? Confirm allowlisted.

### 6. Forbidden edits — block on sight

These categories require an ADR / explicit approval, not just a
PR. Block immediately:

- Disabling RLS or removing `withTenant` calls.
- Editing `apps/api/openapi.yml` directly (it is generated).
- Editing a merged `docs/adr/<NNNN>-*.md`.
- `--no-verify`, `--force` push to `main`/`master`, or amending
  commits on shared branches.
- Adding `console.log(user.email)` or any PII to a log line.
- Importing a vendor SDK in `apps/*` (must live in
  `packages/integrations/<vendor>/`).
- Adding a "global" / cross-tenant table without an ADR.
- Bypassing the outbox for emails, webhooks, charges.
- Embedding secrets / API keys in code or env files.
- Skipping pre-commit hooks.

### 7. Performance & cost review

- New DB query? Confirm `EXPLAIN` plan if hot path; add an index if
  needed. Confirm pagination is cursor-based, not offset.
- New LLM call? Confirm model + budget per
  [`docs/billing/plans.md`](../../../docs/billing/plans.md) and the
  slot's `expected_avg_cost_usd`.
- New worker job? Confirm the queue isn't unbounded; concurrency
  limits sane.
- New large payload (>1 MB) being stored in Redis or in a row?
  Push back; that belongs in R2.
- New image / asset > 200 KB? Confirm dimensions and format.

### 8. Test coverage check

For PRs that change runtime behavior:

- Unit tests for new pure functions.
- Integration tests for new DB / queue paths (incl. cross-tenant
  negative).
- E2E test for new user-visible flows (per [`add-e2e-test`](../add-e2e-test/SKILL.md)).
- Idempotency replay test for any new mutating route or worker.
- Eval pass-rate threshold met for any new prompt slot.

A PR with no tests for new behavior is a block unless the entire
diff is generated/configuration.

### 9. Comment hygiene

- Group comments. Don't leave 30 nits as 30 separate inline
  comments — bundle into a single review.
- Distinguish **must-fix**, **should-fix**, and **nit**. Suggested
  prefixes: `block:`, `consider:`, `nit:`.
- Suggest replacements where possible (`suggestion` blocks).
- Praise something explicitly. Reviews that are 100% complaint
  shape a culture of fear.
- Be terse. "Why a `Map` instead of `Record<string, T>`?" is
  better than three paragraphs.

### 10. Approving

Use the right action:

- **Approve** when every must-fix is addressed *and* you would be
  happy to be on-call for this code.
- **Comment** when you don't have authority to approve / block but
  want to leave thoughts.
- **Request changes** when there are must-fix items.

Do **not** "approve with comments" if there are real must-fix
items; that's how surprises ship.

### 11. After approval — observed for 24 h

If you reviewed something that ships in the same release train,
add a calendar reminder to skim the dashboards 24 h after deploy.
Reviewers are part of the on-call story for what they approved.

### 12. Self-review (when YOU are the author)

Treat your own PR like someone else's. The day before opening it:

- Re-read the diff start-to-end as if you'd never seen it.
- Run the unit + integration tests.
- Open the PR, fill the template, and **let it sit overnight** if
  the change is non-trivial. The morning re-read finds 80% of the
  bugs you'd otherwise ship.

---

## Pitfalls

- **"LGTM with one comment"** when the comment is a must-fix. Use
  *Request changes* and be explicit.
- **Drive-by approvals** on a domain you don't know. Decline; route
  to the right CODEOWNER.
- **Stylistic monologues.** If the codebase has a linter, a
  reviewer's job is *not* to enforce style; the linter does.
- **Solving the problem in the review.** If the PR is mis-shaped,
  request a redesign in a follow-up doc; don't try to land it via
  20 inline `suggestion` blocks.
- **Ignoring the description-vs-diff drift.** Description says one
  thing, diff does another → block until reconciled.
- **AI reviewers hallucinating filenames or APIs.** Always quote
  the line you're commenting on. If it doesn't exist in the diff,
  it doesn't exist.

---

## Done when

- Every hard-rule checklist item is verified or explicitly waived
  with reason.
- Security review is complete; no PII in new logs.
- Tests cover new behavior.
- Forbidden-edit checks pass.
- Performance / cost considerations stated.
- Approval, comment, or change request issued — never "approved
  with must-fix".
- Author has 1 sentence to start a thread on if they disagree (you
  did not just say "fix this").

---

## References

- [`AGENTS.md` §8, §13, §16](../../../AGENTS.md).
- [`.github/pull_request_template.md`](../../../.github/pull_request_template.md).
- [`docs/architecture/threat-model.md`](../../../docs/architecture/threat-model.md).
- [`docs/api/conventions.md`](../../../docs/api/conventions.md),
  [`docs/api/idempotency.md`](../../../docs/api/idempotency.md).
- Every other skill in `.agents/skills/` — they tell you what
  *should* be in the PR.
- Plan §69, §70, §97.
