# Vireya tokens — full catalog

All tokens are CSS custom properties prefixed `--v-*`, declared on `body` by `@vireya/next/styles`. Inspect them at runtime in DevTools (Computed → Variables) or read the built CSS at `node_modules/@vireya/next/dist/styles.css`.

---

## Base units (overridable via `<ThemeProvider vars={...}>`)

```
--v-height           2px    base unit for graded --v-height-N
--v-width            2px    base unit for graded --v-width-N
--v-motion-delay     0ms
--v-motion-duration  250ms    alias of --v-motion-default
--v-motion-timing    cubic-bezier(0.4, 0, 0.2, 1)
--v-overlay-z-index  9
--v-w                #ffffff   "white" anchor (theme-invariant)
--v-b                #0a0a0a   "black" anchor (theme-invariant, NOT pure black)
```

`--v-w` and `--v-b` are foreground anchors used by auto-derived `foreground` sub-tokens. Override them at the root if your app needs a different "light text" / "dark text" pair.

> **Removed in 0.1.0:** the bare `--v-font` and `--v-radius-md` base units (and the graded `--v-font-N`, `--v-radius-N` derived from them). Font sizes and radii are now semantic, not graded.

---

## Font families & weights

```
--v-font-family-sans   Inter, system-ui, -apple-system, "Segoe UI", Roboto,
                       "Helvetica Neue", Arial, sans-serif
--v-font-family-mono   ui-monospace, "SF Mono", Menlo, "Cascadia Code",
                       "JetBrains Mono", Consolas, monospace

--v-font-weight-regular    400    body text
--v-font-weight-medium     500    UI active, subheads, buttons, badges
--v-font-weight-semibold   600    display, H2/H3 ≥24px, section headings
--v-font-weight-bold       700    art-direction only (rare)
```

---

## Type ramp (semantic)

`--v-font-size-{name}` — 9 degraus modulares. Use os nomes semânticos via `<Text size="...">` ou direto em CSS.

| Token | px | Role |
|---|---|---|
| `--v-font-size-caption` | 12 | Eyebrow, caption, badge, micro-label |
| `--v-font-size-body-sm` | 14 | Small body, labels, table cells, helpers |
| `--v-font-size-body` | 16 | **Body default** — paragraphs, list items, form inputs |
| `--v-font-size-body-lg` | 18 | Lead paragraph |
| `--v-font-size-title-sm` | 20 | Card title, subhead |
| `--v-font-size-title` | 24 | Section subtitle, H3 |
| `--v-font-size-title-lg` | 32 | H2, dashboard heading |
| `--v-font-size-display` | 48 | Hero medium, pricing display |
| `--v-font-size-display-lg` | 64 | Hero principal |

Em React: `<Text size="body-lg">`. Em CSS: `font-size: var(--v-font-size-body);`.

---

## Line-height & letter-spacing

```
--v-leading-tight     1.05      display
--v-leading-snug      1.15      titles
--v-leading-normal    1.4       UI / labels / dense
--v-leading-relaxed   1.55      body paragraph

--v-tracking-tight    -0.02em   display
--v-tracking-normal   0         default
--v-tracking-wide     0.04em    UI labels small caps
--v-tracking-eyebrow  0.08em    uppercase eyebrows
```

Em CSS: `line-height: var(--v-leading-relaxed);`, `letter-spacing: var(--v-tracking-eyebrow);`. Sem literais inline.

---

## Spacing scale (graded)

`--v-width-N` (horizontal) e `--v-height-N` (vertical). Mesma escala numérica em ambos:

```
1, 2, 4, 6, 8, 10, 12, 14, 16, 18, 20, 22, 24, 32, 40, 48, 64, 80, 96, 128
```

20 valores. Faixa principal step=2 (2..24), depois saltos maiores (32, 40, 48, 64, 80, 96, 128) + hairline `1`.

