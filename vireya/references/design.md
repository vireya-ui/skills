# Vireya design guidelines — making decisions, not just following rules

This is the **decision layer**: when you face a fork (which color, which component, which spacing step), pick the right one. Token names and scales live in `tokens.md`. This file is about **judgment**.

---

## Foundational principles

These are non-negotiable. Every decision below derives from one of them.

1. **Token-first — ABSOLUTE.** Visual properties go through `var(--v-*)`. If the right token doesn't exist, override locally via inline `style` or a CSS Module variable, then open an issue at [`vireya-ui/skills/issues`](https://github.com/vireya-ui/skills/issues) so it can be added upstream. Don't proliferate one-off literals — concentrate them in one CSS module if multiple uses appear.
2. **Composition over configuration.** Vireya components avoid prop explosion: when variation grows, they expose slots (`ReactNode` props) or `asChild`. Apply the same rule to your own wrappers.
3. **Server-renderable by default.** Most Vireya components are server-renderable. Reach for `"use client"` only in your own components when you genuinely need state, refs, or browser APIs.
4. **Escape hatches always available.** Vireya never traps you: `className`, `style`, `asChild`, `<Component>Styles` reexport. Use them before forking.
5. **Accessibility is a baseline, not a feature.** `:focus-visible`, ARIA, keyboard, reduced-motion, contrast — floors, not opt-ins. Vireya enforces these internally; mirror the same in your own components.

---

## Color decisions

### Contrast pairing matrix — pick a row, paint the surface

The single biggest source of "ugly / no-contrast" Vireya UIs is mis-pairing color sub-tokens. Internalize this table — it covers 95% of cases:

| Surface tier | Background | Default text | Muted / supporting text | Border |
|---|---|---|---|---|
| Page (root, hero) | `--v-background-fill` | `--v-background-foreground` | `--v-secondary-active` | — |
| Card / nested container / sticky header strip | `--v-secondary-fill` | `--v-background-foreground` | `--v-secondary-active` | `--v-secondary-border` |
| Solid CTA (default) | `--v-primary-fill` | `--v-primary-foreground` | — | `--v-primary-border` |
| Accent CTA / promo surface | `var(--v-accent-fill, var(--v-primary-fill))` | `var(--v-accent-foreground, var(--v-primary-foreground))` | — | `var(--v-accent-border, var(--v-primary-border))` |
| Destructive | `--v-destructive-fill` | `--v-destructive-foreground` | — | `--v-destructive-border` |

**Two surprises worth memorizing:**

1. **Muted text uses `--v-secondary-active`, NOT `--v-secondary-foreground`.** `*-foreground` is reserved for "text painted ON the matching `*-fill`". For low-emphasis text on a normal page background, `--v-secondary-active` is what `<Text type="muted">` resolves to.
2. **A nested card with `secondary-fill` background still uses `background-foreground` for default text** — not `secondary-foreground`. The text color is determined by the *page* foreground, not the local fill, because the nested container is a tone shift, not a tier shift.

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

| Role | size | weight | leading | tracking |
|---|---|---|---|---|
| Hero display | `display-lg` (64) | `semibold` | `tight` | `tight` |
| Hero medium | `display` (48) | `semibold` | `tight` | `tight` |
| H2 (section) | `title-lg` (32) | `semibold` | `snug` | `tight` |
| H3 large (alternating feature) | `title` (24) | `semibold` | `snug` | `normal` |
| H3 card / tier | `body-lg`–`title-sm` (18–20) | `medium` | `snug` | `normal` |
| Pricing price (display) | `display` (48) | `semibold` | `tight` | `tight` |
| Body lead | `body-lg` (18) | `regular` | `relaxed` | `normal` |
| Body default / section description | `body` (16) | `regular` | `relaxed` | `normal` |
| Body small | `body-sm` (14) | `regular` | `relaxed` | `normal` |
| Eyebrow | `caption` (12) | `medium` | `normal` | `eyebrow` + `text-transform: uppercase` |
| Button / Badge / Tab | (field-font) | `medium` | `normal` | `normal` |

