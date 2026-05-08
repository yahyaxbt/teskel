---
name: add-prompt-slot
description: Register a new prompt slot in the TESKEL prompt registry — version it, write Promptfoo evals (regression + golden set), wire to the AI gateway with model routing/fallbacks, capture Langfuse traces, and add cost/latency budget. Use when introducing any new LLM call.
---

# add-prompt-slot — register a new LLM prompt

> Plan ref: [Sec. 25 (AI gateway)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#25-ai-gateway--llm-routing),
> [Sec. 49 (AI guardrails)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#49-ai-guardrails),
> [Sec. 89 (Phase 1 walkthrough)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#89-phase-1--core-workflow--ai-minggu-510).
>
> **Hard rule:** All LLM calls go through `getPrompt(slot)` +
> `aiGateway.run(...)`. Inline prompt strings in product code are
> forbidden (AGENTS.md §10).

A prompt slot is a versioned, named LLM contract:

- A **slot id** (`workflow.summarize-doc`, `marketplace.review-listing`).
- An **input schema** (Zod).
- An **output schema** (Zod, often JSON).
- A **prompt template** (system + user messages, with variables).
- A **default model + fallbacks** + **budget** (max tokens, max cost).
- An **eval set** (Promptfoo) defining regressions.

## Steps

1. **Pick a slot id.** `<domain>.<verb-object>` lowercase, kebab-case.
   Examples: `workflow.summarize-doc`, `studio.suggest-next-node`.

2. **Define schemas** in `packages/ai/prompts/<slot>/schema.ts`:

   ```ts
   import { z } from 'zod';

   export const Input = z.object({
     document: z.string().min(1).max(20_000),
     locale: z.enum(['en', 'id']).default('en'),
   });
   export type Input = z.infer<typeof Input>;

   export const Output = z.object({
     summary: z.string().min(1).max(2_000),
     keyPoints: z.array(z.string()).max(10),
     language: z.string(),
   });
   export type Output = z.infer<typeof Output>;
   ```

3. **Write the template** (`packages/ai/prompts/<slot>/v1/prompt.ts`).
   Templates are **versioned per directory** — you never edit a
   shipped version, you create `v2/` next to `v1/`.

   ```ts
   import { definePrompt } from '@teskel/ai/registry';
   import { Input, Output } from '../schema';

   export default definePrompt({
     id: 'workflow.summarize-doc',
     version: 1,
     input: Input,
     output: Output,
     model: { primary: 'openai/gpt-4o-mini', fallbacks: ['anthropic/claude-3-5-haiku'] },
     budget: { maxOutputTokens: 800, maxCostUsd: 0.01, maxLatencyMs: 4_000 },
     messages: ({ document, locale }) => [
       { role: 'system', content:
         `You are a concise document summarizer. Reply in ${locale}. ` +
         `Output strict JSON matching: { summary: string, keyPoints: string[], language: string }. ` +
         `Do not include any text outside JSON.` },
       { role: 'user', content: document.slice(0, 18_000) },
     ],
     temperature: 0.2,
     responseFormat: 'json',
     // PII redaction is applied by the gateway by default; toggle off
     // only with security-team approval.
     redactPII: true,
   });
   ```

4. **Register the slot.**

   ```ts
   // packages/ai/prompts/index.ts
   export { default as workflowSummarizeDoc } from './workflow.summarize-doc/v1/prompt';
   ```

5. **Use it via the gateway.** Product code never knows about the
   model — only the slot.

   ```ts
   // some feature
   import { aiGateway } from '@teskel/ai';
   const result = await aiGateway.run('workflow.summarize-doc', {
     input: { document, locale: 'id' },
     traceContext: { orgId, userId, projectId, traceId },
   });
   // result is typed as Output
   ```

6. **Promptfoo eval.** Every slot must have a `promptfoo.yaml` next to
   the prompt with at minimum 10 test cases, covering: happy path,
   short input, long input, multi-language, edge tokens, refusal /
   unsafe input, JSON-only enforcement, locale switch.

   ```yaml
   # packages/ai/prompts/workflow.summarize-doc/v1/promptfoo.yaml
   description: Summarize document
   prompts: [./prompt.ts]
   providers:
     - openai:gpt-4o-mini
     - anthropic:claude-3-5-haiku
   tests:
     - vars: { document: "...", locale: "en" }
       assert:
         - type: is-json
         - type: javascript
           value: 'JSON.parse(output).summary.length > 30 && JSON.parse(output).keyPoints.length > 0'
     - vars: { document: "<5k char input>", locale: "id" }
       assert:
         - type: is-json
         - type: javascript
           value: 'JSON.parse(output).language.toLowerCase().startsWith("in") || JSON.parse(output).language === "id"'
     # ...8 more
   ```

   Run `pnpm --filter @teskel/ai eval workflow.summarize-doc` locally.
   CI runs it on PRs that touch `packages/ai/`.

7. **Budgets.** The gateway enforces `maxOutputTokens`, `maxCostUsd`,
   `maxLatencyMs` per call. Exceeding any budget raises a typed error
   `BudgetExceededError`. Calling code must handle it gracefully.

   Set realistic budgets — measure with `eval --record-cost` first.

8. **Routing & fallbacks.** Provide `primary` + 1–2 `fallbacks`. Order
   matters; the gateway tries in order on retryable errors (rate-limit,
   5xx, timeout). Choose fallbacks with similar capability tier.

9. **Caching (optional).** If the prompt is deterministic (low temp,
   stable input), enable `cache: { keyFields: ['document','locale'], ttlSeconds: 3600 }`.

10. **Langfuse trace.** The gateway auto-traces. Add custom tags for
    business context: `traceContext.tags = ['workflow', 'summarize']`.
    Verify in Langfuse UI that the trace shows up under the right
    project + has input/output redacted of PII.

11. **Guardrails.** All slots inherit:
    - PII redaction (in/out).
    - Profanity / illegal-content filter.
    - Output JSON validation against `Output` schema; on failure, retry
      once with a "your previous output was invalid JSON, fix it"
      repair turn; if still invalid, raise `OutputInvalidError`.
    - Maximum input length cap (per-slot).

12. **Tests.**
    - Vitest unit: schema parse rejects bad input/output.
    - Promptfoo eval: passes ≥90% of cases on primary model and ≥80%
      on fallback.
    - Cost regression: median cost ≤ slot's `maxCostUsd × 0.5`.

13. **Telemetry events.** Slot calls auto-emit
    `ai.slot.invoked { slot, model, durationMs, costUsd, success }`.
    Verify dashboards in Grafana cover the new slot (auto-generated by
    template).

14. **Document.** Add a short page in
    `apps/docs/content/ai/prompts/<slot>.mdx` describing intent,
    inputs, outputs, expected use, and known failure modes.

## Versioning rules

- Never mutate an existing `v<n>/` directory once shipped.
- Create `v2/` for breaking changes; gateway routes traffic via the
  feature-flag `flag.ai.<slot>.version` (default to v1; ramp to v2
  after eval + canary).
- Deprecate old versions only after telemetry shows zero callers for
  ≥14 days; then delete and tombstone in CHANGELOG.

## Pitfalls

- Inline prompt strings outside the registry — forbidden.
- Calling provider SDK directly (`openai.chat.completions.create`) —
  forbidden; always go through `aiGateway`.
- Setting `temperature: 1` or higher without justification — bad cost
  and reliability.
- No JSON schema enforcement — model drifts and breaks parsers.
- Forgetting to provide a fallback model — single-vendor outage takes
  the feature down.
- Eval suite with <10 cases — regression risk on small upgrades.
- PII not redacted — compliance violation.

## Done when

- Schemas + template + registry registration committed.
- Promptfoo eval suite passes locally.
- Budgets defined and validated against eval cost report.
- Langfuse trace shows up correctly.
- Tests green; docs page added.
- All product callers go through `aiGateway.run(slot, ...)`.
