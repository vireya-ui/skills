# Vireya blocks — compound API reference

Pre-composed landing-page sections in `@vireya/blocks`. All blocks use a compound API (`<Block.Root>` + items). Common props recur across families: `eyebrow`, `title`, `description` (all `ReactNode`); `variant: "default" | "shaded"` for background variants where applicable.

**`emphasis: "primary" | "accent"` (default `"accent"`) is opt-in by design** — only the 8 blocks with concrete accent-eligible visual elements expose it: `hero/{centered,splitVisual}`, `pricing/{tiers,twoTier}`, `feature/{steps,checkList,highlights}`, `eyebrow`. The other 19 blocks intentionally lack `emphasis` because they have no internal element where `accent` vs `primary` would have a visible effect (footers/navbars/logo clouds are neutral chrome; CTAs use `variant`; FAQ/contact/hero-minimal/etc. defer to the `eyebrow` slot — pass `<Eyebrow emphasis="accent">` directly when you need accent there).

For prop details beyond what's listed here, open the DTS at `node_modules/@vireya/blocks/dist/components/<family>/<name>/index.d.ts` (or use IDE go-to-definition on the import).

---

## navbar

### `@vireya/blocks/navbar/simple` — `NavbarSimple`
- `Root`: `position?: "sticky" | "static"` (default `"sticky"`)
- `Logo`, `Link` (`href`, `children`), `Actions` (right-side button slot)

### `@vireya/blocks/navbar/dropdown` — `NavbarDropdown`
- `Root`: `position?: "sticky" | "static"`
- `Logo`, `Link`, `Actions`
- `Dropdown`: `label: ReactNode`, `columns?: number` — opens a multi-column menu
- `MenuLink` (inside `Dropdown`): `href`, `description?`, `icon?`, `children` (title)

---

## hero

All four accept `eyebrow?`, `title` (required), `description?`, `className?`, `children?` (action buttons), and extend `HTMLAttributes<HTMLElement>`.

### `@vireya/blocks/hero/centered` — `HeroCentered`
- Extra: `badge?`, `align?: "center" | "left"` (default `"center"`), `emphasis?: "primary" | "accent"`

### `@vireya/blocks/hero/minimal` — `HeroMinimal`
- No badge, no emphasis. Just eyebrow → title → description → actions.

### `@vireya/blocks/hero/backgroundImage` — `HeroBackgroundImage`
- Extra: `imageSrc` (required), `imageAlt?`, `overlay?: number` (0–1, default `0.5`), `badge?`, `align?`

### `@vireya/blocks/hero/splitVisual` — `HeroSplitVisual`
- Extra: `badge?`, `reverse?: boolean` (swap text/visual sides), `emphasis?`
- Subcomponent: `Visual` — slot for the right-side (or left-side if `reverse`) media

---

## logoCloud

### `@vireya/blocks/logoCloud/grid` — `LogoCloudGrid`
- `Root`: `eyebrow?`, `title?`, `columns?: 3 | 4 | 5 | 6` (default `4`), `bordered?` (default `true`), `grayscale?` (default `true`)
- `Logo`: `src` (required), `alt` (required), `href?`

### `@vireya/blocks/logoCloud/row` — `LogoCloudRow`
- `Root`: `eyebrow?`, `title?`, `grayscale?` (default `true`)
- `Logo`: same as above

---

## feature

### `@vireya/blocks/feature/alternating` — `FeatureAlternating`
- `Root`: `eyebrow?`, `title?`, `description?`
- `Row`: `eyebrow?`, `title` (required), `description?` — alternates left/right automatically
- `Visual`: media slot inside each `Row`

### `@vireya/blocks/feature/checkList` — `FeatureCheckList`
- `Root`: `eyebrow?`, `title?`, `description?`, `icon?: ReactNode` (custom check icon, defaults to a built-in), `columns?: 1 | 2` (default `1`), `emphasis?: "primary" | "accent"`
- `Item`: `title` (required), `description?`

### `@vireya/blocks/feature/highlights` — `FeatureHighlights`
- `Root`: `eyebrow?`, `title?`, `description?`, `columns?: 2 | 3 | 4` (default `4`), `emphasis?`
- `Item`: `icon?`, `title` (required), `description` (required)

### `@vireya/blocks/feature/iconGrid` — `FeatureIconGrid`
- `Root`: `eyebrow?`, `title?`, `description?`, `columns?: 2 | 3 | 4` (default `3`)
- `Item`: `icon?`, `title` (required), `description?`

### `@vireya/blocks/feature/steps` — `FeatureSteps`
- `Root`: `eyebrow?`, `title?`, `description?`, `emphasis?`
- `Item`: `number?`, `title` (required), `description?`, `action?` (per-step CTA slot)

---

## pricing

### `@vireya/blocks/pricing/tiers` — `PricingTiers`
- `Root`: `eyebrow?`, `title?`, `description?`, `emphasis?: "primary" | "accent"`
- `Tier`: `name` (required), `price` (required), `period?`, `description?`, `highlighted?: boolean` (highlights with accent ring + badge), `highlightedLabel?: ReactNode` (defaults to `"Most popular"`)
- `Feature`: checklist row inside a tier — pass children as the feature label