**Section H2 calibrated default is `title-lg` (32px), not 36.** Marketing pages use `<Text size="title-lg" weight="semibold">` paired with `<Text size="body" type="muted">` description. Mobile (`@media (max-width: 640px)`) shrinks the title to `var(--v-font-size-title) !important`. Use `display`/`display-lg` only for hero/art-direction headings (rare).

### Typography forbidden moves

- ❌ `<p style={{ fontSize: 18, fontWeight: 500 }}>` — use `<Text size="body-lg" weight="medium">`.
- ❌ Inventing `<Text size="huge">` or any name outside the 9-step ramp. Scale: `caption · body-sm · body · body-lg · title-sm · title · title-lg · display · display-lg`.
- ❌ `line-height: 1.55;` literal in CSS Modules. Always `var(--v-leading-relaxed)` (or `tight`/`snug`/`normal`).
- ❌ `letter-spacing: -0.02em;` literal. Always `var(--v-tracking-tight)` (or `normal`/`wide`/`eyebrow`).
- ❌ `font-weight: 500;` in CSS Modules. Always `var(--v-font-weight-medium)`.
- ❌ `<Text size={32}>` numeric. Numeric escape hatch was removed in 0.1.0 — pick the semantic name.

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
| Hairline (chart inner element) | `--v-radius-xs` |
| Sharp (chart bar, micro indicator) | `--v-radius-xs` |
| Inline (badge, chip, tag, tooltip) | `--v-radius-sm` |
| Small (input, alert, dropdown menu) | `--v-radius-md` |
| Medium (button `shape_square`) | `--v-radius-lg` |
| Large (card, feature item, dialog) | `--v-radius-xl` |
| X-large (pricing tier, hero card, drawer) | `--v-radius-2xl` |
| Pill / circle | `--v-radius-full` (= 9999px) or `50%` for circle |

**Rule:** children take a smaller radius than the parent. A button (`--v-radius-lg`) inside a card (`--v-radius-xl`) — never the inverse. Step down the scale: `2xl → xl → lg → md → sm → xs`.

**Anti-patterns:**
- ❌ `--v-radius-sm` on a 200px-tall card (looks sharp/cheap)
- ❌ `--v-radius-2xl` on a 24px badge (looks like a balloon)
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

## Visual depth — gradients, spotlights, masks

Vireya's signature look isn't shadows — it's **soft radial spotlights on top of solid token surfaces**. The treatment is restrained, token-driven, and built from a small set of recipes you compose. Six patterns cover what the official Vireya marketing site uses.

### Universal rules — apply to every gradient

1. **Translucent stops use `color-mix(in srgb, X N%, transparent)` — never `rgba()`.** Tokens have no alpha channel; `color-mix` is the alpha mechanism that stays inside the token system. Banned: `rgba(0,0,0,0.4)`. Required: `color-mix(in srgb, var(--v-primary-fill) 40%, transparent)`.
2. **Always layer the gradient over a solid `var(--v-background-fill)` (or `secondary-fill`)**, never let the gradient be the sole background. If the radial fails to render or the user has a forced theme, you still get a clean surface underneath.
3. **End every spotlight stop at `transparent`** — never end at a solid color. Sharp edges read as bugs.
4. **All colors are tokens.** No `#fff`, no `hsl(...)` literals. Spotlights use `--v-secondary-fill` as the "warm tint"; accent glows use `var(--v-accent-fill, var(--v-primary-fill))` or a scoped `--v-accent-example-*`.
5. **Geometry is proportional (%), not pixel.** Spotlights resize fluidly with the container — no mobile breakpoint needed for the gradient itself.

### Pattern A — Top-spotlight hero (THE Vireya signature)

The single most recognizable Vireya treatment. Used on every page hero and showcase header. "Light bleeds from the top centerline."

```css
.hero {
  width: 100%;
  background:
    radial-gradient(
      ellipse 60% 50% at 50% 0%,
      var(--v-secondary-fill) 0%,
      transparent 70%
    ),
    var(--v-background-fill);
  border-bottom: 1px solid var(--v-secondary-border);
}
```

