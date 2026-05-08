---
name: add-feature-flag
description: Add a PostHog feature flag with a typed wrapper, default OFF, targeting rules, rollout plan, kill-switch, and removal SLA. Use whenever shipping risky, in-progress, or phase-ahead changes.
---

# add-feature-flag — add a feature flag

> Plan ref: [Sec. 64 (A/B + flags)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#64-experiments-ab--feature-flags),
> [Sec. 70 (DoR/DoD)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#70-definition-of-ready--done),
> AGENTS.md §12 (phase awareness).

Default: **every user-visible non-trivial change ships behind a flag,
default OFF**. Flags are NOT a long-term substitute for proper
versioning; every flag has a removal date.

## Naming

`flag.<scope>.<name>` lowercase, kebab-case.
- `flag.studio.ai-suggest-next-node`
- `flag.marketplace.refund-self-serve`
- `flag.billing.metering-v2`

Boolean flags only by default. Use multivariate (string) only for A/B
tests with PostHog Experiments.

## Steps

1. **Create the flag in PostHog.**
   - Go to PostHog → Feature Flags → New flag.
   - Key: `flag.studio.ai-suggest-next-node`.
   - Description: 1 sentence + Linear/issue link.
   - Default: `false` (boolean) or first variant (multivariate).
   - Initial rollout: **0%** to "everyone".
   - Targeting rules: by `org_id` cohort if internal-only first.
   - Toggle "Persist flag across authentication" → ON.

2. **Add to the typed flag registry.**

   ```ts
   // packages/shared/flags/registry.ts
   export const flags = {
     'studio.ai-suggest-next-node': {
       key: 'flag.studio.ai-suggest-next-node',
       default: false,
       owner: '@yahyaxbt',
       removeBy: '2026-09-30',
       description: 'AI suggests next workflow node based on canvas context',
     },
     // ...
   } as const;

   export type FlagKey = keyof typeof flags;
   ```

3. **Read the flag — server side**:

   ```ts
   import { isFlagEnabled } from '@teskel/shared/flags/server';

   const enabled = await isFlagEnabled('studio.ai-suggest-next-node', {
     userId: user.id, orgId: org.id,
   });
   ```

   Always pass identity (`userId` + `orgId`) — flag evaluation must be
   sticky per user.

4. **Read the flag — client side**:

   ```tsx
   'use client';
   import { useFlag } from '@teskel/shared/flags/client';
   const enabled = useFlag('studio.ai-suggest-next-node');
   ```

   Wrapper handles SSR + bootstrap (no flicker on first paint — the
   server snapshot is hydrated into the client). Never call PostHog SDK
   directly.

5. **Branch the code.** Keep both paths green; never let a flag-OFF
   path bitrot.

   ```tsx
   {enabled ? <SuggestNextNode /> : null}
   ```

   For backend, prefer guard-and-fallthrough:

   ```ts
   if (await isFlagEnabled('billing.metering-v2', ctx)) {
     return runMeteringV2(...);
   }
   return runMeteringV1(...);
   ```

6. **Telemetry.** Emit `flag.exposed { key, variant }` automatically
   via the wrapper. Downstream events should include the variant so
   funnels can be split by flag.

7. **Rollout plan.** Document in PR description:

   ```
   Flag: flag.studio.ai-suggest-next-node
   Default: OFF
   Rollout:
     - Week 1: internal org (acme-internal) only
     - Week 2: 5% of beta cohort
     - Week 3: 25% of beta cohort
     - Week 4: 100% of beta cohort
     - Week 5: 100% of all users → schedule flag removal
   Kill criteria: error rate >1% on flag-ON traffic OR p95 latency
                  +30% vs flag-OFF.
   ```

8. **Kill-switch monitoring.** Add a Grafana panel that compares
   key metrics (error rate, latency, conversion) split by flag value.
   On-call runbook entry: how to flip it OFF in PostHog.

9. **Removal.** Every flag is debt. The registry has `removeBy`. CI
   warns when `today > removeBy` and fails when `today > removeBy + 30
   days`. Removal PR:
   - Delete the flag in PostHog (or archive).
   - Inline the winning path; delete the loser.
   - Remove from registry.
   - Update any dashboards.

10. **Tests.** Both paths must have tests, until the flag is
    permanent. Use `mockFlag('studio.ai-suggest-next-node', true)` /
    `false` helper in Vitest + Playwright.

## When NOT to flag

- Pure refactor with identical behavior.
- Bug fix that should ship to 100% immediately.
- Marketing copy change.
- Phase-aligned, well-tested, low-risk feature.

For these, just ship.

## When you MUST flag

- Phase-ahead work (current-phase says you can't ship this yet).
- Risky migration (new billing path, new AI model, new query plan).
- A/B experiment.
- Feature gated by plan tier (Pro/Team/Enterprise).
- Anything compliance-sensitive that needs gradual rollout.

## Pitfalls

- Reading PostHog SDK directly without the wrapper — bypasses
  identity, telemetry, and SSR snapshot.
- Flag without `removeBy` — accumulates debt.
- Flag-OFF path bitrots — when you finally remove, you can't switch
  back.
- Boolean flag for what should be a config value — flags are
  experiments, not config.
- Different flag values across server + client for the same user —
  causes UI flicker. Always evaluate on the server first and pass
  down.

## Done when

- Flag exists in PostHog (default OFF, scoped targeting).
- Registered in `packages/shared/flags/registry.ts` with `removeBy`.
- Both code paths exist and are tested.
- Rollout + kill plan in PR description.
- Grafana panel + runbook entry added.