### `@vireya/blocks/pricing/twoTier` — `PricingTwoTier`
- `Root`: same as `tiers`
- `Tier`: same shape (`name`, `price`, `period?`, `description?`, `highlighted?: boolean`)
- `Feature`: same

---

## cta

### `@vireya/blocks/cta/banner` — `CTABanner`
- `eyebrow?`, `title` (required), `description?`, `variant?: "default" | "shaded"` (default `"default"`), `children` (action buttons)

### `@vireya/blocks/cta/centered` — `CTACentered`
- Same shape as `banner`, centered layout.

### `@vireya/blocks/cta/inlineEmail` — `CTAInlineEmail`
- `Root`: `eyebrow?`, `title` (required), `description?`, `disclaimer?`, `variant?: "default" | "shaded"` (default `"shaded"`)
- `Form`: `onSubmit?: (email: string) => void | Promise<void>` — manages internal state
- `Input`: `placeholder?: string` (default `"you@example.com"`)
- `Submit`: extends `Button` props

---

## faq

### `@vireya/blocks/faq/accordion` — `FAQAccordion`
- `Root`: `eyebrow?`, `title?`, `description?`, `openMultiple?: boolean` (default `false`), `surface?: "outlined" | "ghosted"` (default `"outlined"`)
- `Item`: `question` (required), `children` = answer

### `@vireya/blocks/faq/twoColumn` — `FAQTwoColumn`
- `Root`: `eyebrow?`, `title` (required), `description?`, `openMultiple?`, `surface?: "outlined" | "ghosted"` (default `"ghosted"`)
- `Item`: same as above
- `Aside`: left-side content slot (e.g. extra description, contact CTA)

---

## testimonials

### `@vireya/blocks/testimonials/single` — `TestimonialsSingle`
- `eyebrow?`, `logo?`, `quote` (required), `author` (required), `role?`, `avatar?`, `variant?: "default" | "shaded"` (default `"default"`)

### `@vireya/blocks/testimonials/grid` — `TestimonialsGrid`
- `Root`: `eyebrow?`, `title?`, `description?`, `columns?: 2 | 3` (default `3`)
- `Card`: `quote` (required), `author` (required), `role?`, `avatar?`

---

## contact

### `@vireya/blocks/contact/simple` — `ContactSimple`
- `Root`: `eyebrow?`, `title` (required), `description?`, `disclaimer?`, `variant?: "default" | "shaded"`
- `Form`: `onSubmit?: (values: { name, email, message }) => void | Promise<void>`
- `Name`, `Email`, `Message`, `Submit`: pre-styled fields wired to the form's internal state

### `@vireya/blocks/contact/split` — `ContactSplit`
- `Root`: `eyebrow?`, `title` (required), `description?`
- `Info` (left): container for `InfoItem` rows
- `InfoItem`: `icon?`, `label` (required), `value` (required), `href?` (turns `value` into a link)
- `Form` (right): `onSubmit?: (values: { name, email, subject, message }) => void | Promise<void>`
- `FieldRow`: groups `Name`/`Email` side-by-side
- `Name`, `Email`, `Subject`, `Message`, `Submit`: wired fields

---

## footer

### `@vireya/blocks/footer/minimal` — `FooterMinimal`
- `Root`, `Brand` (logo slot), `Link` (`href`, `children`), `Trailing` (right-side slot for socials/legal)

### `@vireya/blocks/footer/columns` — `FooterColumns`
- `Root`, `Brand`
- `Column`: `title` (required), `children` = `Link` items
- `Link`: `href`, `children`
- `Bottom`: footer-bottom slot (copyright, legal links)

---

## eyebrow (utility, also reused inside other blocks)

### `@vireya/blocks/eyebrow` — `Eyebrow`, `EyebrowGroup`
- `Eyebrow`: `variant?: "plain" | "muted" | "pill" | "dot"` (default `"plain"`), `size?: number` (default `12`), `weight?: "regular" | "medium" | "semibold" | "bold"` (default `"medium"`), `emphasis?: "primary" | "accent"` (default `"accent"`), `children` (required)
- `EyebrowGroup`: container with alignment support

When a block accepts an `eyebrow?: ReactNode` prop, you can pass either a plain string or a fully-composed `<Eyebrow variant="dot" emphasis="accent">Featured</Eyebrow>`.

---

## Patterns

- **Variant dedup:** if two showcases of a block differ only by an opt-in prop (e.g. `popular`, `bordered`), don't duplicate them. The block already supports both.
- **Emphasis prop:** defaults to `"accent"`. Pass `"primary"` for sections that should not draw promotional attention (footer, contact, neutral CTAs).
- **`variant: "shaded"`:** uses `--v-secondary-fill` as background — useful to break visual rhythm between sections without leaving the token system.
- **Children-as-actions:** most blocks treat `children` as the action button slot. Use `<Button asChild><Link href="..." /></Button>` to wire to Next.js routing.