**Geometry to memorize:** `ellipse 60% 50% at 50% 0%` — 60% wide, 50% tall, centered horizontally, anchored to the top edge. Stop fades from `secondary-fill` to `transparent` at 70%.

**Bottom-cap is part of the look.** The `border-bottom: 1px solid var(--v-secondary-border)` is what makes the hero feel anchored. Don't drop it.

### Pattern B — Bottom-spotlight card visual

Inverse of A — used inside cards where an illustration sits at the bottom and you want it to "rise from light". A common use is templates card visuals.

```css
.cardVisual {
  background:
    radial-gradient(
      ellipse 60% 80% at 50% 100%,
      var(--v-secondary-fill) 0%,
      transparent 70%
    ),
    var(--v-background-fill);
  width: 100%;
  height: 100%;
}
```

**Geometry:** `ellipse 60% 80% at 50% 100%` — same width as A, taller (80%), anchored to bottom-center. Pair with Pattern D (bottom-fade mask on the illustration itself) for the canonical "illustration melting into card" effect.

### Pattern C — Accent glow overlay (per-card branded glow)

Two-layer composite that paints an accent halo at the top of a card and fades the bottom into the page. Common use: real-world-examples sections that need brand color over neutral screenshots.

```css
.glow {
  position: absolute;
  inset: 0;
  z-index: 1;
  pointer-events: none;
  background:
    /* Top: accent halo */
    radial-gradient(
      ellipse 90% 70% at 50% 0%,
      color-mix(in srgb, var(--v-accent-fill, var(--v-primary-fill)) 30%, transparent) 0%,
      transparent 60%
    ),
    /* Bottom: fade into page bg, helps text legibility */
    linear-gradient(
      to bottom,
      transparent 55%,
      color-mix(in srgb, var(--v-background-fill) 65%, transparent) 100%
    );
}
```

**Rules:**
- `pointer-events: none` is mandatory — the glow is decoration, not a click target.
- Absolute `inset: 0` + a parent with `position: relative; isolation: isolate` (so `z-index` stacking is local).
- The accent stop tops out at **30% opacity** — anything more saturates and reads cheap.
- The bottom linear-gradient is what makes white text on the glow stay readable.
- Per-instance accent: scope a CSS custom property on a wrapper (`<main className="landing">` defines `--v-accent-example-fill: hsl(...)`), and the glow reads it via `var(--v-accent-example-fill, var(--v-accent-fill, var(--v-primary-fill)))`. **Never set a literal color on the glow itself.**

### Pattern D — Bottom-fade mask (illustration → card edge)

Makes an SVG illustration dissolve into the bottom of its container. The mask is purely geometric — no color tokens involved.

```css
.illustration {
  mask-image: linear-gradient(to top, transparent 0%, black 35%);
  -webkit-mask-image: linear-gradient(to top, transparent 0%, black 35%);
}
```

**Geometry:** the bottom **35%** of the element fades to transparent. The number is calibrated — at 25% the illustration looks abruptly cropped; at 50% it loses the focal point.

**Always include the `-webkit-mask-image` line** for Safari. The two declarations always come together.

Pair with a hover lift to make the illustration feel alive:
```css
.card:hover .illustration { transform: translateY(calc(var(--v-height-4) * -1)); }
```

### Pattern E — Translucent overlay surface (frosted-glass UI)

Floating UI element layered over a busy background (screenshot, illustration, gradient). Combines a translucent fill, backdrop blur, and a translucent border.

```css
.overlay {
  background: color-mix(in srgb, var(--v-background-fill) 70%, transparent);
  backdrop-filter: blur(var(--v-width-8));
  -webkit-backdrop-filter: blur(var(--v-width-8));
  border: 1px solid color-mix(in srgb, var(--v-secondary-border) 60%, transparent);
  border-radius: var(--v-radius-lg);
  padding: var(--v-height-4);
}
```

**Recipe:**
- Background: 70% of the page fill, 30% transparent — lets the underlying content tint through.
- Border: 60% of the standard border token, 40% transparent — keeps the edge visible without going hard.
- Blur: `var(--v-width-8)` (16px) is the calibrated default for "noticeable but not smeary".
- Always include the `-webkit-backdrop-filter` prefix.

