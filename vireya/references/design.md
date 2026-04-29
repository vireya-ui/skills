# Vireya design guidelines — making decisions, not just following rules

This is the **decision layer**: when you face a fork (which color, which component, which spacing step), pick the right one. Code-mechanical rules (imports, `forwardRef`, CSS Module conventions) live in `SKILL.md`. Token names and scales live in `tokens.md`. This file is about **judgment**.

The canonical source is `DESIGN.md` at the repo root (637 lines). This skill page is the distilled decision tree. If the two ever conflict, `DESIGN.md` wins — fix this file.

---

## Foundational principles

These are non-negotiable. Every decision below derives from one of them.

1. **Token-first — ABSOLUTE.** Visual properties go through `--v-*` tokens. If the right token doesn't exist, **add it to `@vireya/core` first**, then use it. Never inline a literal "for now". A token is cheaper to ship than a future migration.
2. **Composition over configuration.** Three boolean variants on a component is fine. Twelve is a contract failure. When variation explodes, expose slots (`ReactNode` props) or `asChild`.
3. **Server-renderable by default.** Reach for `"use client"` only when the component genuinely needs state, refs, or browser APIs. Static layout, links, headings — never client.
4. **Escape hatches always.** `className`, `style`, `asChild`, exposed `<Component>Styles`. The user is never trapped.
5. **Accessibility is a baseline, not a feature.** `:focus-visible`, ARIA, keyboard, reduced-motion, contrast — floors, not opt-ins.
6. **Source of truth lives in `@vireya/core`.** If `@vireya/ui` is computing something the core could expose, **fix the core**. If a block reaches for a value that should be a token, **add the token**. Never patch downstream.

---

## Color decisions

### The primary / accent split

`primary` and `accent` are NOT interchangeable. They serve different roles:

| Color | Identity | Use for |
|---|---|---|
| `primary` | neutral-strong (near-black on light, near-white on dark) | body text, default CTAs, icons, default borders, neutral foreground |
| `accent` | brand-vivid (magenta, teal, green, etc.) | promotional CTAs, "Popular" badge, focus rings, eyebrow dots, illustrative highlights |

This is the same split used by Vercel, Linear, and Stripe Press. **It exists so a consumer can ship a magenta `accent` without making body text magenta.**

**Rules:**
- Components that expose `variant="accent"`: **`Button` and `Badge` only**. Other components consume accent implicitly via roles (focus rings, selection states).
- `accent` is **optional**. Themes that don't declare it must render visually identical to a pre-accent design.
- **Always chain the fallback** in CSS: `var(--v-accent-fill, var(--v-primary-fill))`, same for every sub-token (`-foreground`, `-hover`, `-active`, `-subdued`, `-border`).
- **Never put two `primary` Buttons in the same form.** Promote one (the actual decision) and demote the other to `secondary`/`outlined`/`ghosted`. Two equally-loud CTAs = no CTA.

### Color hierarchy on a surface

Reach for these in order. If you find yourself reaching for #4 or #5, double-check the layout first.

1. `var(--v-background-fill)` — page bg, default surface
2. `var(--v-secondary-fill)` — subtle surfaces, muted fills, disabled bg
3. `var(--v-primary-fill)` — strong CTAs, foreground text on background
4. `var(--v-accent-fill, var(--v-primary-fill))` — branded moments only (one per surface, max)
5. `var(--v-destructive-fill)` — errors, destructive actions only
6. `var(--v-link-fill)` — inline hyperlinks only

### Sub-token pairing — pick all six together

Every named color has 6 sub-tokens. You don't pick which to use — you pick the **role** and pair them correctly:

```css
.solid {
  background-color: var(--v-primary-fill);
  color:            var(--v-primary-foreground);
  border:           1px solid var(--v-primary-border);
}
.solid:not(:disabled):hover  { background-color: var(--v-primary-hover); }
.solid:not(:disabled):active { background-color: var(--v-primary-active); }
.solid[aria-disabled="true"] { background-color: var(--v-primary-subdued); }
```

**Anti-pattern — sub-token mix-and-match:**
```css
/* WRONG — pairing primary fill with secondary border by hand */
background: var(--v-primary-fill);
border: 1px solid var(--v-secondary-border);   /* should be --v-primary-border */
```
If the auto-derived border looks wrong for a specific theme, **override it in `createTheme({ primary: { fill, border: "..." } })`** — don't hardcode in the component.

