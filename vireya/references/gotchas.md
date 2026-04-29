# Vireya gotchas — read first when something feels off

Institutional knowledge, traps already hit, and surprises that aren't visible from the code alone.

---

## Setup / wiring

### `AppProvider` is required — without it, components break silently
The `<AppProvider>` client wrapper from `@vireya/ui` injects two contexts:
- **`locale`**: powers `Calendar`, `DateField`, and any future i18n. Without it, those components render but pickers can throw on missing `locale.calendar`.
- **`icons`**: maps canonical names (`Check`, `ChevronDown`, `Circle`, `Close`, `Calendar`, etc.) to React components. Without these, indicator slots in `Checkbox`, `RadioGroup`, `DropdownMenu` (checkbox/radio items), `Select`, `Toast` close, `PasswordField` toggle, `CopyField` confirmation, `Command` search etc. render empty.

If a Vireya component renders but a chevron/check/close icon is missing, **the icon name isn't passed to `AppProvider`**. The full required map is in `references/setup.md`.

### `--v-page-padding` and `--v-page-max-width` are NOT in `@vireya/core`
Blocks reference them, the design system doesn't ship them. Define on `body` in your app's `globals.css`:
```css
body { --v-page-max-width: 1520px; --v-page-padding: var(--v-width-16); }
```

### Subpath imports only
`@vireya/ui` and `@vireya/blocks` roots **do not** export components. The only allowed root import is `import { AppProvider } from "@vireya/ui"`. Everything else is a subpath: `@vireya/ui/form/button`, `@vireya/blocks/hero/centered`. Trying to barrel-import will return `undefined`.

### `<html suppressHydrationWarning>` is mandatory
The theme bootstrap script runs before React hydrates and sets the theme class on `<html>`. Without `suppressHydrationWarning`, React logs a hydration mismatch every time.

---

## React Server Components

### Non-component named exports from `"use client"` modules return `undefined` in RSC
If you write:
```ts
// my-component.tsx
"use client";
export const meta = { title: "..." };
export const MyComponent = () => <div />;
```
Then in a server component, `import { meta } from "./my-component"` gives **`undefined`** (Next.js' "use client" boundary only marshals component exports). Bake metadata at build time, move it to a separate server-only file, or define it inline in the server component that needs it.

### Most Vireya UI components are `"use client"`
Form controls, overlays, anything stateful — they're all client components and infect any module that imports them. Keep the boundary tight: import them only in leaf client components, not in shared server modules.

---

## Tokens

### `accent` is OPTIONAL — always chain the fallback
Apps can ship without an accent (e.g. greyscale-only). Vars like `--v-accent-fill` are `undefined` then. CSS that doesn't fall back will silently break:
```css
/* bad */
background: var(--v-accent-fill);

/* good */
background: var(--v-accent-fill, var(--v-primary-fill));
```
This applies to `fill`, `foreground`, `hover`, `active`, `subdued`, `border` — every accent sub-token.

### `--v-w` and `--v-b` are foreground anchors, not theme colors
They're theme-invariant (`#ffffff` and `#0a0a0a`) and used as auto-derived foregrounds. Don't redefine them per-theme — set them once at root if you need different absolutes.

### Default dark theme is pure greyscale
The bundled `dark` theme has no accent (it falls back to white-on-black `primary`). If you want a vivid dark mode, define your own theme via `createTheme({ accent: "..." })` and pass it through `customThemes`.

---

## CSS Modules / styling

### Variant class names use underscores, not hyphens
`.variant_primary`, `.surface_solid`, `.shape_circle` — NOT `.variant-primary`. The convention is consistent across all components; mirror it in new ones for the `tatcn` composition pattern to work.

### `:not(:disabled):hover` to gate hover styles
Otherwise disabled buttons still light up on hover. Same for `:active`, `:focus-visible`. Pattern:
```css
.button:not(:disabled):hover { background-color: var(--v-primary-hover); }
```

### Disabled = `opacity: 0.5; cursor: not-allowed;`
Don't invent a custom disabled gray color. The opacity + cursor combination is the universal Vireya signal.

### Never `transition: all`
Always name properties:
```css
transition: background-color var(--v-motion-duration) var(--v-motion-timing),
            color            var(--v-motion-duration) var(--v-motion-timing);
```

---

## Animation flicker patterns

### Close-side opacity snap-back
When animating an overlay closed with `@keyframes` on `opacity`, browsers can snap to the resting opacity value the instant the animation ends, causing a visible flicker. Fix: animate via `transition` instead of `@keyframes` for close, or use `animation-fill-mode: forwards` with the final keyframe matching the resting state.

### Open-side height overshoot
Animating `height: 0 → auto` doesn't work, and `max-height` based hacks overshoot when content is shorter than the max. Recipe:
1. Measure intrinsic height once with a ref.
2. Animate `height` from `0` to the measured value (not `max-height`).
3. After the transition ends, set `height: auto` so resize works normally.
4. For raf-based regression tests, sample frame-by-frame to assert no overshoot.

---

## Components — surprises

### `Treemap` (recharts) `stroke` bleeds into custom `<text>`
Recharts' `<Treemap>` accepts a top-level `stroke` prop that **inherits onto SVG children, including custom `<text>` you render inside `content`**. The result is an outline painted over your text glyphs. Fix at the text element:
```tsx
<text stroke="none" fill={...}>{label}</text>
```

### `DateField` and `Calendar` need locale + chevron icons
Even with `AppProvider` set, if your `locale` doesn't include a `calendar` key (matching `react-day-picker`'s `Locale`), the picker throws on render. Use `import enUS from "@vireya/ui/locales/enUS"` or `ptBR` — both are pre-shipped with the right shape.

### Optional peer deps must be installed before use
`dataTable`, `chart`, `carousel`, `command`, `drawer`, `resizable`, `treeView`, `calendar`/`dateField` import their underlying libs as peers (so apps that don't use them don't pay the bundle cost). If you import `@vireya/ui/data/dataTable` without `@tanstack/react-table` installed, the build fails with a missing-module error at the import boundary, not at usage.

### `Eyebrow` is a slot, not a fixed text component
Block props like `eyebrow?: ReactNode` accept any node. Pass a `<Eyebrow variant="dot" emphasis="accent">Label</Eyebrow>` for richer treatments, not just a string. Don't write a custom span — the `Eyebrow` primitive already handles the variants.

### Showcase variant dedup
If a new showcase only differs from an existing one by toggling an opt-in prop (`popular`, `bordered`, `reverse`), fold it into the existing showcase with a prop comment. Recurring custom layers in showcase code → promote to a real block primitive.

---

## Build / monorepo

### Don't hand-edit `packages/ui/package.json` exports
The `exports` map for `@vireya/ui` is regenerated on build by `ExportMakerPlugin` (in `packages/ui/src-build/`), based on a fast-glob over `src/components/<group>/<name>/index.tsx`. Adding a new component folder is enough — don't touch the long `exports` block manually. If exports look stale, just run `pnpm --filter @vireya/ui build`.

### React is pinned at 19.2.1 — sync any bump
React 19.2.1 is a peer dep across every package. Bumping in one package without the rest will produce duplicate-React runtime errors. If you bump, bump everywhere in the same commit, regenerate the lockfile, and re-run the test suite (Storybook + addon-vitest + Playwright in `@vireya/ui`).

### tsup `.d.ts` gotcha — re-export interfaces from the same file
If you define `IComponentProps` in one file and re-export from `index.tsx`, tsup's DTS bundler can drop the re-export under certain config combinations. Pattern that always works: define `IProps` in the same file as the component, export both with `export { Component, type IComponentProps }`.