### Pattern F — Halo text-shadow (legibility on busy bg)

When text sits over a screenshot, illustration, or any non-uniform background, add a tiny halo built from the page bg color. Reads as legibility aid, not as decorative shadow.

```css
.textOverImage {
  text-shadow: 0 1px 2px color-mix(in srgb, var(--v-background-fill) 80%, transparent);
}
```

**Rules:** offset `0 1px`, blur `2px`, color built from `--v-background-fill` at 80%. **Never** use a black shadow over light text — it reads as drop-shadow drama instead of a halo.

### Composition — what makes a Vireya page "feel like Vireya"

Stack these three pieces and you've got the canonical look:

1. **Hero with Pattern A** (top-spotlight + bottom-cap border).
2. **Sections with `Section.Root`** (no gradient — solid `--v-background-fill` with the section header underline).
3. **Cards with Pattern B inside the visual area** + Pattern D mask on illustrations + Pattern E for any floating chip/palette/badge over media.

The rhythm is: **gradient at the top of the page, neutral middle, gradient inside cards.** Don't gradient every section — the top-spotlight is a punctuation mark, not body type.

### Anti-patterns

| Smell | Fix |
|---|---|
| `background: rgba(0,0,0,0.4)` | `background: color-mix(in srgb, var(--v-primary-fill) 40%, transparent)` |
| `background: linear-gradient(...)` with no solid fallback | Layer the gradient OVER `var(--v-background-fill)` |
| Spotlight ending at a solid color stop | End every spotlight at `transparent` |
| Gradients on every section | Spotlights are punctuation — hero + cards only |
| Accent glow at >40% opacity | Cap at 30% — anything more saturates |
| Hardcoded `#fff` text-shadow | `color-mix(in srgb, var(--v-background-fill) 80%, transparent)` |
| Missing `-webkit-mask-image` / `-webkit-backdrop-filter` | Always pair the prefixed version with the standard one |
| Per-card accent set as a literal | Scope it as a CSS custom property on a wrapper class, read via `var(...)` chain |

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

`forwardRef + ...props` spread is the contract Vireya follows. When you write your own components, do the same: never filter unknown props — consumers (you, future-self, teammates) add `aria-*`, `data-*`, event handlers you don't know about.

---

## CTA decision matrix — Button variant × surface × size

| Role | `variant` | `surface` | `size` | Notes |
|---|---|---|---|---|
| Hero primary | `accent` (or `primary` if no accent in theme) | `solid` | `lg` | The single highest-emphasis CTA on the page |
| Hero secondary | `secondary` | `outlined` | `lg` | Sits next to hero primary, lower emphasis |
| Section primary | `primary` | `solid` | `md` | Default in-page CTA inside a section |
| Section secondary | `secondary` | `outlined` or `ghosted` | `md` | Adjacent to a primary, or a quiet standalone action |
| Card action | `secondary` | `ghosted` | `sm` | Card hover-revealed CTA, "Learn more →" |
| Destructive | `destructive` | `solid` | `md` | Pair with `AlertDialog` for confirmation |
| Toolbar / table row | `secondary` | `ghosted` | `sm` or `ss` | Dense surfaces — icon + label or icon-only |

**Rules:**
- **Never two `solid` buttons of the same `variant` in the same group.** One leads (solid), the other demotes to `outlined` or `ghosted`. Two equally-loud CTAs = no CTA.
- **Wrap router links via `asChild + <Link>` — never invent an `href` prop on Button.**
- **For accent-driven branded moments**, prefer `variant="accent"` over `variant="primary"`. Reserve `primary` solid for the neutral-strong default.

---

## Emphasis decision rule — `accent` or `primary` on blocks

The 8 blocks that expose `emphasis: "primary" | "accent"`: `hero/{centered, splitVisual}`, `pricing/{tiers, twoTier}`, `feature/{steps, checkList, highlights}`, `eyebrow`. Default is `"accent"`.

**Default to `accent`** for hero, pricing, and feature highlights — promotional surfaces that anchor the page.