### Surface tier conflict — pick one

A surface gets its tone from EITHER its container OR a `variant="shaded"` prop, never both:

- ❌ `<Card variant="shaded">` inside a section already using `var(--v-secondary-fill)` as bg → two stacked shadeds collapse the visual rhythm
- ✅ Pick the outer surface tier. Inner cards stay `variant="outlined"` (border only).

### Color forbidden moves

- ❌ Don't introduce a 4th base color (success, warning, info) ad-hoc in a component. Add it to the theme contract via `createTheme` or compose `destructive` + neutral.
- ❌ Raw `rgba()` for transparency. Use `color-mix(in srgb, var(--v-{name}-fill) N%, transparent)` (already a pattern in Button shaded surface).
- ❌ Hardcoded hex/gray. `#e5e5e5` is `var(--v-secondary-border)`. `#f5f5f5` is `var(--v-secondary-fill)`. Always.
- ❌ Custom disabled gray. Disabled = `opacity: 0.5; cursor: not-allowed;` — universal.

---

## Typography decisions

### Hierarchy is built with size > weight > color, in that order

Texto pequeno em peso alto fica grosso e compete com headings. **Size is the primary lever**; weight is secondary. A `card title 18px @ medium` reads stronger than `18px @ semibold` when the H2 above is also semibold (contraste preservado).

### Three weight levels — that's the contract

| Weight | When to use |
|---|---|
| `regular` (400) | Body text, paragraphs, descriptions, table cells |
| `medium` (500) | UI assertion: eyebrows, badges, buttons, tabs, nav links, card titles ≤20px, form labels, table headers, chart labels |
| `semibold` (600) | Visual dominance: hero display, section headings, large H3 (≥24px), pricing prices, brand logo |
| `bold` (700) | **Reserved.** Art-direction only — explicit `weight="bold"` on a single element |

Never use `300` (light) or `800/900` (black). Inconsistent across fonts and breaks the 3-level hierarchy.

### Canonical role → token mapping for sections

When building a block or page, lean on this table instead of guessing:

| Role | size | weight | line-height | letter-spacing |
|---|---|---|---|---|
| Hero display | 64–72 (number) | `semibold` | 1.05 | -0.03em |
| H2 (section) | `xxl` (36) – 48 | `semibold` | 1.1 | -0.025em |
| H3 large (alternating feature) | `sl` (28) | `semibold` | 1.15 | -0.015em |
| H3 card / tier | `sm`–`md` (18–20) | `medium` | 1.2–1.3 | — |
| Pricing price (display) | 48–56 (number) | `semibold` | 1 | -0.02em |
| Body lead | `sm` (18) | `regular` | 1.55 | — |
| Body small | `xs` (14) | `regular` | 1.55 | — |
| Eyebrow | `xxs` (12) | `medium` | — | `0.08em` + `text-transform: uppercase` |
| Button / Badge / Tab | (field-font) | `medium` | 1 | — |

### Typography forbidden moves

- ❌ `<p style={{ fontSize: 18, fontWeight: 500 }}>` — use `<Text size="sm" weight="medium">`.
- ❌ Inventing `<Text size="xxxl">`. The named scale stops at `xxl` (36px). Display sizes pass numeric: `<Text size={64} weight="semibold">`.
- ❌ Line-height token. `line-height` scales with role, not size — set inline per use (table above).
- ❌ `font-weight: 500;` in CSS Modules. Always `var(--v-font-weight-medium)`.

---

## Spacing rhythm

### The ladder

Think in steps. Each "level" of grouping uses a value 1–2 steps higher than the previous:

| Step | Token | Use |
|---|---|---|
| Inline | `--v-{height,width}-2` to `-8` | Internal component spacing, icon gap, label-to-input |
| Tight | `--v-height-12` | eyebrow → title |
| Standard inline | `--v-height-16` | row gaps, default gap inside a card |
| Group | `--v-height-24` | badge → title group, tier feature list |
| Major group | `--v-height-32` | title group → actions |
| Section internal | `--v-height-48` to `-64` | header → grid, intro → content |
| Section padding compact | `--v-height-96` | logoCloud, footer, mobile sections |
| Section padding standard | `--v-height-128` | hero, feature, pricing |