Common usage:
- `1`–`8`: internal component spacing (icon gap, label-to-input)
- `12`: tight vertical (eyebrow → title)
- `16`: standard inline gap
- `24`: standard vertical group gap
- `32`: major group gap (title → actions)
- `40`: field/button height (default)
- `48`–`64`: section-internal gaps
- `80`: mid gutter
- `96`–`128`: section padding (mobile / desktop hero)

> **Removidos** em 0.1.0: `3, 5, 28, 38, 42` (valores arbitrários). Snap pra vizinho mais próximo via codemod.

---

## Radius scale (semantic)

`--v-radius-{name}` — 7 degraus, sem furos numéricos. Substitui a antiga escala graded `--v-radius-N`.

| Token | px | Use |
|---|---|---|
| `--v-radius-xs` | 2 | Chart bars, hairline accents |
| `--v-radius-sm` | 4 | Badges, chips, tags, tooltip |
| `--v-radius-md` | 6 | Inputs, buttons (default), alerts, dropdown |
| `--v-radius-lg` | 8 | Buttons large, button `shape_square` |
| `--v-radius-xl` | 12 | Cards, feature items, dialog |
| `--v-radius-2xl` | 16 | Pricing tier, hero card, drawer |
| `--v-radius-full` | 9999px | Pill / circle (intencional) |

---

## Composite tokens (component sizing — used inside CSS Modules)

Five size tiers — `ss, sm, md, lg, xl` — pre-composed from the graded scales. Use these instead of recomposing manually.

### Field (button, input, select, etc.)

| Tier | height | padding | font | size (inner) |
|---|---|---|---|---|
| `ss` | 24px | 12px | `caption` (12) | 20px |
| `sm` | 32px | 12px | `body-sm` (14) | 24px |
| `md` (default) | 40px | 16px | `body-sm` (14) | 24px |
| `lg` | 40px | 20px | `body` (16) | 32px |
| `xl` | 48px | 24px | `body` (16) | 40px |

```css
height: var(--v-field-md-height);
padding-inline: var(--v-field-md-padding);
font-size: var(--v-field-md-font);
```

### Avatar

| Tier | size | font |
|---|---|---|
| `ss` | 24px | `caption` (12) |
| `sm` | 32px | `caption` (12) |
| `md` | 48px | `body` (16) |
| `lg` | 64px | `title` (24) |
| `xl` | 96px | `title-lg` (32) |

### Badge

| Tier | font | padding |
|---|---|---|
| `ss` | `caption` (12) | 8px |
| `sm` | `caption` (12) | 12px |
| `md` | `body-sm` (14) | 12px |
| `lg` | `body` (16) | 16px |
| `xl` | `body-lg` (18) | 16px |

### Icon

| Tier | size |
|---|---|
| `ss` | 4px |
| `sm` | 8px |
| `md` | 12px |
| `lg` | 16px |
| `xl` | 20px |

(Icon size composite is intentionally small because icons usually sit inside fields/buttons; for standalone icons use `--v-width-N` directly.)

---

## Color sub-token model

Every named color expands to **6 sub-tokens** via `parseColor()`:

| Sub-token | Purpose | Auto-derivation if not overridden |
|---|---|---|
| `fill` | base color (button bg, badge fill, primary text) | required input |
| `foreground` | text/icon on top of `fill` | `var(--v-b)` if fill is light, else `var(--v-w)` |
| `hover` | `:hover` state | `safeLighten(fill, +20%)` |
| `active` | `:active` / `[aria-selected]` | `safeLighten(fill, +40%)` |
| `subdued` | de-emphasized (disabled bg, muted text) | `safeLighten(fill, +50%)` |
| `border` | subtle border on a fill of this color | `safeLighten(fill, +8%)` |

**Always pair them correctly:**
```css
.solid {
  background-color: var(--v-primary-fill);
  color: var(--v-primary-foreground);
  border: 1px solid var(--v-primary-border);
}
.solid:not(:disabled):hover { background-color: var(--v-primary-hover); }
.solid:not(:disabled):active { background-color: var(--v-primary-active); }
.solid[aria-disabled="true"] { background-color: var(--v-primary-subdued); }
```

