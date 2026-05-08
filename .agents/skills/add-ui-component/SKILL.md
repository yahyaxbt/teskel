---
name: add-ui-component
description: Scaffold a new shadcn/ui-style React component in packages/ui or apps/web for TESKEL. Enforces Tailwind design tokens, dark mode, accessibility (WCAG 2.2 AA), keyboard nav, and consistent variants. Use when adding any new visual primitive used in 2+ places.
---

# add-ui-component — add a UI component

> Use this for any visual primitive that will be reused (button variant,
> form field, modal, table cell, badge, callout). Single-use markup
> stays in the feature folder.
>
> Plan ref: [Sec. 10 (Design System)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#10-design-system-spec),
> [Sec. 13 (Empty/Error/Loading)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#13-empty--error--loading-patterns),
> [Sec. 14 (Accessibility)](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#14-accessibility-wcag-22-aa).

## Decide where it lives

| Where | When |
| --- | --- |
| `packages/ui/src/components/<name>.tsx` | Reusable across apps; brand-neutral; data-agnostic |
| `apps/web/src/components/<name>.tsx` | Web-app-specific composition (uses TanStack Query, business state) |
| `apps/web/src/features/<feature>/<name>.tsx` | Single-feature only (do not reach for "ui component" skill here) |

Default: `packages/ui` if shared, otherwise `apps/web/src/components`.

## Steps

1. **Check for existing.** Search shadcn/ui registry, `packages/ui/`, and
   `apps/web/src/components/` first. Don't duplicate.

2. **Pick a name.** PascalCase, no abbreviations
   (`OrgSwitcher`, not `OrgSw`). File: `kebab-case.tsx`
   (`org-switcher.tsx`).

3. **Scaffold the component.**

   ```tsx
   // packages/ui/src/components/org-switcher.tsx
   'use client';

   import * as React from 'react';
   import { cva, type VariantProps } from 'class-variance-authority';
   import { cn } from '@teskel/ui/lib/cn';

   const orgSwitcherVariants = cva(
     'inline-flex items-center gap-2 rounded-md border bg-background text-sm transition focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring',
     {
       variants: {
         size: {
           sm: 'h-8 px-2',
           md: 'h-9 px-3',
           lg: 'h-10 px-4',
         },
         tone: {
           default: 'border-border text-foreground hover:bg-muted',
           subtle: 'border-transparent text-muted-foreground hover:bg-muted',
         },
       },
       defaultVariants: { size: 'md', tone: 'default' },
     },
   );

   export interface OrgSwitcherProps
     extends React.ButtonHTMLAttributes<HTMLButtonElement>,
       VariantProps<typeof orgSwitcherVariants> {
     orgName: string;
     onOpenChange?: (open: boolean) => void;
   }

   export const OrgSwitcher = React.forwardRef<HTMLButtonElement, OrgSwitcherProps>(
     function OrgSwitcher({ className, size, tone, orgName, ...props }, ref) {
       return (
         <button
           ref={ref}
           type="button"
           aria-haspopup="menu"
           className={cn(orgSwitcherVariants({ size, tone }), className)}
           {...props}
         >
           <span className="font-medium">{orgName}</span>
           <ChevronDownIcon className="size-4 opacity-60" aria-hidden />
         </button>
       );
     },
   );
   ```

4. **Use design tokens, never raw colors.** All color/spacing/radius
   come from CSS variables defined in `packages/ui/src/styles/tokens.css`
   and exposed in `tailwind.config.ts`. If you need a token that does
   not exist, add it once + ADR if cross-cutting.

5. **Mandatory states.** Every interactive component renders correctly
   in:
   - Default
   - Hover
   - Focus-visible (visible ring; do **not** disable focus outline)
   - Active / pressed
   - Disabled (`aria-disabled`, not just CSS opacity)
   - Loading (skeleton or spinner; preserve dimensions)
   - Empty (when applicable, e.g., empty list inside a panel)
   - Error (e.g., form field shows `aria-invalid`)
   - Dark mode (verify in Storybook + `next-themes`)

6. **Accessibility checklist (WCAG 2.2 AA).**
   - Semantic HTML first (`button`, not `div onClick`).
   - Keyboard reachable; logical tab order.
   - Visible focus ring (≥3:1 contrast vs background).
   - `aria-*` only when semantics are missing.
   - Color contrast ≥4.5:1 for text, ≥3:1 for UI elements.
   - Hit target ≥24×24 logical px (mobile ≥44×44).
   - Reduced-motion respected (`prefers-reduced-motion`).
   - Screen-reader text via `<span class="sr-only">` when icons are
     standalone.
   - Forms: every input has a label; errors announced via `role="alert"`
     or `aria-live`.

7. **Responsive.** Default to mobile-first; add `sm:`, `md:`, `lg:`,
   `xl:` Tailwind variants. Test at 320, 768, 1024, 1440 widths.

8. **Storybook story.**

   ```tsx
   // packages/ui/src/components/org-switcher.stories.tsx
   import { Meta, StoryObj } from '@storybook/react';
   import { OrgSwitcher } from './org-switcher';

   const meta: Meta<typeof OrgSwitcher> = {
     component: OrgSwitcher,
     args: { orgName: 'Acme Inc' },
     parameters: { a11y: { test: 'error' } },
   };
   export default meta;

   export const Default: StoryObj<typeof OrgSwitcher> = {};
   export const Subtle: StoryObj<typeof OrgSwitcher> = { args: { tone: 'subtle' } };
   export const Loading: StoryObj<typeof OrgSwitcher> = { args: { 'aria-busy': true } };
   export const Disabled: StoryObj<typeof OrgSwitcher> = { args: { disabled: true } };
   ```

9. **Tests.**
   - Unit (Vitest + React Testing Library):
     - Renders all variants.
     - Forwards `ref`.
     - Disabled state blocks `onClick`.
     - Calls callbacks on user events.
   - A11y axe test:
     ```ts
     import { axe } from 'vitest-axe';
     it('has no a11y violations', async () => {
       const { container } = render(<OrgSwitcher orgName="x" />);
       expect(await axe(container)).toHaveNoViolations();
     });
     ```

10. **Document.** Add a short page at
    `apps/docs/content/ui/<name>.mdx` with: when to use, when not to
    use, props table, examples (default/subtle/disabled/loading), and
    accessibility notes.

11. **Update the design system index** in
    `packages/ui/src/index.ts` to re-export the component.

## Pitfalls

- Hard-coded colors (`#fff`, `text-gray-500`) — use tokens
  (`text-muted-foreground`, `bg-background`).
- Forgetting `'use client'` for interactive components in the App
  Router.
- Disabling focus rings — never. Style them, don't remove them.
- Mixing concerns — UI components don't fetch, don't read auth, don't
  read URL state. Pass data as props.
- Skipping the dark mode pass — at least verify in Storybook with the
  `dark` toolbar toggle.
- Using `index.ts` barrel re-exports that pull in heavy deps eagerly —
  keep `packages/ui/src/index.ts` lightweight; deep-import for heavy
  components.

## Done when

- Component implemented with all variants, states, and a11y.
- Storybook story renders without warnings; axe shows zero violations.
- Vitest unit tests + a11y test green.
- Tokens used everywhere (no hard-coded colors).
- Docs page added.
- `packages/ui/src/index.ts` updated.