**Switch to `primary`** when:
- The page has no defined accent in the theme (renders identical to primary, but the explicit choice signals intent).
- The section is legal/footer/utility chrome that shouldn't draw promotional attention.
- You've already used accent in the same viewport-fold. **Rule of thumb: one accent moment per fold.** Two accent CTAs above the fold dilute the brand color.

The other 19 blocks intentionally lack `emphasis` because they have no internal element where the choice would have a visible effect (footers/navbars/logo clouds are neutral chrome; CTAs use `variant`). Drive accent there via the `eyebrow` slot — pass `<Eyebrow emphasis="accent">Featured</Eyebrow>` directly when you need accent.

---

## Self-audit checklist — run before declaring a page done

If any check fails, the page is not done.

- [ ] **Tokens only**: `grep -RE '#[0-9a-fA-F]{3,8}\b|rgba?\(|\b\d+px\b' src/**/*.module.css` returns nothing inside your section CSS (allowed: `0`, `100%`, `9999px`, `1px`/`1.4px` in inline SVG).
- [ ] **Dark theme survives**: toggle to dark theme — no text becomes invisible, no accent CTA disappears (means every accent reference has a `var(--v-accent-*, var(--v-primary-*))` fallback chain).
- [ ] **Focus-visible everywhere**: tab through the page — every interactive element shows a visible 2px outline.
- [ ] **Server-renderable**: hero, sections, footer all render their content without JS (interactivity may be inert; content must show).
- [ ] **Mobile (640px wide)**: section titles shrink (`var(--v-font-size-title) !important`); section padding shrinks to `--v-height-32`; no horizontal scroll.
- [ ] **No twin-`solid` CTAs**: in any group, at most one button is `solid` of the lead variant; siblings demote to `outlined` / `ghosted`.
- [ ] **No nested blocks**: `@vireya/blocks` blocks never contain other blocks. Sub-sections compose `@vireya/ui` primitives inside `Section.Root`.
- [ ] **`<html suppressHydrationWarning>`** is set in `app/layout.tsx`.
- [ ] **`AppProvider` wired** with the full 14-icon map and a locale.
- [ ] **No barrel imports**: every `@vireya/ui` and `@vireya/blocks` import is a subpath (the ONE exception is `import { AppProvider } from "@vireya/ui"`).
- [ ] **Section.Title pairing**: section titles are `size="title-lg" weight="semibold"`, paired with `<Text size="body" type="muted">` descriptions.
- [ ] **One accent moment per viewport-fold**. If two folds both have an accent CTA, demote one to `primary`.

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
| `--v-radius-2xl` on a 24px badge | `--v-radius-sm` (children smaller than parent) |
| `box-shadow: 0 2px 8px ...;` on a resting card | Border + tone, not blur. Shadows on float layers only. |
| `@media (max-width: 819px)` | `@media (max-width: 768px)` (canonical breakpoint) |
| `<p style={{ fontSize: 18, fontWeight: 500 }}>` | `<Text size="body-lg" weight="medium">` |
| `background: rgba(0,0,0,0.4);` | `background: color-mix(in srgb, var(--v-primary-fill) 40%, transparent);` |

---

## When in doubt — escalate

If a decision feels wrong but the tokens/components don't accommodate it cleanly, **the system is missing something** — not your problem to bend the rules around. The right move:

1. **Try the escape hatches first.** Every Vireya component accepts `className` and `style`. Many also export their CSS Module as `<Component>Styles` — compose locally with `clsx`. For polymorphism, `asChild` lets you swap the rendered element.
2. **Override block-scoped tokens** when a block exposes them (see [Block-scoped tokens](#block-scoped-tokens) above). Consumer-controllable tokens live under `--v-block-{family}-*` — set them on a wrapper class or via inline `style` to retheme one block instance.
3. **If nothing fits, open an issue at [`vireya-ui/skills/issues`](https://github.com/vireya-ui/skills/issues).** Describe the use case, what you tried, and what surface (token / component variant / sub-component) would solve it.
4. **Don't invent your own `--v-*` tokens.** That namespace belongs to the design system; future Vireya releases may collide. Use a distinct prefix (`--app-*`, `--mybrand-*`) for app-local CSS variables.