### Color names

| Name | Required? | Use | Notes |
|---|---|---|---|
| `background` | yes | page bg, neutral surfaces | |
| `secondary` | yes | subtle surfaces, borders, muted fills, disabled | |
| `primary` | yes | foreground text, neutral CTAs, default icons/borders | neutral-strong (near-black/white) |
| `destructive` | no (default) | errors, delete buttons, invalid states | red — same in light/dark |
| `link` | no (default) | inline hyperlinks | blue — same in light/dark |
| `success` | no (default) | success feedback (Toast/Progress success variant, confirmation states) | green — same in light/dark |
| `warning` | no (default) | warning feedback (Toast warning, caution states) | amber — same in light/dark |
| `info` | no (default) | informational feedback (Toast info, search-match highlights) | blue — same in light/dark |
| `accent` | no (optional) | brand-vivid CTA, popular badge, focus rings, eyebrow dot | falls back to `primary` |

### `accent` fallback contract — REQUIRED

Apps that don't define `accent` get `undefined` on those vars. **Every accent reference in CSS must chain a fallback:**
```css
background-color: var(--v-accent-fill, var(--v-primary-fill));
color:            var(--v-accent-foreground, var(--v-primary-foreground));
border-color:     var(--v-accent-border, var(--v-primary-border));
```

---

## Motion

Role-based duration scale + base easing/delay. Pick the tier that matches the interaction; don't tokenize one-offs.

```
--v-motion-instant   0ms       (micro acks, debug)
--v-motion-fast      150ms     hover, focus ring
--v-motion-default   250ms     state transitions, accordion expand
--v-motion-slow      400ms     drawer slide, page reveal
--v-motion-slower    600ms     orchestrated/sequenced reveals

--v-motion-duration  250ms     alias of `default` (legacy)
--v-motion-timing    cubic-bezier(0.4, 0, 0.2, 1)
--v-motion-delay     0ms
```

Sempre nomeie a propriedade no `transition` — nunca `transition: all`:
```css
transition: background-color var(--v-motion-default) var(--v-motion-timing),
            color            var(--v-motion-default) var(--v-motion-timing);
```

---

## Page-level tokens

Shipped by `@vireya/core` (defaults below) and used as the outer-gutter / max-width contract for blocks. Apps may override on `body` in their globals.css if they want a different rhythm.

```
--v-page-padding     clamp(16px, 5vw, 32px)
--v-page-max-width   1200px
```

Apps that want a wider canvas (e.g. `1520px`) override on `body`:
```css
body {
  --v-page-max-width: 1520px;
  --v-page-padding: var(--v-width-16);
}
```

---

## Content-width tokens (block inner content)

For inner-content max-widths (hero subtitle, section description, FAQ list, contact form). Page-level wrappers use `--v-page-max-width` instead.

| Token | Value | Use |
|---|---|---|
| `--v-content-width-sm` | `560px` | Compact inner content (single contact form, narrow card) |
| `--v-content-width-md` | `720px` | Standard section header (title + description), most CTA blocks |
| `--v-content-width-lg` | `880px` | Wide hero copy, two-column splits, large quotes |

---

## Elevation

Low-elevation system: borders + tone over shadows. Reach for the right tier:

| Token | Value | Use |
|---|---|---|
| `--v-elevation-rest` | `none` | Resting cards — use border + tone, NOT shadow |
| `--v-elevation-hover` | `0 8px 24px -12px rgba(0,0,0,.08)` | Hover lift on cards / tier cards |
| `--v-elevation-popover` | `0 8px 24px -8px rgba(0,0,0,.12)` | Popovers, dropdowns, tooltips |
| `--v-elevation-modal` | `0 24px 48px -16px rgba(0,0,0,.20)` | Modals, drawers, sheets |

