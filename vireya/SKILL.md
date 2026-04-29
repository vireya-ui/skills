---
name: vireya
description: Use when building or modifying web UIs with the Vireya Design System (@vireya/core, @vireya/ui, @vireya/next, @vireya/blocks). Covers design guidelines (color hierarchy, primary vs accent, typography rhythm, spacing ladder, radius hierarchy, elevation philosophy, motion budget, state contracts, component pairing decisions like Dialog vs Sheet vs Popover), package map, per-component subpath imports, mandatory --v-* token rules, forwardRef + asChild + CSS Module conventions, Next.js setup (ThemeProvider + AppProvider with locale + icons), and a flat catalog of 67 components and 26 blocks. Trigger on mentions of "vireya", "@vireya/*", building landing pages or dashboards with Vireya, designing with Vireya tokens, choosing between Vireya components, or creating Vireya components.
---

# Vireya Design System (web)

Vireya is a token-first React design system shipped as a pnpm monorepo. Web consumers compose `@vireya/ui` components and `@vireya/blocks` sections inside a Next.js app wired up with `@vireya/next`.

This skill teaches you the rules, package boundaries, import shape, and conventions. **For per-component prop details, open the component's `index.tsx` directly** — `IProps` interfaces are exported next to the code and stay authoritative. Don't memorize props from this skill; navigate to source.

---

## NON-NEGOTIABLE RULES

1. **Tokens are absolute.** Every CSS visual property goes through a `--v-*` token. Hardcoded `px`/`#hex`/`rgb()`/`rgba()` is a bug. The only allowed literals are: `0`, `100%`, `9999px` (true pill/circle), `1px` or `1.4px` for inline-SVG hairlines, `currentColor` in icons, and percentages in calc. If the token you need doesn't exist, **add it to `@vireya/core` first**, then use it. Never ship one-off literals "for now".

2. **`primary` ≠ `accent`.** `primary` is neutral-strong (near-black on light, near-white on dark) — use it for body text and default CTAs. `accent` is brand-vivid (magenta, teal, green, etc.) — use it only for promotional/highlight roles (popular badge, focus ring, eyebrow dot, hero CTA). `accent` is **optional** and falls back to `primary`. Always chain the fallback in CSS:
   ```css
   background-color: var(--v-accent-fill, var(--v-primary-fill));
   color: var(--v-accent-foreground, var(--v-primary-foreground));
   border-color: var(--v-accent-border, var(--v-primary-border));
   ```

3. **React 19.2.1 is pinned everywhere.** Never bump React in a single package without syncing the rest of the monorepo.

4. **Per-component subpath imports only.** Never `import { Button } from "@vireya/ui"`. Always `import { Button } from "@vireya/ui/form/button"`. Same for blocks: `@vireya/blocks/hero/centered`, not `@vireya/blocks`.

5. **Focus and disabled states.** Use `:focus-visible` (never `:focus`). Disabled = `opacity: 0.5; cursor: not-allowed;` — no custom gray colors.

---

## PACKAGES

| Package | Purpose | Typical import |
|---|---|---|
| `@vireya/core` | Tokens, `createTheme`, color/grade/CSS utils | `import { createTheme, parseColor } from "@vireya/core"` |
| `@vireya/ui` | 67 React components, 8 categories | `import { Button } from "@vireya/ui/form/button"` |
| `@vireya/next` | `ThemeProvider` (server), `useTheme` (client), motion preset CSS files | `import { ThemeProvider } from "@vireya/next"` |
| `@vireya/blocks` | 26 pre-composed sections (landing focus today) | `import { HeroCentered } from "@vireya/blocks/hero/centered"` |

The `AppProvider` wrapper is exported from the **root** `@vireya/ui` barrel — that's the one allowed root import. Locales are default exports: `import enUS from "@vireya/ui/locales/enUS"`.

---

## TOKEN CHEAT SHEET

Full catalog (every scale, composite tokens, color sub-tokens) lives in `references/tokens.md`. The 90% case:

