---
name: add-block
description: Add a new block to the Puck visual builder library used by TESKEL apps. Includes Puck config, props schema, render component, default props, locked editing semantics, render-on-publish, and tests. Use when extending the no-code page builder.
---

# add-block — add a Puck visual builder block

> Plan ref: [Sec. 28 (Visual builder)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#28-visual-builder--puck--craftjs-fallback),
> [Sec. 90 (Phase 2 walkthrough)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#90-phase-2--visual-builder--sandbox-minggu-1116).
>
> **Prereqs:** ADR for using Puck must already be merged
> (`docs/adr/0006-visual-builder-puck.md`). Block source lives in
> `packages/builder-blocks/`.

A Puck block has:

- A **render component** (the actual UI rendered on publish + in
  preview).
- A **fields config** (Puck inspector panel — text/number/select/array
  fields the user edits).
- A **default props** value (when the user drops the block on canvas).
- An optional **resolveData** (server-side resolver for live data
  bindings, e.g., a query).

## Steps

1. **Decide the block lives in shared or app-local.**
   - Shared (used by multiple sites): `packages/builder-blocks/src/<name>/`.
   - App-local (one app only): `apps/web/src/builder/blocks/<name>/`.

2. **Pick a `name` and slug.**
   - Display name: `Hero (centered)` (human-readable for the palette).
   - Slug: `hero-centered` (kebab-case; used as Puck `type`).

3. **Define the props schema.** Zod is the source of truth so we can
   validate the saved JSON on read, and import to TS types.

   ```ts
   // packages/builder-blocks/src/hero-centered/schema.ts
   import { z } from 'zod';

   export const HeroCenteredProps = z.object({
     title: z.string().min(1).max(120),
     subtitle: z.string().max(240).optional(),
     ctaLabel: z.string().max(40).optional(),
     ctaHref: z.string().url().optional(),
     align: z.enum(['left', 'center', 'right']).default('center'),
     theme: z.enum(['light', 'dark']).default('light'),
   });
   export type HeroCenteredProps = z.infer<typeof HeroCenteredProps>;
   ```

4. **Implement the render component.** Render must be safe for both
   editor preview and publish. **No external network calls inside
   render.** All data must be resolved up-front via `resolveData`.

   ```tsx
   // packages/builder-blocks/src/hero-centered/render.tsx
   import { cn } from '@teskel/ui/lib/cn';
   import type { HeroCenteredProps } from './schema';

   export function HeroCentered({ title, subtitle, ctaLabel, ctaHref, align, theme }: HeroCenteredProps) {
     return (
       <section
         data-block="hero-centered"
         className={cn(
           'py-24',
           theme === 'dark' ? 'bg-slate-900 text-white' : 'bg-background text-foreground',
           align === 'left' && 'text-left',
           align === 'center' && 'text-center',
           align === 'right' && 'text-right',
         )}
       >
         <div className="container mx-auto max-w-3xl">
           <h1 className="text-4xl font-bold">{title}</h1>
           {subtitle ? <p className="mt-4 text-lg opacity-80">{subtitle}</p> : null}
           {ctaLabel && ctaHref ? (
             <a className="btn btn-primary mt-6" href={ctaHref}>
               {ctaLabel}
             </a>
           ) : null}
         </div>
       </section>
     );
   }
   ```

5. **Define the Puck config (`fields`, `defaultProps`).**

   ```ts
   // packages/builder-blocks/src/hero-centered/puck.ts
   import type { ComponentConfig } from '@measured/puck';
   import { HeroCentered } from './render';
   import { HeroCenteredProps } from './schema';

   export const heroCentered: ComponentConfig<HeroCenteredProps> = {
     fields: {
       title: { type: 'text' },
       subtitle: { type: 'textarea' },
       ctaLabel: { type: 'text' },
       ctaHref: { type: 'text' },
       align: { type: 'select', options: [
         { label: 'Left', value: 'left' },
         { label: 'Center', value: 'center' },
         { label: 'Right', value: 'right' },
       ] },
       theme: { type: 'radio', options: [
         { label: 'Light', value: 'light' },
         { label: 'Dark', value: 'dark' },
       ] },
     },
     defaultProps: {
       title: 'Build something great',
       align: 'center',
       theme: 'light',
     },
     render: (props) => <HeroCentered {...props} />,
     // Optional: server-side data resolver. Use only if the block
     // needs live data (counts, queries). Render must remain pure.
     // resolveData: async (props, { metadata }) => ({ ...props }),
   };
   ```

6. **Register in the block index.**

   ```ts
   // packages/builder-blocks/src/index.ts
   import { heroCentered } from './hero-centered/puck';
   // ... other blocks

   export const teskelBlocks = {
     'hero-centered': heroCentered,
     // ...
   } as const;

   export type TeskelBlockType = keyof typeof teskelBlocks;
   ```

7. **Add to the palette config.** The editor uses category groupings
   defined in `apps/web/src/builder/palette.ts`. Add your block under
   the correct category (Layout / Content / Media / CTA / Form / Data
   / Custom).

8. **Validate saved data on read.** The page renderer reads JSON from
   the DB; always parse with the Zod schema before render.

   ```ts
   const props = HeroCenteredProps.parse(node.props);
   ```

   Drop or migrate any block whose schema fails to parse — never throw
   at the page level. Add a fallback "[block missing]" placeholder for
   editors.

9. **Migration helper.** If you change the schema in a non-additive
   way:
   - Bump a `version` field on the block: `{ type: 'hero-centered', v: 2, props: {...} }`.
   - Write a migrator in `packages/builder-blocks/src/migrate.ts` that
     up-converts v1 → v2 on read.
   - Never mutate stored docs; produce migrated objects in memory.

10. **Tests.**
    - Schema: Zod accepts valid props, rejects invalid ones.
    - Render: snapshot for each variant of `align` × `theme`.
    - A11y axe test on render.
    - Storybook story (Default + Dark + No-CTA).

11. **Documentation.** Add a section in `apps/docs/content/builder/blocks/`
    with screenshot, props table, and 1 example use case.

12. **Telemetry.**
    - On block insertion, the editor emits `builder.block.inserted`
      with `{ type: 'hero-centered' }` automatically — verify the new
      type is captured by checking the type union in
      `packages/shared/analytics/events.ts`.

## Pitfalls

- Reading from `localStorage` / `window` inside `render` — breaks SSR.
- Calling `fetch` in `render` — use `resolveData` instead, or hoist to
  the page.
- Hard-coded copy in `defaultProps` that isn't translatable — wrap
  with `t()` or accept that it's an editor-time placeholder.
- Including heavy deps eagerly — lazy-load with `next/dynamic` for
  blocks that pull in charts, editors, or maps.
- Breaking schema without migrator — any saved page using v1 will fail
  to render.
- Forgetting to add to `palette.ts` — block exists but users can't drop
  it.

## Done when

- Schema, render, Puck config registered + tested.
- Palette + docs updated.
- Snapshot + axe tests green.
- Storybook story present.
- Migrator added if schema changed.
- `packages/builder-blocks/src/index.ts` re-exports the block.