Forbidden: multi-layer shadows (`box-shadow: 0 1px 2px ..., 0 4px 8px ...`), warm tints (`rgba(20, 10, 0, .1)` — use pure black), resting shadows on non-floating surfaces.

---

## Common usage cheat sheet — "I want X, paste Y"

A reverse-lookup index. Match your intent on the left → use the tokens on the right. **Every right-hand value is a complete CSS line, ready to paste.**

### Surfaces & containers

| Intent | Tokens to use |
|---|---|
| Page background (default) | `background: var(--v-background-fill);` |
| Subtle/muted background (sidebar, card-inside-card, table header) | `background: var(--v-secondary-fill);` |
| Primary surface (sticky CTA strip, dark bg in light mode) | `background: var(--v-primary-fill); color: var(--v-primary-foreground);` |
| Standard card / tile (resting) | `border: 1px solid var(--v-secondary-border); background: var(--v-background-fill); border-radius: var(--v-radius-xl);` |
| Card hover lift | `transform: translateY(-2px); border-color: var(--v-secondary-active); box-shadow: var(--v-elevation-hover);` |
| Floating layer (popover, dropdown) | `border: 1px solid var(--v-secondary-border); background: var(--v-background-fill); box-shadow: var(--v-elevation-popover); border-radius: var(--v-radius-lg);` |
| Modal / drawer | `background: var(--v-background-fill); box-shadow: var(--v-elevation-modal); border-radius: var(--v-radius-2xl);` |

### Text

| Intent | Tokens to use |
|---|---|
| Body default | `color: var(--v-background-foreground); font-size: var(--v-font-size-body); line-height: var(--v-leading-relaxed);` |
| Muted body / description / caption | `color: var(--v-secondary-active);` *(NOT `--v-secondary-fill` — `-active` is the muted text on neutral)* |
| Strong heading text | `color: var(--v-background-foreground); font-weight: var(--v-font-weight-semibold); line-height: var(--v-leading-snug);` |
| Eyebrow / kicker (mono uppercase) | `font-family: var(--v-font-family-mono); font-size: var(--v-font-size-caption); text-transform: uppercase; letter-spacing: var(--v-tracking-eyebrow); color: var(--v-secondary-active);` |
| Inline code / kbd | `font-family: var(--v-font-family-mono); font-size: var(--v-font-size-body-sm); padding: var(--v-height-2) var(--v-width-6); border-radius: var(--v-radius-sm); background: var(--v-secondary-fill); color: var(--v-background-foreground);` |
| Link text (inline) | `color: var(--v-link-fill);` and `text-decoration-color: var(--v-link-border);` |
| Destructive text (error helper, validation) | `color: var(--v-destructive-fill);` |

### Borders

| Intent | Tokens to use |
|---|---|
| Default 1px border (most common) | `border: 1px solid var(--v-secondary-border);` |
| Stronger border (focused/active card) | `border: 1px solid var(--v-secondary-active);` |
| Soft divider line between sections | `border-top: 1px solid var(--v-secondary-border);` |
| Dashed / decorative divider | `border-top: 1px dashed var(--v-secondary-border);` |
| Destructive border (invalid input) | `border-color: var(--v-destructive-fill);` |

### Buttons / CTAs (when you can't use `<Button>`)

| Intent | Tokens to use |
|---|---|
| Primary solid CTA | `background: var(--v-primary-fill); color: var(--v-primary-foreground); border: 1px solid var(--v-primary-border);` + hover `background: var(--v-primary-hover);` |
| Outlined secondary | `background: transparent; color: var(--v-background-foreground); border: 1px solid var(--v-secondary-border);` + hover `background: var(--v-secondary-fill);` |
| Ghost link-button | `background: transparent; color: var(--v-secondary-active); border: none;` + hover `color: var(--v-background-foreground);` |
| Accent CTA (branded) | `background: var(--v-accent-fill, var(--v-primary-fill)); color: var(--v-accent-foreground, var(--v-primary-foreground));` |
| Destructive button | `background: var(--v-destructive-fill); color: var(--v-destructive-foreground);` |
| Disabled (any) | `opacity: 0.5; cursor: not-allowed;` (no separate gray) |