| CSS property | Token |
|---|---|
| `font-size` | `var(--v-font-N)` (N = 10, 12, 14, 16, 18, 20, 24, 32, 36, 48, 64) **or** `<Text size="xs\|ss\|sm\|md\|lg\|sl\|xl\|xxl">` |
| `font-weight` | `var(--v-font-weight-{regular,medium,semibold,bold})` |
| `font-family` | `var(--v-font-family-{sans,mono})` |
| `padding`, `margin`, `gap`, offsets | `var(--v-{height,width}-N)` (N = 1, 2, 4, 6, 8, 10, 12, ..., 128) |
| Outer page gutter | `var(--v-page-padding)` (defined per-app, see setup) |
| `border-radius` | `var(--v-radius-N)` (N = 1, 2, 4, 6, 8, 10, 12, ..., 32). Only use `9999px` for pill/circle. |
| `background`, `color`, `fill`, `stroke` | `var(--v-{name}-{fill,foreground,hover,active,subdued,border})` |
| Color names | `primary`, `secondary`, `background`, `destructive`, `link`, `success`, `warning`, `info`, `accent` (optional) |
| `transition-duration`, `animation-duration` | `var(--v-motion-duration)` (250ms baseline) |
| `transition-timing-function` | `var(--v-motion-timing)` (`cubic-bezier(0.4, 0, 0.2, 1)`) |
| `box-shadow` (elevation) | `var(--v-elevation-{rest,hover,popover,modal})` — low-elevation system; rest is `none` (use border + tone) |
| Inner content `max-width` | `var(--v-content-width-{sm,md,lg})` (560 / 720 / 880 px). Page wrappers use `--v-page-max-width`. |
| Component sizing inside CSS Modules | Composite tokens: `var(--v-field-{ss,sm,md,lg,xl}-{height,padding,font,size})`, plus `--v-avatar-*`, `--v-badge-*`, `--v-icon-*` |

Color sub-token model: every color expands to 6 sub-tokens — `fill`, `foreground` (text on fill), `hover`, `active`, `subdued`, `border`. Always pair them correctly:
```css
background-color: var(--v-primary-fill);
color: var(--v-primary-foreground);
border: 1px solid var(--v-primary-border);
&:hover { background-color: var(--v-primary-hover); }
```

---

## COMPONENT CATALOG (67 components, 8 groups)

For prop details, open the file at `packages/ui/src/components/<group>/<name>/index.tsx`. Every component exports a typed `IProps` (e.g. `IButtonProps`).

### form (15)
`@vireya/ui/form/button` `baseField` `checkbox` `colorField` `colorPicker` `copyField` `dateField` `input` `passwordField` `radiogroup` `select` `slider` `switch` `textArea` `toggleGroup` `uploadArea`

### data (14)
`@vireya/ui/data/alert` `avatar` `badge` `calendar` `card` `carousel` `chart` `collapsible` `colorArea` `command` `dataTable` `heatmap` `tab` `table`

### feedback (4)
`@vireya/ui/feedback/progress` `skeleton` `spinner` `toast`

### overlay (9)
`@vireya/ui/overlay/alertDialog` `contextMenu` `dialog` `drawer` `dropdownMenu` `hoverCard` `popover` `sheets` `tooltip`

### navigation (5)
`@vireya/ui/navigation/breadcrumb` `header` `menu` `sidebar` `treeView`

### layout (3)
`@vireya/ui/layout/aspectRatio` `resizable` `scrollArea`

### typography (2)
`@vireya/ui/typography/text` `accordion`

### common (5)
`@vireya/ui/common/brand` `flex` `icon` `page` `separator`

**Notes:**
- `Card`, `Dialog`, `Drawer`, `DropdownMenu`, `ContextMenu`, `Sidebar`, `Toast`, `Carousel`, `DataTable`, `Heatmap`, `Chart`, `Tooltip`, `HoverCard`, `Sheet`, `AlertDialog`, `Accordion`, `Tab`, `RadioGroup`, `Select`, `Slider`, `ToggleGroup`, `Breadcrumb`, `Header`, `TreeView`, `ScrollArea`, `Resizable`, `Command`, `Popover`, `Avatar`, `Alert`, `Table` use **compound APIs** (Root + subcomponents like `.Item`, `.Content`, `.Trigger`). The folder's `index.tsx` shows the full surface.
- `DateField`, `Calendar`, and `DropdownMenu`/`Select` indicators **require** `AppProvider` to inject locale + icons. See setup.
- `Button`, `Card`, `AspectRatio`, `Dialog`, `AlertDialog`, `Sheet`, `Popover`, `Tooltip`, `HoverCard`, `Breadcrumb`, `TreeView`, `Sidebar` support `asChild` for composing with custom elements (e.g. Next.js `<Link>`).
- Heavy peer-dep components (opt-in): `dataTable` → `@tanstack/react-table`; `chart` → `recharts`; `carousel` → `embla-carousel-react`; `calendar`/`dateField` → `react-day-picker`; `command` → `cmdk`; `drawer` → `vaul`; `resizable` → `react-resizable-panels`; `treeView` → `@headless-tree/core`. Install only what you use.

---

## BLOCK CATALOG (26 blocks, 7 families)

Pre-composed landing-page sections, all using compound APIs (`Root` + items like `.Tier`, `.Feature`, `.Item`, `.Logo`, `.Form`, etc.). Common props across blocks: `eyebrow`, `title`, `description` (all `ReactNode`), `emphasis: "primary" | "accent"` (defaults to `"accent"`), `variant: "default" | "shaded"` for surface variants where applicable.

