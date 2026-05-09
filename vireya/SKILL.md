---
name: vireya
description: Use when building or modifying React UIs with Vireya (`@vireya/core`, `@vireya/ui`, `@vireya/next`, `@vireya/blocks`). Triggers on "vireya", "@vireya/*", or asks to build a landing/marketing/dashboard page with these packages.
---

# Vireya

Token-first React design system on Next.js. You consume `@vireya/*` from npm — no source access needed. The DTS at `node_modules/@vireya/{ui,blocks}/dist/components/<group>/<name>/index.d.ts` is the source of truth for props.

## How to use this skill

This file is a **dispatcher**, not a manual. It carries the rules + a token cheat sheet + a routing table to references. Don't read every reference upfront — load only the one you need for the current task.

- For a component's props → open the DTS, not a reference.
- For setup, gotchas, or a copy-paste recipe → open the matching `references/<name>.md`.
- For comprehensive offline usage (no network) → `node_modules/@vireya/ui/LLMS.md` is bundled with the package.

## Non-negotiable rules

1. **Tokens are absolute.** Every CSS visual property → `var(--v-*)`. No hardcoded `px`, `#hex`, `rgb()`, `rgba()`, numeric `font-weight`, or font-family literals. Allowed: `0`, `100%`, `1px`/`1.4px` for inline-SVG hairlines, `currentColor` in icons, `%` inside `calc()`. Pill/circle uses `--v-radius-full`. If a token is missing for your case, override locally via inline `style` — don't invent literals.

2. **`primary` ≠ `accent`.** primary = neutral-strong (text, default CTAs, default borders). accent = brand-vivid (promo CTAs, "Popular" badge, focus rings, eyebrow dots), **optional**, falls back to primary. Always chain accent fallbacks in CSS:
   ```css
   background-color: var(--v-accent-fill, var(--v-primary-fill));
   color:            var(--v-accent-foreground, var(--v-primary-foreground));
   border-color:     var(--v-accent-border, var(--v-primary-border));
   ```
   Components exposing `variant="accent"`: **`Button` and `Badge` only**.

3. **Pin `react@19.2.1` exactly** (and `react-dom`). Vireya peers React 19 on every package. Mismatches cause duplicate-React runtime errors — `pnpm dedupe` if your tree hoists multiple.

4. **Per-component subpath imports only.** `import { Button } from "@vireya/ui/form/button"`. The **only** allowed root import is `import { AppProvider } from "@vireya/ui"`. Locales are default exports: `import enUS from "@vireya/ui/locales/enUS"`. Same for blocks: `@vireya/blocks/hero/centered`.

5. **Focus + disabled.** `:focus-visible { outline: 2px solid var(--v-primary-fill); outline-offset: 2px; }` — never bare `:focus`, never `outline: none` without a replacement. Disabled = `opacity: 0.5; cursor: not-allowed;` — universal, no custom gray.

## Token cheat sheet

Full catalog in `references/tokens.md`. The 90% case:

| Property | Token |
|---|---|
| `font-size` | `var(--v-font-size-{caption,body-sm,body,body-lg,title-sm,title,title-lg,display,display-lg})` or `<Text size="...">` |
| `font-weight` | `var(--v-font-weight-{regular,medium,semibold,bold})` or `<Text weight>` |
| `font-family` | `var(--v-font-family-{sans,mono})` (mono for code/CLI/path/URL only) |
| `line-height` | `var(--v-leading-{tight,snug,normal,relaxed})` |
| `letter-spacing` | `var(--v-tracking-{tight,normal,wide,eyebrow})` |
| `padding`, `margin`, `gap`, offsets | `var(--v-{height,width}-N)` (N = 1, 2, 4, 6, 8, 10, 12, 14, 16, 18, 20, 22, 24, 32, 40, 48, 64, 80, 96, 128) |
| Outer page gutter | `var(--v-page-padding)` |
| `border-radius` | `var(--v-radius-{xs,sm,md,lg,xl,2xl,full})` |
| Colors | `var(--v-{name}-{fill,foreground,hover,active,subdued,border})` (name: `background`, `secondary`, `primary`, `accent`, `destructive`, `link`, `success`, `warning`, `info`) |
| `transition-duration` | `var(--v-motion-{instant,fast,default,slow,slower})` |
| `transition-timing-function` | `var(--v-motion-timing)` |
| `box-shadow` | `var(--v-elevation-{rest,hover,popover,modal})` |
| Inner content max-width | `var(--v-content-width-{sm,md,lg})` |
| Field/avatar/badge/icon sizing | Composites: `var(--v-{field,avatar,badge,icon}-{ss,sm,md,lg,xl}-*)` |

