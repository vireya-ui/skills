# Vireya 0.0.x → 0.1.0 migration guide

For agents helping a consumer upgrade their app from a pre-0.1.0 `@vireya/*` to 0.1.0+. The token API was reshaped — this is a **breaking** release. Skim §1 to understand scope, run the codemod (§3), then manually patch the cases in §4.

---

## 1. What changed

| Area | Before | After |
|---|---|---|
| Font scale | `--v-font-N` (22 graded tokens, linear 2px steps 1–64) | `--v-font-size-{name}` (9 semantic: `caption`, `body-sm`, `body`, `body-lg`, `title-sm`, `title`, `title-lg`, `display`, `display-lg`) |
| `<Text size>` prop | `"xxs" \| "xs" \| "ss" \| "sm" \| "md" \| "lg" \| "sl" \| "xl" \| "xxl"` + numeric (`size={32}`) | `"caption" \| "body-sm" \| "body" \| "body-lg" \| "title-sm" \| "title" \| "title-lg" \| "display" \| "display-lg"`. Numeric escape removed. |
| Radius scale | `--v-radius-N` (17 graded tokens, 1–32) + `--v-radius` singleton | `--v-radius-{xs,sm,md,lg,xl,2xl,full}` (7 semantic). Singleton removed. |
| Spacing scale | `1, 2, 3, 4, 5, 6, 8, 10, 12, 14, 16, 18, 20, 22, 24, 28, 32, 38, 42, 48, 64, 96, 128` | `1, 2, 4, 6, 8, 10, 12, 14, 16, 18, 20, 22, 24, 32, 40, 48, 64, 80, 96, 128`. Removed: 3, 5, 28, 38, 42. Added: 80. |
| Line-height | Inline literals (`line-height: 1.55`) | `--v-leading-{tight,snug,normal,relaxed}` (new) |
| Letter-spacing | Inline literals (`letter-spacing: -0.02em`) | `--v-tracking-{tight,normal,wide,eyebrow}` (new) |
| Motion | `--v-motion-duration` only | `--v-motion-{instant,fast,default,slow,slower}` role scale. `--v-motion-duration` kept as legacy alias of `default`. |
| Elevation | Inline `box-shadow` literals | `--v-elevation-{rest,hover,popover,modal}` (new) |
| Page tokens | Defined in app `globals.css` | Shipped by core: `--v-page-padding`, `--v-page-max-width`. Apps may still override on `body`. |
| Default font family | `Arial, Helvetica, sans-serif` | `Inter, system-ui, -apple-system, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif` |
| Field default height | `md = 38px`, `ss = 28px` | `md = 40px`, `ss = 24px`, `lg/md` collapse to same height with different padding+font |

## 2. Visual diff to expect

Snapping is intentional but not invisible. Expect these per-element shifts after the codemod runs:

| Old token | New | Δ px |
|---|---|---|
| `--v-font-40` | `display` (48) | **+8** |
| `--v-font-36`, `<Text size="xxl">` | `title-lg` (32) | −4 |
| `--v-font-28`, `<Text size="sl">` | `title` (24) | −4 |
| `--v-font-26` | `title` (24) | −2 |
| `--v-font-44` | `display` (48) | +4 |
| `--v-radius-10` | `xl` (12) | +2 |
| `--v-radius-{18..32}` | `2xl` (16) | up to −16 |
| `--v-height/width-28` | 32 | +4 |
| `--v-height/width-42` | 40 | −2 |
| Field `md` height | 40 | +2 |
| Field `ss` height | 24 | −4 |
| Sub-12px font literals (1, 2, 4, 6, 8, 10) | `caption` (12) | up to +20% |

The `--v-font-40 → 48` is the single biggest jump — review hero/pricing components that depended on it.

## 3. Run the codemod

If the consumer's repo is in this monorepo, the codemod is at `scripts/migrate-tokens.mjs`. Otherwise the consumer can paste the same script into their own repo (no dependencies; pure Node). It rewrites:

- `var(--v-font-N)` → `var(--v-font-size-<semantic>)` (snap-to-nearest)
- `var(--v-radius-N)` → `var(--v-radius-<semantic>)`
- bare `var(--v-radius)` → `var(--v-radius-md)` (the old singleton was 6px)
- `var(--v-height-N)` / `var(--v-width-N)` for N ∈ {3, 5, 28, 38, 42} → nearest valid step
- `<Text size="..."` JSX/MDX strings → semantic
- `<Text size={N}>` JSX/MDX numeric → semantic via FONT_MAP

Usage:

```bash
# Dry-run first to inspect impact
node scripts/migrate-tokens.mjs src apps --dry-run

# Apply
node scripts/migrate-tokens.mjs src apps
```

It scans `.css`, `.tsx`, `.ts`, `.jsx`, `.js`, `.mjs`, `.md`, `.mdx`. The `<Text size>` rewriter only touches `<Text` opener tokens — it ignores `<Icon size={14}>`, `<Check size={14}>`, etc.

The script outputs a per-file report (`font=N radiusN=N radiusBare=N spacing=N textProp=N`) and warns on any unmapped values. **Common unmapped fonts**: 11, 13, 15, 17, 19, 72 — extend `FONT_MAP` in the script for any pixel value the codebase uses that isn't pre-mapped, or accept the snap-to-nearest defaults.

## 4. Manual fixes the codemod can't do

The codemod handles `<Text>` JSX, but it **can't see**:

### 4.1 Default-prop destructure in component wrappers

```tsx
// Before
const CardTitle = ({ size = 18, ... }) => <Text size={size} ... />;
const PopoverTitle = ({ size = 16, ... }) => <Text size={size} ... />;
const AlertTitle = ({ size = 18, ... }) => <Text size={size} ... />;
const Eyebrow = ({ size = "xxs", ... }) => <Text size={size} ... />;

// After
const CardTitle = ({ size = "body-lg", ... }) => <Text size={size} ... />;
const PopoverTitle = ({ size = "body", ... }) => <Text size={size} ... />;
const AlertTitle = ({ size = "body-lg", ... }) => <Text size={size} ... />;
const Eyebrow = ({ size = "caption", ... }) => <Text size={size} ... />;
```

Find them with: `rg -nE 'size\s*=\s*[0-9]+|size\s*=\s*"(xxs|xs|ss|sm|md|lg|sl|xl|xxl)"' --type tsx --type ts`. Filter out matches on `<Icon>`, `<Skeleton>`, `<PoolProvider>`, etc.

### 4.2 Storybook `args: { size: "..." }` and `argTypes.options`

The codemod's JSX rewriter looks for `<Text` opener — args object literals don't match. Update story files manually:

```tsx
// Before
argTypes: { size: { options: ["xxs", "xs", "ss", "sm", "md", "lg", "sl", "xl", "xxl"], ... } }
args: { size: "ss" }

// After
argTypes: { size: { options: ["caption", "body-sm", "body", "body-lg", "title-sm", "title", "title-lg", "display", "display-lg"], ... } }
args: { size: "body" }
```

### 4.3 `calc()` math with `var(--v-radius-1)` for hairline subtraction

`--v-radius-1` was 1px — used as a hairline in `calc(var(--v-radius) - var(--v-radius-1))`. The codemod snaps it to `--v-radius-xs` (2px), which changes the result by 1px.

**Fix: use the literal `1px` for hairline-width math:**

```css
/* Before */
border-radius: calc(var(--v-radius) - var(--v-radius-1));   /* = 5px */

/* After */
border-radius: calc(var(--v-radius-md) - 1px);              /* = 5px (preserved) */
```

`calc()` math with `var(--v-radius)` divided by a number (e.g. `/ 3`) is preserved automatically by the codemod — `--v-radius` (was 6px) → `--v-radius-md` (still 6px), so divisions yield the same result.

