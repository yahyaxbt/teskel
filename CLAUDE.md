# CLAUDE.md

This file is read by [Claude Code](https://docs.anthropic.com/en/docs/claude-code)
when working in this repository.

**The canonical agent instructions live in [`AGENTS.md`](./AGENTS.md).**
This file intentionally re-exports the same rules so that Claude Code
(and any tooling that looks for `CLAUDE.md` specifically) gets the full
context.

Read these files in order before making any change:

1. [`AGENTS.md`](./AGENTS.md) — operating manual (rules, conventions,
   forbidden actions, escalation triggers).
2. [`TESKEL_FULLSTACK_BUILD_BREAKDOWN.md`](./TESKEL_FULLSTACK_BUILD_BREAKDOWN.md)
   — the master plan and source of truth for product / architecture /
   roadmap.
3. [`.agents/state/current-phase.md`](./.agents/state/current-phase.md)
   — the phase the project is currently in. Stay inside that scope.
4. The relevant skill in [`.agents/skills/`](./.agents/skills/) if your
   task matches one (e.g., `add-package`, `add-table`,
   `add-workflow-node`, `release-canary`, `incident-sev1`).

If you are about to do anything that's listed under
[§16 Forbidden Actions](./AGENTS.md#16-forbidden-actions-without-human-approval)
of `AGENTS.md` — stop and ask the human owner first.

When you find a conflict between this file and `AGENTS.md`, **`AGENTS.md`
wins** and this file should be updated to match in the same PR.