**Rules of thumb:**
- Children take a smaller radius than parent. Button (`md`) inside Card (`xl`).
- Muted text uses `--v-secondary-active`, **not** `--v-secondary-foreground`. `*-foreground` is reserved for "text painted ON the matching `*-fill`".
- Translucency uses `color-mix(in srgb, X N%, transparent)`, never `rgba()`.

## Packages

| Package | Purpose |
|---|---|
| `@vireya/core` | Tokens, `createTheme`, color utilities |
| `@vireya/ui` | React component primitives (8 groups: `form`, `data`, `feedback`, `overlay`, `navigation`, `layout`, `typography`, `common`) |
| `@vireya/next` | `ThemeProvider` (server), `useTheme` (`/client` subpath), motion preset CSS files (`/motion/*`), bundled tokens (`/styles`) |
| `@vireya/blocks` | Pre-composed landing-page sections (10 families, 26 blocks) |

Install: `pnpm add @vireya/core @vireya/ui @vireya/next @vireya/blocks react@19.2.1 react-dom@19.2.1 next lucide-react`. Optional peers (install only when used): `@tanstack/react-table`, `recharts`, `embla-carousel-react`, `react-day-picker` + `date-fns`, `cmdk`, `vaul`, `react-resizable-panels`, `@headless-tree/core`, `react-hook-form`. Full setup recipe in `references/setup.md` or `node_modules/@vireya/ui/LLMS.md`.

## Routing table — when to read what

| Need | Open |
|---|---|
| Setup, providers, themes, motion presets | `references/setup.md` |
| Section primitive, hero CTA pair, pricing, install snippet, scoped accent recipes | `references/recipes.md` |
| Color contrast pairing matrix, CTA decision matrix, emphasis rule, type roles, visual depth (gradients/spotlights/masks), anti-patterns gallery | `references/design.md` |
| Block sub-component shapes (`PricingTiers.Tier`, `ContactSplit.Form` types), which blocks expose `emphasis` | `references/blocks.md` |
| Full token catalog, composite tokens, every scale | `references/tokens.md` |
| Type errors, RSC traps, hydration issues, animation flickers, missing icons | `references/gotchas.md` |
| **Upgrading from a pre-0.1.0 `@vireya/*` (token API reshape, codemod, manual fixes)** | `references/migration.md` |
| Detailed props of a specific component | DTS at `node_modules/@vireya/ui/dist/components/<group>/<name>/index.d.ts` |
| Comprehensive offline reference | `node_modules/@vireya/ui/LLMS.md` (bundled in tarball) |

## Top 5 gotchas

Full list in `references/gotchas.md`. Failures that cost most time:

1. **`AppProvider` is mandatory** with the full 14-icon map + locale. Missing icons silently break `DateField`, `Calendar`, `Select` chevron, `Toast` close, `DropdownMenu` checks. See `references/setup.md`.
2. **`accent` needs fallback chain in every CSS reference.** Themes without `accent` leave those vars `undefined` → accent CTAs vanish in dark mode.
3. **No barrel imports.** `@vireya/ui` root only exports `AppProvider`. Subpath everything else.
4. **`"use client"` modules don't marshal non-component named exports across the RSC boundary.** Metadata constants come back as `undefined` in server components — bake them at build time or move to a server-only module.
5. **`<html suppressHydrationWarning>`** is required — the theme bootstrap script runs before React hydrates.

## Self-audit before declaring a page done

- [ ] No hardcoded `px`/`#hex`/`rgba()`/numeric `font-weight` in your CSS Modules (allowed: `0`, `100%`, `1px`/`1.4px` SVG hairlines).
- [ ] Every `--v-accent-*` reference is chained to a `--v-primary-*` fallback.
- [ ] `:focus-visible` defined on every interactive element (no bare `:focus`, no `outline: none` without replacement).
- [ ] Every `@vireya/ui` and `@vireya/blocks` import is a subpath (only `AppProvider` may come from the root).
- [ ] `<html suppressHydrationWarning>` set in `app/layout.tsx`; `AppProvider` wired with the full icon map + locale.
- [ ] React `19.2.1` pinned in `package.json`.
- [ ] Dark theme survives a toggle: no invisible text, no disappearing accent CTAs.
- [ ] Mobile (640px wide): section padding shrinks, no horizontal scroll.
- [ ] No twin `solid` CTAs of the same variant in one group — at most one solid; siblings demote to `outlined`/`ghosted`.
- [ ] At most one accent moment per viewport-fold.
