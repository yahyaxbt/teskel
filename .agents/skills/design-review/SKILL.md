---
name: design-review
description: Self-review checklist an agent runs against any UI change before opening a PR. Covers visual hierarchy, design tokens, dark mode, responsive breakpoints, accessibility (WCAG 2.2 AA), empty/loading/error/success states, motion, copy, and i18n. Use after add-ui-component, add-page, or add-block; or when reviewing any user-facing diff.
---

# design-review — UI/UX self-review checklist

> Plan ref: [Sec. 10–14](../../../TESKEL_FULLSTACK_BUILD_BREAKDOWN.md#10-design-system-spec).
>
> Run this every time a change touches `apps/web/`, `packages/ui/`, or
> `packages/builder-blocks/`. Skip a section only with explicit
> justification in the PR description.

## A. Visual hierarchy & layout

- [ ] One clear primary action per view; secondary actions visually
      de-emphasized.
- [ ] Headings form a logical outline (H1 → H2 → H3), no skipped
      levels.
- [ ] White space breathes — no cramped touch targets (min 24×24
      logical px, 44×44 mobile).
- [ ] Long text capped to ~70 characters per line for readability.
- [ ] Consistent spacing scale (Tailwind 4 / 8 / 12 / 16 / 24 / 32 /
      48 / 64), no magic px values.

## B. Design tokens

- [ ] No hard-coded colors (`#fff`, `text-gray-500`). Use semantic
      tokens (`bg-background`, `text-muted-foreground`, `border-border`,
      `ring-ring`).
- [ ] Border radius from token scale (`rounded-sm`, `rounded-md`,
      `rounded-lg`).
- [ ] Shadows from token scale (`shadow-sm`, `shadow-md`).
- [ ] Typography uses token classes (`text-sm`, `text-base`, `text-lg`,
      `font-medium`, `font-semibold`).
- [ ] Icon size from `size-4 / size-5 / size-6`, not arbitrary values.

## C. Dark mode

- [ ] Toggle dark mode (`next-themes`); UI still readable.
- [ ] No "white flash" on initial load — dark theme applied before
      first paint.
- [ ] Images / illustrations have a dark variant or a tinted overlay.
- [ ] Charts use chart-aware tokens (the Recharts/Tremor wrapper
      already maps these).
- [ ] Status colors (success/error/warning/info) preserve ≥4.5:1
      contrast in both modes.

## D. Responsive

Test at: **320, 360, 768, 1024, 1280, 1440** widths.

- [ ] No horizontal scroll at 320px.
- [ ] Long titles wrap or truncate with `text-ellipsis` (never overflow
      silently).
- [ ] Tables collapse to cards or scroll horizontally on mobile.
- [ ] Sticky headers don't cover focused inputs.
- [ ] Modals fit in viewport on mobile (use sheet pattern below 768).
- [ ] Touch targets ≥44×44 px on mobile.

## E. States — every interactive component

| State | Required behavior |
| --- | --- |
| Default | As designed. |
| Hover | Subtle but visible cue (color/elevation). |
| Focus-visible | Visible ring (≥3:1 contrast); never `outline:none` without replacement. |
| Active / pressed | Slight visual depression. |
| Disabled | `aria-disabled`, reduced contrast, no hover. |
| Loading | Skeleton or spinner; **preserves dimensions** (no layout shift). |
| Empty | Friendly illustration + 1 CTA. No "No data" alone. |
| Error | `role="alert"`, suggested next action, retry where reasonable. |
| Success | Toast or inline confirm; auto-dismiss 5s. |

## F. Accessibility (WCAG 2.2 AA)

- [ ] Semantic HTML before ARIA (`button`, not `div onClick`).
- [ ] Every form input has a `<label>` (or `aria-label`).
- [ ] Errors associated via `aria-invalid` + `aria-describedby`.
- [ ] Keyboard-only walkthrough: every action reachable; tab order is
      logical; focus is visible at every step.
- [ ] Esc closes modals/drawers; trap focus inside while open; restore
      focus on close.
- [ ] Color is not the sole carrier of meaning (status icons + text).
- [ ] Color contrast ≥4.5:1 (text), ≥3:1 (UI).
- [ ] Live regions (`aria-live="polite"`) for async updates.
- [ ] No `tabindex` > 0.
- [ ] Animations respect `prefers-reduced-motion`.
- [ ] Page lang set (`<html lang="en">`).
- [ ] axe automated scan reports zero violations.

## G. Motion & feedback

- [ ] Animations 150–300ms; easing `ease-out` on enter, `ease-in` on
      exit.
- [ ] No "snake-charmer" infinite loops without purpose.
- [ ] Optimistic UI for fast interactions (delete, toggle).
- [ ] Slow ops (>1s) show progress, not just a spinner.
- [ ] After mutation: clear success or error toast; never silent.

## H. Copy

- [ ] Tone is direct and professional; no exclamation marks; no
      condescending copy.
- [ ] Errors describe what went wrong + how to fix
      ("Email is required" vs. "Invalid input").
- [ ] No raw enum values shown to users (`PENDING_REVIEW` →
      "Pending review").
- [ ] Empty-state CTAs are imperative ("Create project", not "You can
      create a project here").
- [ ] No copy hard-coded outside `packages/i18n` — all wrapped in
      `t()`.

## I. i18n

- [ ] Strings in `packages/i18n/locales/en.json` (canonical) with
      stable keys.
- [ ] Plurals via ICU MessageFormat, not string concatenation.
- [ ] Date/number/currency formatted via `Intl.*` with locale param.
- [ ] RTL safe — use `start`/`end` instead of `left`/`right` (Tailwind
      logical properties).
- [ ] No truncation of long German / Japanese strings — verify with
      pseudo-locale (`en-XA` if configured).

## J. Performance

- [ ] LCP ≤2.5s on 4G throttle.
- [ ] INP ≤200ms.
- [ ] CLS ≤0.1 (Lighthouse local + checked in PR preview).
- [ ] Above-the-fold images use `next/image` with `priority` and
      explicit `sizes`.
- [ ] Below-the-fold heavy components lazy-loaded
      (`next/dynamic({ ssr: false })` only when truly client-only).
- [ ] No third-party script blocks render. If unavoidable, defer.

## K. Privacy & analytics

- [ ] No PII in event payloads (no email, no full name unless
      consented).
- [ ] Cookie banner respected — if user opts out, PostHog disabled.
- [ ] Sentry session replay masks input fields by default; verify any
      newly added inputs aren't accidentally captured.

## L. Brand consistency

- [ ] Logo, favicon, social meta, OG image all updated if route is
      shareable.
- [ ] Page `title` and `description` set; OG image uses
      `next/og` template.
- [ ] Light/dark variants of brand assets present.

## M. Test it as a user

Walk through the flow as the persona it's built for:

- [ ] First-time user (no data, no muscle memory) — discoverable?
- [ ] Returning user (lots of data) — fast & uncluttered?
- [ ] Mobile user — every primary action reachable with one thumb?
- [ ] Slow network — degrades gracefully?
- [ ] Screen reader — try VoiceOver or NVDA on at least one critical
      flow.

## Sign-off

If this checklist is fully green, paste this in the PR description:

```
design-review: PASS
- visual: ok
- tokens: ok
- dark mode: ok
- responsive: 320/768/1024/1440 ok
- a11y: axe 0 violations, keyboard ok, sr ok
- motion / copy / i18n / perf / privacy / brand: ok
```

If anything is intentionally skipped, replace the relevant line with
`SKIPPED: <reason>` and tag a designer/owner.