### Pills / chips / badges

| Intent | Tokens to use |
|---|---|
| Neutral pill (mono label, count) | `padding: var(--v-height-2) var(--v-width-8); border-radius: var(--v-radius-full); background: var(--v-secondary-fill); color: var(--v-secondary-active); font-family: var(--v-font-family-mono); font-size: var(--v-font-size-caption);` |
| Accent badge ("Most popular", "New") | `background: var(--v-accent-fill, var(--v-primary-fill)); color: var(--v-accent-foreground, var(--v-primary-foreground));` |
| Status: success / warning / info | swap `--v-{success,warning,info}-fill` and matching `-foreground` |
| Status with translucent fill | `background: color-mix(in srgb, var(--v-success-fill) 12%, transparent); color: var(--v-success-fill); border: 1px solid color-mix(in srgb, var(--v-success-fill) 30%, transparent);` |

### Spacing patterns (compose from the ladder)

| Intent | Token |
|---|---|
| Inline icon ↔ text gap | `gap: var(--v-width-6);` or `-8` |
| Form label ↔ input | `gap: var(--v-height-6);` |
| Card inner padding (small) | `padding: var(--v-height-12) var(--v-width-16);` |
| Card inner padding (standard) | `padding: var(--v-height-20) var(--v-width-24);` |
| Section header → content | `gap: var(--v-height-48);` |
| Title group → action button | `gap: var(--v-width-32);` |
| Outer page gutter (any section) | `padding-inline: var(--v-page-padding);` |
| Vertical rhythm between sections | section padding-block `--v-height-32` (gives 64px between siblings) |

### Radii — match parent → child

| Container | Children should use |
|---|---|
| Card / tile (`--v-radius-xl`) | Buttons, inputs `lg` (8px); badges `sm` (4px); nested icon tiles `md` (6px) |
| Hero card / pricing tier (`--v-radius-2xl`) | Buttons `lg`; badges `md` |
| Input (`--v-radius-md`) | Inline icons inside: no radius (or `xs`) |
| Pill/avatar (`--v-radius-full`) | Children should not have radius (children are usually a single glyph) |

### Motion

| Intent | Tokens to use |
|---|---|
| Default state transition (color, bg, opacity) | `transition: background-color var(--v-motion-default) var(--v-motion-timing);` *(name the property!)* |
| Hover lift transform | `transition: transform var(--v-motion-default) var(--v-motion-timing);` + gate by `@media (hover: hover) and (pointer: fine)` |
| Reduced motion guard | wrap meaningful animation in `@media (prefers-reduced-motion: reduce) { .x { transition: none; animation: none; } }` |

### `color-mix` recipes (alpha without `rgba`)

```css
/* 12% tint of primary as a translucent bg */
background: color-mix(in srgb, var(--v-primary-fill) 12%, transparent);

/* 60% opacity over the page bg (useful for backdrops) */
background: color-mix(in srgb, var(--v-primary-fill) 60%, transparent);

/* Soft accent ring overlay */
background: color-mix(in srgb, var(--v-accent-fill, var(--v-primary-fill)) 30%, transparent);
```

This replaces every `rgba(0,0,0,0.X)` and any hardcoded translucency.

For gradient spotlights, frosted overlays, and accent glows that compose `color-mix` into the canonical Vireya visual depth treatment, see `design.md` → **"Visual depth — gradients, spotlights, masks"**. Six recipes covering top-spotlight heroes, bottom-spotlight cards, illustration masks, frosted-glass UI, accent glows, and halo text-shadows.

---

## Current limitations

- `--v-density-multiplier` — deferred until a second consumer needs it.