| Family | Blocks (import as `@vireya/blocks/<family>/<name>`) |
|---|---|
| `navbar` | `simple`, `dropdown` |
| `hero` | `centered`, `minimal`, `backgroundImage`, `splitVisual` |
| `logoCloud` | `grid`, `row` |
| `feature` | `alternating`, `checkList`, `highlights`, `iconGrid`, `steps` |
| `pricing` | `tiers`, `twoTier` |
| `cta` | `banner`, `centered`, `inlineEmail` |
| `faq` | `accordion`, `twoColumn` |
| `testimonials` | `single`, `grid` |
| `contact` | `simple`, `split` |
| `footer` | `minimal`, `columns` |
| `eyebrow` | utility (also reused inside other blocks) |

For full subcomponent shapes (e.g. `PricingTiers.Tier` props, `ContactSplit.Form` value type), see `references/blocks.md`.

**`emphasis` prop is opt-in by design** — only blocks with concrete accent-eligible elements expose it: `hero/{centered,splitVisual}`, `pricing/{tiers,twoTier}`, `feature/{steps,checkList,highlights}`, `eyebrow`. Footers, navbars, logo clouds, FAQ, CTA banners, contact forms, and the rest are intentionally neutral chrome — drive accent there via the `eyebrow` slot (`<Eyebrow emphasis="accent">`) when needed.

**Pricing "highlighted" tier** uses `highlighted` + `highlightedLabel` props (not `popular`), consistent across both `PricingTiers.Tier` and `PricingTwoTier.Tier`.

---

## CONVENTIONS — writing NEW components for `@vireya/ui`

Canonical reference: `packages/ui/src/components/form/button/index.tsx`. Mirror its shape exactly:

1. `"use client"` at the top **only if** the component needs state, refs, or hooks. Keep it server-renderable when possible.
2. `forwardRef<HTMLElement, IProps>` — always. Forward to the underlying DOM node.
3. Export the `IProps` interface (`IButtonProps`, `ICardProps`, etc.). Extend the appropriate HTML attribute type (`ButtonHTMLAttributes<HTMLButtonElement>`, `HTMLAttributes<HTMLDivElement>`, ...).
4. **Spread unknown `...props`** — never filter. Consumers add `aria-*`, `data-*`, event handlers.
5. Colocate `index.module.css` in the same folder. CSS Module class names use `kebab-case` for the base, `underscore_separators` for variant names: `.variant_primary`, `.surface_solid`, `.shape_circle`, `.size_md`.
6. Compose classNames with `tatcn` from `src/utils/modules/array-to-classname` (it's a `clsx` alias):
   ```ts
   className={tatcn([className, styles.button, styles[`variant_${variant}`], full && styles.full])}
   ```
7. For polymorphism, accept `asChild?: boolean` and use `Slot` (+ `Slottable` when there are extra children) from `src/utils/modules/slot`:
   ```tsx
   const Btn = asChild ? Slot : "button";
   return <Btn ref={forwardedRef} {...props} className={tatcn(...)}>{asChild ? <Slottable>{children}</Slottable> : children}</Btn>;
   ```
8. Re-export the styles object so consumers can compose: `export { Button, ButtonStyles }; const ButtonStyles = styles;`
9. Set `Button.displayName = "Button"`.
10. In CSS: gate hover with `:not(:disabled):hover`. Disabled = `opacity: 0.5; cursor: not-allowed`. Transitions name explicit properties (never `transition: all`).
11. Every CSS value is a token. **Re-read NON-NEGOTIABLE RULE #1.**

**Build system:** the `tsup` config auto-discovers every `index.tsx` under `src/components/` via fast-glob. Adding a new component folder is enough — `ExportMakerPlugin` regenerates the package.json `exports` map on build. Don't hand-edit the exports block. Reference: `packages/ui/tsup.config.ts`.

---

## NEXT.JS SETUP

The full template (with copy-pasteable files) is in `references/setup.md`. Minimum surface:

**`app/layout.tsx`** (server component, no `"use client"`):
- Import `"@vireya/next/styles"` **first**.
- Optional: `import "@vireya/next/motion/wipe-reveal"` (or `mask-reveal` / `circle-reveal` / `angled-reveal`) for animated route/theme transitions via View Transitions API.
- Import your app's `globals.css` **last**.
- Wrap in `<ThemeProvider customThemes={{...createTheme(...)}}>` and `<AppProvider>` (the client wrapper).
- `<html suppressHydrationWarning>` is **required** (theme hydration script runs before React).

**`app/layout.client.tsx`** (`"use client"`):
- `import { AppProvider as UIAppProvider } from "@vireya/ui"` (root barrel — the one allowed exception).
- Pass `locale` (e.g. `import ptBR from "@vireya/ui/locales/ptBR"`) and `icons` (a map from canonical names to lucide-react components).
- **Without this wrapper, `Calendar`/`DateField`/`Select` indicator/`Toast`/`DropdownMenu` checkmarks/etc. break.** Required icon names: `Calendar`, `Check`, `ChevronDown`, `ChevronLeft`, `ChevronRight`, `ChevronUp`, `Circle`, `Close` (alias of `X`), `CloudUpload`, `Copy`, `Eye`, `EyeOff`, `Search`, `Brush`.

**`styles/globals.css`** (or wherever your app globals live):
- Define `--v-page-max-width` and `--v-page-padding` on `body`. **These tokens are NOT shipped by `@vireya/core`** but blocks assume they exist.
- Recommended: `--v-page-max-width: 1520px; --v-page-padding: var(--v-width-16);` (or `clamp(16px, 5vw, 32px)`).

---

## DESIGN DECISIONS — quick distillation

Full decision guide in `references/design.md`. The five rules that prevent most design bugs:

1. **Hierarchy is built with size > weight > color, in that order.** A `card title 18px @ medium` reads stronger than `18px @ semibold` when the H2 above is also semibold. Don't reach for weight to compensate for too-small text.
2. **Never two `primary` Buttons in the same form.** Promote one (the actual decision) and demote the other to `secondary`/`outlined`/`ghosted`. Two equally-loud CTAs = no CTA.
3. **Children take a smaller radius than the parent.** A button (`--v-radius-8`) inside a card (`--v-radius-12`) — never the inverse. `9999px` only on true pills/avatars.
4. **Vireya is a low-elevation system.** Borders + tone over shadows. Resting cards have `border: 1px solid var(--v-secondary-border)`, NO shadow. Shadows live on float layers (popover, dropdown, modal) and hover lifts only.
5. **Never nest a block inside another block.** `@vireya/blocks` are full sections. If you need a sub-section, compose `@vireya/ui` primitives directly.

**Modal/overlay picker:** `Dialog` for blocking decisions, `AlertDialog` for destructive confirmations only, `Sheet` for contextual side panels (desktop), `Drawer` for bottom slide-up (mobile/touch), `Popover` for inline disclosure ≤300px, `HoverCard` for previews ≤1 sentence (never actionable), `Tooltip` for clarification ≤1 sentence (never actionable). Anything actionable on hover → `Popover` or `DropdownMenu`, never `Tooltip`.

**Section vertical rhythm:** section padding `--v-height-128` (hero/feature) or `-96` (compact); header→content `-48`/`-64`; title→subtitle `-12`; title group→actions `-32`. Mobile drops padding to `-96` and shrinks display titles via `font-size: Npx !important` (the only legit `!important` in the system).

---

## ROUTING TABLE — when to read what

| Need | Open |
|---|---|
| Choosing between components, color hierarchy, spacing rhythm, weight/size mapping, anti-patterns, "should this be a Dialog or a Sheet" | `references/design.md` |
| Detailed props of a specific component | `packages/ui/src/components/<group>/<name>/index.tsx` directly |
| Full token catalog (every scale, every composite token, full color sub-token model) | `references/tokens.md` |
| Block subcomponent shapes (e.g. `PricingTiers.Tier`, `ContactSplit.Form` value type) | `references/blocks.md` |
| Boilerplate for a new Next.js app, custom themes, theme switching, motion presets | `references/setup.md` |
| Weird bug, surprising behavior, type errors, RSC issues, animation flicker | `references/gotchas.md` **first** |

---

## TOP 5 GOTCHAS — keep these in mind always

Full list in `references/gotchas.md`. The ones that cause most pain:

1. **`AppProvider` is required.** Without the client wrapper providing icons + locale, components like `DateField`, `Calendar`, `Select` (chevron indicator), `Toast` close button, `DropdownMenu` checkmarks render broken or invisible.
2. **`--v-page-padding` and `--v-page-max-width` ship from `@vireya/core` with sane defaults** (`clamp(16px, 5vw, 32px)` and `1200px`). Apps may override on `body` if they want a different rhythm — `apps/www` uses `1520px`. Blocks consume them transparently.
3. **Don't export non-component metadata from `"use client"` modules.** Constants, objects, arrays, `meta` exports return `undefined` when imported from a server component. Bake metadata at build time or move it to a server-only module.
4. **No barrel imports.** `@vireya/ui` root only exports `AppProvider`. Components are subpath-only: `@vireya/ui/form/button`. Same for `@vireya/blocks`.
5. **`accent` requires fallback chains.** Apps that don't define an accent get `undefined` on those vars. Always: `var(--v-accent-fill, var(--v-primary-fill))`.