### 4.4 Inline `line-height` and `letter-spacing` literals

The codemod doesn't rewrite these (no semantic mapping that's safe to automate). After running, `grep -RnE 'line-height: 1\.[0-9]+\b|letter-spacing: -?0\.[0-9]+em\b' src/` and replace by role:

| Before | After |
|---|---|
| `line-height: 1.05` | `line-height: var(--v-leading-tight)` |
| `line-height: 1.15` | `line-height: var(--v-leading-snug)` |
| `line-height: 1.4` | `line-height: var(--v-leading-normal)` |
| `line-height: 1.55` | `line-height: var(--v-leading-relaxed)` |
| `letter-spacing: -0.02em` (or `-0.025em`, `-0.03em`) | `letter-spacing: var(--v-tracking-tight)` |
| `letter-spacing: 0.04em` | `letter-spacing: var(--v-tracking-wide)` |
| `letter-spacing: 0.08em` | `letter-spacing: var(--v-tracking-eyebrow)` |

### 4.5 Inline `box-shadow` literals

If the consumer hardcoded shadow recipes, swap to elevation tokens:

```css
/* Before */
box-shadow: 0 8px 24px -12px rgba(0, 0, 0, 0.08);   /* hover lift */
box-shadow: 0 8px 24px -8px rgba(0, 0, 0, 0.12);    /* popover */
box-shadow: 0 24px 48px -16px rgba(0, 0, 0, 0.20);  /* modal */

/* After */
box-shadow: var(--v-elevation-hover);
box-shadow: var(--v-elevation-popover);
box-shadow: var(--v-elevation-modal);
```

### 4.6 Inline motion duration literals

```css
/* Before */
transition: background-color 150ms cubic-bezier(0.4, 0, 0.2, 1);
transition: transform 400ms ease-out;

/* After (pick the role tier) */
transition: background-color var(--v-motion-fast) var(--v-motion-timing);
transition: transform var(--v-motion-slow) var(--v-motion-timing);
```

### 4.7 Cleanup app `globals.css`

If the app defined `--v-page-padding` / `--v-page-max-width` in `globals.css` because pre-0.1.0 didn't ship them, those overrides are still valid (they win over core defaults). If the values are the **same as the new defaults** (`clamp(16px, 5vw, 32px)` and `1200px`), delete them.

## 5. Verify

After codemod + manual passes:

```bash
# 1. No legacy tokens left in source
grep -RnE 'var\(--v-(font|radius)-[0-9]+\)|var\(--v-radius\)|var\(--v-(height|width)-(3|5|28|38|42)\)' src/ apps/

# 2. No legacy <Text> sizes left
grep -RnE 'size=("xxs"|"xs"|"ss"|"sl"|"xxl")' src/ apps/

# 3. Type check
pnpm exec tsc --noEmit

# 4. Build
pnpm build

# 5. Visual review — toggle dark theme; check hero, pricing, cards, fields
```

If a Storybook + Chromatic setup exists, run the visual diff. Expect intentional shifts per §2 — those are not regressions.

## 6. If you can't migrate yet

The migration is breaking. To stay on a pre-0.1.0 version, pin `@vireya/core`, `@vireya/ui`, `@vireya/next`, `@vireya/blocks` to the last 0.0.x release in `package.json` and avoid `pnpm update --latest`. The 0.0.x line is unmaintained going forward — only security patches.

## 7. Reporting issues

If a snap doesn't match design intent (e.g. the −4px on `title-lg` breaks a hero you can't reshape), the right fix is:
- Override the specific element with `font-size: <inline-px>` (last resort).
- Or compose a custom typography role using `<Text>` props + a custom CSS Module.
- Then file an issue with the specific role + screenshot — token additions need consumer evidence.

Don't hand-edit `--v-font-size-*` values in node_modules. They're shared across every `@vireya/*` package and will revert on next install.
