---
name: add-workflow-node
description: Register a new workflow node type end-to-end across the React Flow studio (UI), runner (execution), validator (Zod schema), prompt registry slot (if AI), and template manifest spec for TESKEL. Use whenever the workflow runtime needs a new step kind.
---

# add-workflow-node — register a new workflow node type

> Workflow nodes are the unit of execution in the TESKEL workflow
> runtime. Adding one cleanly requires touching the studio (UI), the
> runner (execution), the schema (validator), and possibly the prompt
> registry. This skill walks through all of it.
>
> Plan ref: [Sec. 24 (Workflow Runtime)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#24-workflow-runtime),
> [Sec. 23 (AI Infrastructure)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#23-ai-infrastructure).

## Built-in node types (already exist)

`start`, `llm`, `http`, `branch`, `code`, `loop`, `wait`, `email`,
`webhook`, `kb_query`, `end`. Reuse these unless the new behavior is
genuinely distinct.

## When to use

- The new behavior is not a parameter variation of an existing node.
- The behavior has its own UI affordances in the canvas (icon, props
  panel, validation rules).
- The behavior is reusable across templates (otherwise put it in a
  template-specific code node).

## Steps

1. **Pick an id.** Lowercase, snake_case, ≤16 chars (e.g.,
   `slack_post`, `pdf_render`, `ocr_extract`).

2. **Define the Zod schema** in
   `packages/runner/src/nodes/<id>/schema.ts`:

   ```ts
   import { z } from 'zod';

   export const slackPostNodeSchema = z.object({
     type: z.literal('slack_post'),
     id: z.string(),
     channel: z.string().regex(/^#?[a-z0-9-]+$/),
     message: z.string().max(4000),
     thread_ts: z.string().optional(),
     mention: z.array(z.string()).default([]),
   });

   export type SlackPostNode = z.infer<typeof slackPostNodeSchema>;
   ```

3. **Implement the executor** in
   `packages/runner/src/nodes/<id>/execute.ts`:

   ```ts
   import { NodeContext, NodeResult } from '../types';
   import { slackPostNodeSchema } from './schema';

   export async function executeSlackPost(
     ctx: NodeContext,
     raw: unknown,
   ): Promise<NodeResult> {
     const node = slackPostNodeSchema.parse(raw);
     const start = Date.now();
     try {
       // 1. Resolve credentials via ctx.credentials (Org-scoped, RLS aware).
       // 2. Apply rate limit + idempotency-key from ctx.runId+node.id.
       // 3. Call external API. Wrap in retry policy ctx.retry.
       // 4. Emit telemetry: ctx.span(name, attrs).
       const result = await postToSlack(node, ctx);
       return { ok: true, output: result, duration_ms: Date.now() - start };
     } catch (err) {
       ctx.logger.error({ err, nodeId: node.id }, 'slack_post.failed');
       return { ok: false, error: serialize(err), duration_ms: Date.now() - start };
     }
   }
   ```

4. **Register the executor** in `packages/runner/src/nodes/index.ts`:

   ```ts
   import { executeSlackPost } from './slack_post/execute';
   import { slackPostNodeSchema } from './slack_post/schema';

   export const nodeRegistry = {
     // ...existing...
     slack_post: { execute: executeSlackPost, schema: slackPostNodeSchema },
   } as const;
   ```

5. **Add the studio UI component** in
   `apps/web/src/features/workflow/nodes/slack-post-node.tsx`:

   - Custom React Flow node (icon, label, status pill).
   - Properties panel (form using React Hook Form + Zod resolver,
     reusing the schema from step 2).
   - Validation: highlight errors inline.

6. **Add to the palette** in
   `apps/web/src/features/workflow/palette.ts`:

   ```ts
   { id: 'slack_post', label: 'Slack Post', icon: SlackIcon, group: 'integrations' }
   ```

7. **If the node calls an LLM:**
   - Add a prompt slot in `packages/ai/src/prompts/slots.ts`.
   - Use `getPrompt(slot)` inside the executor.
   - Add a Promptfoo eval at `evals/prompts/<slot>.yaml`.
   - Tracing: emit a Langfuse span tagged with `node_type=<id>`.

8. **If the node touches an external service:**
   - Define credentials shape in `packages/shared/credentials/<id>.ts`.
   - Add to the credentials UI (`apps/web/src/features/credentials/`).
   - Wrap calls with idempotency-key, retry policy (only if the action
     is idempotent), and circuit breaker.

9. **Add tests:**
   - Schema test (golden inputs + invalid inputs).
   - Executor unit test (happy path + 1 failure path) using mocked
     external client.
   - Integration test in `packages/runner/test/integration/` running the
     node inside a real BullMQ worker.

10. **Update docs:**
    - Add a row to the Plan Sec. 24 nodes table (or amend if Sec. 24
      already lists the table).
    - Add a doc page at `apps/docs/content/workflow/nodes/<id>.mdx` —
      includes example JSON, properties, retry semantics, errors.
    - Add a row to the manifest spec at Plan Appendix C if templates
      need to reference it.

11. **Telemetry:**
    - Emit `workflow.node.executed` with `{ node_type, duration_ms,
      ok }` on every run.
    - Add a Grafana panel (or amend an existing one) showing p50/p95
      latency and error rate per node type.

12. **PR conventions:**
    - Title: `feat(workflow): add <id> node`.
    - Mark feature flag `workflow.node.<id>` (default OFF) so we can
      gate rollout.

## Pitfalls

- Forgetting to add the schema to the registry → runtime crash.
- Forgetting `'use client'` on the React Flow node component.
- Calling external APIs without retry / idempotency → duplicate side
  effects on retry.
- Missing Langfuse trace for LLM calls → ungrokkable workflow runs.
- Renaming the node id later → breaks every existing workflow JSON.
  Pick a stable id once.

## Done when

- Schema, executor, UI, palette, tests, docs, and telemetry all present.
- Feature flag added.
- One example workflow exists in `seeds/templates/<demo>/workflows/`
  using the new node.
- CI green.