**Rule:** never mix two adjacent steps in the same axis without intent. If a value you need isn't in the scale (e.g. 30px), **pick the nearest token (28 or 32)** — almost certainly the design wants the rhythm, not the literal.

### Section vertical rhythm — the canonical block template

Every section block follows this pattern. Mirror it when composing custom sections:

```
section padding-top    : --v-height-128 (hero/feature) or --v-height-96 (compact)
header → content gap   : --v-height-48 to --v-height-64
title → subtitle gap   : --v-height-12
title group → actions  : --v-height-32
section padding-bottom : same as top
```

Mobile (`@media (max-width: 768px)`): drop padding to `--v-height-96`, shrink display titles to ~32–40px via `font-size: Npx !important` (only legitimate `!important` use — overrides `Text`'s inline `--font-size`).

### `--v-page-padding` is the outer gutter

Use `--v-page-padding` for the outermost horizontal padding of any section. Never substitute a numeric token (`--v-width-16`) for the page gutter — apps may override the page gutter to a different rhythm.

---

## Radius hierarchy — children smaller than parent

Rounded shapes should **increase with element size**, and **inner elements take a smaller radius than their container**:

| Element | Token |
|---|---|
| Hairline (chart inner element) | `--v-radius-1` |
| Sharp (chart bar, micro indicator) | `--v-radius-2` |
| Inline (badge, chip, tag, tooltip) | `--v-radius-4` |
| Small (input, alert, dropdown menu) | `--v-radius-6` or `-8` |
| Medium (button `shape_square`) | `--v-radius-8` |
| Large (card, feature item, dialog) | `--v-radius-12` |
| X-large (pricing tier, hero card, drawer) | `--v-radius-16` |
| Pill / circle | `9999px` or `50%` (intentional, off-scale) |

**Rule:** all children of a container with `--v-radius-N` use `--v-radius-{N-2}` or smaller — never larger. A button (38px) inside a card (radius-12) takes radius-8, never radius-16.

**Anti-patterns:**
- ❌ `--v-radius-4` on a 200px-tall card (looks sharp/cheap)
- ❌ `--v-radius-16` on a 24px badge (looks like a balloon)
- ❌ `9999px` on anything that isn't a true pill (avatar, status dot, single-line tag)

---

## Elevation — Vireya is low-elevation

We prefer **borders + subtle background tint over shadows**. Shadows appear only on:
1. Hover lifts (cards, tier cards) — soft, short-throw
2. Floating layers (popover, dropdown, tooltip, modal) — defined, more spread

| Layer | box-shadow |
|---|---|
| Resting (card at rest) | **none** — use `border: 1px solid var(--v-secondary-border)` instead |
| Hover lift | `0 8px 24px -12px rgba(0, 0, 0, 0.08)` |
| Pricing/featured hover | `0 12px 32px -16px rgba(0, 0, 0, 0.10)` |
| Popover / dropdown | `0 8px 24px -8px rgba(0, 0, 0, 0.12)` |
| Modal / drawer | `0 24px 48px -16px rgba(0, 0, 0, 0.20)` |

**Forbidden:**
- ❌ Multi-layer shadows (`box-shadow: 0 1px 2px ..., 0 4px 8px ...`)
- ❌ Warm tints (`rgba(20, 10, 0, 0.1)`) — shadows are pure black + low alpha
- ❌ Resting shadow on non-floating surfaces — depth comes from border + tone

(These are inline today; tokenizing as `--v-elevation-{rest,hover,popover,modal}` is a tracked gap.)

---

## Motion — animate state changes, not vibes

### Budget

| Role | Duration |
|---|---|
| Instant | `0ms` — focus ring (immediate, communication) |
| Fast | `150ms` — micro-interactions (hover color, focus ring fade) |
| Default | `var(--v-motion-duration)` (250ms) — state transitions, accordion expand |
| Slow | `400ms` — drawer slide, page-level reveal |
| Slower | `600ms` — orchestrated, sequenced reveals |

### Easing

Default `var(--v-motion-timing)` (`cubic-bezier(0.4, 0, 0.2, 1)`, material standard) covers ~95% of cases. When you need:
- **Enter** (decelerating in): `cubic-bezier(0, 0, 0.2, 1)` (ease-out)
- **Exit** (accelerating out): `cubic-bezier(0.4, 0, 1, 1)` (ease-in)
- **Emphasized/playful**: `cubic-bezier(0.68, -0.55, 0.27, 1.55)` (overshoot)

**Linear easing is wrong for everything except progress bars and loading shimmer.**

### What to animate, what NOT to

- ✅ State changes that change layout (open/close, expand/collapse)
- ✅ Hover color shifts (≤ default duration)
- ✅ Focus ring fade-in
- ❌ Hover transform lifts on touch (use `@media (hover: hover) and (pointer: fine)` to gate)
- ❌ `transition: all` (always name properties)
- ❌ Mandatory entrance animations with no reduced-motion fallback

### Reduced motion is mandatory

Always wrap meaningful animation:
```css
@media (prefers-reduced-motion: reduce) {
  .myAnim { animation: none; transition: none; }
}
```
Hover color shifts and focus rings are **exempt** — they're communication, not motion.

---

## States — every interactive component must distinguish all of these

| State | Trigger | Visual |
|---|---|---|
| Default | At rest | Base color, base border |
| Hover | `:hover` (pointer only — gate with `@media (hover: hover)` for transforms) | `var(--v-{name}-hover)` bg or border lift |
| Active / Pressed | `:active` | `var(--v-{name}-active)` or compress (transform ≤ 1px) |
| Focus-visible | `:focus-visible` (NEVER `:focus` alone) | `outline: 2px solid var(--v-primary-fill); outline-offset: 2px;` |
| Disabled | `:disabled` / `[aria-disabled="true"]` | `opacity: 0.5; cursor: not-allowed;` no hover response |
| Loading | `data-loading` / prop | Hide content (`visibility: hidden`), absolutely-positioned loader centered |
| Selected / Checked | `[aria-selected]`, `[data-state="checked"]` | Fill with `--v-primary-fill`, border with `--v-primary-fill` |
| Invalid | `aria-invalid="true"` | Border `var(--v-destructive-fill)`, helper text destructive |

**Focus rule:** `:focus` alone fires on mouse clicks too — wrong default. Always `:focus-visible`. Never `outline: none` without a replacement.

**Hover rule on touch:** color shifts are OK on touch (read as tap feedback). Transform lifts are NOT — they tap-flicker. Gate transforms with `@media (hover: hover) and (pointer: fine)`.

---

## Responsive — one canonical breakpoint

Vireya uses **`768px`** as the primary breakpoint. Below = mobile, above = desktop. Tablet is "small desktop" unless content explicitly demands otherwise.

Secondary breakpoints used **sparingly** when a multi-column grid demands a middle tier:
- `1024px` — drop 4-col → 2-col
- `640px` — drop 2-col → 1-col

**Forbidden:**
- ❌ Per-component arbitrary breakpoints (`@media (max-width: 819px)`). Pick one of the three.
- ❌ Mobile-first `min-width` queries in marketing/landing code (we write desktop-first).

### Touch vs hover gating

Use `@media (hover: hover) and (pointer: fine)` to gate hover-only effects (transform lifts, parallax). On touch, those become tap-flickers.

### Charts adapt to their container, not the viewport

Charts can sit in narrow columns (sidebar, 2-up grid, dashboard tile). They use `useChartBreakpoint()` from `@vireya/ui/data/chart` and write `data-chart-bp="xs|sm|md"` on the root. CSS reacts via attribute selectors, not viewport queries.

---

## Composition decisions

### Slots beat configuration when the contained content varies

```tsx
// WRONG — closed configuration explodes
<Hero badgeText badgeVariant badgeIcon badgePosition />

// RIGHT — open slot
<Hero badge={<Badge variant="primary"><Sparkle /> New</Badge>} />
```

### `asChild` for polymorphism — never `href` on every component

```tsx
// RIGHT
<Button asChild><Link href="/">Home</Link></Button>

// WRONG — forces Button to know about routing
<Button href="/">Home</Button>
```

### Compound API rule — simple stays as prop, JSX-rich becomes a child

For multi-part components, expose a namespace (`<Family.Root>`, `<Family.Item>`, …). Split rule:

> **Simple values stay as props on `Root` (text, enums, booleans, numbers). Interactive or JSX-rich content goes through children + named sub-components.**

```tsx
// WRONG — opaque slot props, no IDE structure, forces Fragment
<CTABanner title="…" actions={<><Button/><Button/></>} />

// RIGHT — simple → prop, complex → child
<CTABanner.Root title="…" variant="shaded">
  <Button>Start</Button>
  <Button surface="outlined">Talk</Button>
</CTABanner.Root>
```

Sub-components are identified by `displayName = "FamilyVariant.PartName"` and filtered with `findChild`/`findChildren`/`excludeChildren` (no Context overhead for slot extraction). Use `createContext` only when sub-components share runtime state (e.g. `CTAInlineEmail.Form`'s email input + submit button).

### Compound vs flat — never expose both for the same concern

If `Dialog` already exposes `Dialog.Root + .Trigger + .Content + .Header + .Footer`, don't also ship a `<DialogShorthand title description actions />`. Pick one API per concern.

### Forward refs and rest props — always

`forwardRef + ...props` spread is the contract. Never filter unknown props — consumers add `aria-*`, `data-*`, event handlers you don't know about.

### Tokens before variants

A `variant="warning"` prop on a Button means there should be a `--v-warning-fill` token first. **Decisions go into the token layer; components consume.** No exceptions.

### Fix upstream

Block needs a border-color the theme doesn't provide? Add a `border` sub-token to `@vireya/core/parseColor`. UI button needs a hover state that isn't auto-derived right? Fix `safeLighten`. **Don't paper over in the leaf component.**

---

## Component pairing — which to pick

### Modal / overlay taxonomy

| Component | Use for |
|---|---|
| `Dialog` | **Blocking decisions** — confirmations, forms that need full attention. Modal backdrop. |
| `AlertDialog` | **Destructive confirmations only** — "Delete project?". `Action`/`Cancel` buttons mandatory. |
| `Sheet` | **Contextual side panels** — settings, filters, details. Doesn't fully break the flow. |
| `Drawer` | **Mobile-style bottom slide-up** — share sheets, filters on touch. (Built on Vaul.) |
| `Popover` | **Inline disclosure ≤300px** — color picker, quick form, contextual actions. Anchored to trigger. |
| `HoverCard` | **Preview on hover ≤1 sentence** — user mention preview, link metadata. Never actionable content. |
| `Tooltip` | **≤1 sentence on hover only** — clarification, keyboard hint. Never actionable, never essential info. |
| `DropdownMenu` | **List of actions/links** — context menu, "more" menu. Items are commands. |
| `ContextMenu` | **Right-click menu** — same as dropdown but trigger is `contextmenu` event. |
| `Toast` | **Async feedback ≤3 lines** — "Saved", "Connection lost". Auto-dismisses. Non-blocking. |

**Rules:**
- Anything actionable → `Popover` (anchored) or `DropdownMenu` (list of actions). NEVER `Tooltip` or `HoverCard`.
- Confirmations destructive → `AlertDialog`. Confirmations non-destructive → `Dialog`.
- Filter/settings on desktop → `Sheet`. Same on mobile → `Drawer`.

### Page section taxonomy — `@vireya/blocks` are FULL sections

| Layer | Role |
|---|---|
| `@vireya/blocks` | Full landing-page sections (Hero, Pricing, FAQ, Footer). One per page-region. |
| `@vireya/ui` | Primitives (Button, Card, Input, Tab). Compose into custom sections when no block fits. |

**Rules:**
- Never nest a block inside another block. If a feature section needs a sub-section, compose `@vireya/ui` primitives directly.
- Never re-implement a block as primitives if the block exists. `Hero.Centered` already handles eyebrow + title + description + actions + responsive — don't rebuild it.
- Always pass `eyebrow` as a `ReactNode` slot (use the `Eyebrow` primitive, not a custom span). Block props like `eyebrow?: ReactNode` accept rich nodes.

### `emphasis` prop — defaults to `accent`

Blocks with an `emphasis: "primary" | "accent"` prop default to `"accent"`. Change to `"primary"` only when:
- The eyebrow text **is** a CTA label (rare)
- The page has no defined accent and you want to be explicit
- A neutral/legal/footer section that shouldn't draw promotional attention

---

## Block-scoped tokens

`@vireya/blocks` follows a **two-layer token model**: blocks consume design system tokens (`--v-*`), but expose their own scoped surface (`--v-block-{family}-{property}`) for per-block customization.

### When to add a block-scoped token

Tokenize **identity-defining** properties — values where a brand or consumer would reasonably want to differ:
- ✅ Surface colors (card bg, section bg)
- ✅ Border colors
- ✅ Accent colors (highlights, indicators)
- ✅ Decorative properties unique to the block (overlay color, grayscale opacity)
- ❌ Spacing, padding (use design system tokens directly)
- ❌ Layout properties (grid, flex)
- ❌ Typography weight/size (already tokenized globally)

### Defaults must be design system tokens

```css
.section {
  /* RIGHT — falls back to active theme out of the box */
  --v-block-hero-accent: var(--v-primary-fill);

  /* WRONG — locks the block to a literal */
  --v-block-hero-accent: #00aa55;
}
```

Override at any cascade level: per-instance via `style`, per-page via wrapper class, per-theme via `:root.v-theme-brand body { ... }`.

---

## Accessibility floors

These are minimums. Going below is a bug.

- **Color contrast:** WCAG AA (4.5:1 body, 3:1 large text). Default themes meet this; verify custom themes before shipping.
- **Focus indicators:** every interactive element needs a visible `:focus-visible` ring.
- **Keyboard:** every interactive element reachable + operable via keyboard. No mouse-only handlers.
- **Hit targets:** primary CTAs ≥ 44×44px (use `lg`/`xl` field size). Inline icon buttons may go ≥ 32px in dense contexts.
- **ARIA:** primitives from `@base-ui-components/react` and Radix come with correct ARIA — don't strip it. Propagate `aria-*` props explicitly.
- **Semantic HTML:** `<button>` for actions, `<a>` for navigation, `<h1>`–`<h6>` for hierarchy. `<div onClick>` is wrong.
- **`prefers-reduced-motion`:** respected for all decorative animation. Required communication (focus ring) may stay.
- **Form fields:** every input needs an associated `<label>` (use `<Field.Root>`/`<Field.Label>`). **Placeholder is NOT a label.**
- **i18n:** ship locales (`enUS`, `ptBR` exist). **Hardcoded English strings inside `@vireya/ui` are bugs.**

---

## Anti-patterns gallery — fix these on sight

| Smell | Fix |
|---|---|
| `padding: 30px;` | `padding: var(--v-height-32);` (nearest step) |
| `border: 1px solid #e5e5e5;` | `border: 1px solid var(--v-secondary-border);` |
| `:focus { outline: none; }` | `:focus-visible { outline: 2px solid var(--v-primary-fill); outline-offset: 2px; }` |
| `font-weight: 500;` | `font-weight: var(--v-font-weight-medium);` |
| `transition: all 0.3s ease;` | `transition: background-color var(--v-motion-duration) var(--v-motion-timing);` |
| `<Button href="/">` | `<Button asChild><Link href="/" /></Button>` |
| `<Hero actions={<><Button/><Button/></>} />` | `<Hero.Root>...<Button/><Button/></Hero.Root>` |
| `<Card variant="shaded">` inside section already shaded | Pick one surface tier — outer wins |
| `<Tooltip>Click here to delete</Tooltip>` | Tooltips never carry actions — use Popover |
| Two `variant="primary"` Buttons in same form | Promote one, demote the other to `secondary` or `outlined` |
| `--v-radius-16` on a 24px badge | `--v-radius-4` (children smaller than parent) |
| `box-shadow: 0 2px 8px ...;` on a resting card | Border + tone, not blur. Shadows on float layers only. |
| `@media (max-width: 819px)` | `@media (max-width: 768px)` (canonical breakpoint) |
| `<p style={{ fontSize: 18, fontWeight: 500 }}>` | `<Text size="sm" weight="medium">` |
| `background: rgba(0,0,0,0.4);` | `background: color-mix(in srgb, var(--v-primary-fill) 40%, transparent);` |

---

## When in doubt — escalate

If a decision feels wrong but the tokens/components don't accommodate it cleanly, **the system is missing something** — not your problem to bend the rules around. The right move:

1. Check `DESIGN.md` §15 (open gaps) — your case may already be tracked.
2. Open the gap. Add the token / sub-component / variant to `@vireya/core` or `@vireya/ui` first.
3. Then use it from your code.

This is the "Source of truth" principle in practice. **Never patch downstream.**
