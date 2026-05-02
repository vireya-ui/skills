# Landing page recipes

Practical, copy-pasteable patterns for assembling a marketing/landing page with Vireya. Each recipe shows the **minimum viable shape**; expand from there. Every recipe respects the NON-NEGOTIABLE rules in `SKILL.md` (tokens, no barrel imports, `"use client"` only when needed).

> **Why a recipes file?** Vireya gives you primitives and blocks — but assembling them into a coherent landing has its own conventions (section shell, CMS-style content, accent scoping, RSC boundary calls). These are battle-tested patterns; reach for them before improvising.

---

## 1. Section shell primitive (build once, reuse everywhere)

Every section on a landing repeats the same outer rhythm: outer gutter, max-width, header (eyebrow + title + description), content area. Build a tiny `Section` namespace in your app and every section component composes it.

```tsx
// app/components/section.tsx
import { Text } from "@vireya/ui/typography/text";
import { Eyebrow } from "@vireya/ui/typography/eyebrow";
import clsx from "clsx";
import styles from "./section.module.css";

const Root: React.FC<React.ComponentProps<"section">> = ({
  className, children, ...props
}) => (
  <section className={clsx(styles.root, className)} {...props}>{children}</section>
);

const Header: React.FC<
  React.ComponentProps<"header"> & { action?: React.ReactNode }
> = ({ className, children, action, ...props }) => (
  <header className={clsx(styles.header, className)} {...props}>
    <div className={styles.titling}>{children}</div>
    {action && <div className={styles.headerAction}>{action}</div>}
  </header>
);

const SectionEyebrow: React.FC<
  Omit<React.ComponentProps<typeof Eyebrow>, "variant">
> = (props) => <Eyebrow variant="pill" {...props} />;

const Title: React.FC<
  Omit<React.ComponentProps<typeof Text>, "asElement" | "size" | "weight">
> = ({ className, ...props }) => (
  <Text asElement="h2" size={40} weight="semibold"
    className={clsx(styles.title, className)} {...props} />
);

const Description: React.FC<
  Omit<React.ComponentProps<typeof Text>, "asElement" | "type" | "size">
> = ({ className, ...props }) => (
  <Text asElement="p" size={18} type="muted"
    className={clsx(styles.description, className)} {...props} />
);

export const Section = { Root, Header, Eyebrow: SectionEyebrow, Title, Description };
```

```css
/* section.module.css */
.root {
  width: 100%;
  max-width: var(--v-page-max-width);
  margin: 0 auto;
  /* 32px each side -> 64px between consecutive sections */
  padding: var(--v-height-32) var(--v-page-padding);
  display: flex;
  flex-direction: column;
  gap: var(--v-height-48);
  scroll-margin-top: var(--v-height-32);
}
.header {
  display: flex;
  align-items: flex-end;
  justify-content: space-between;
  gap: var(--v-width-32);
  flex-wrap: wrap;
  padding-bottom: var(--v-height-24);
  border-bottom: 1px solid var(--v-secondary-border);
}
.titling { display: flex; flex-direction: column; gap: var(--v-height-8); min-width: 0; }
.headerAction { flex-shrink: 0; white-space: nowrap; }
.title { line-height: 1.1; letter-spacing: -0.025em; }
.description { max-width: 64ch; line-height: 1.55; }

@media (max-width: 640px) {
  .root { gap: var(--v-height-32); }
  .title { font-size: var(--v-font-28) !important; }
}
```

Usage:
```tsx
<Section.Root>
  <Section.Header action={<Link href="/all">Browse all →</Link>}>
    <Section.Eyebrow>Catalog</Section.Eyebrow>
    <Section.Title>Every block you need</Section.Title>
    <Section.Description>Drop one in, swap the props, ship.</Section.Description>
  </Section.Header>
  <div>{/* section content */}</div>
</Section.Root>
```

---

## 2. CMS-style content pattern

Treat editable copy (titles, lists, FAQ, pricing tiers) as data, not JSX. Section components stay thin and data-driven; non-developers can edit a `.ts` file without touching React.

**Two rules:**
1. Content files export a typed interface + a frozen literal. No React imports.
2. **Icons go as string keys** — content files are CMS-portable, can't serialize React components.

```ts
// content/contact.ts
export type ContactInfoIcon = "mail" | "github" | "clock";

export interface IContactInfoItem {
  icon: ContactInfoIcon;
  label: string;
  value: string;
  href?: string;
}

export interface IContact {
  eyebrow: string;
  title: string;
  description: string;
  info: IContactInfoItem[];
}

export const contact: IContact = {
  eyebrow: "Contact",
  title: "Talk to us",
  description: "...",
  info: [
    { icon: "mail",   label: "Email",  value: "you@example.com", href: "mailto:..." },
    { icon: "github", label: "GitHub", value: "yourorg",         href: "https://..." },
  ],
};
```

```tsx
// app/components/sections/contact.tsx
"use client";
import { ContactSplit } from "@vireya/blocks/contact/split";
import { Mail, Github, Clock } from "lucide-react";
import type { ComponentType } from "react";
import { contact, type ContactInfoIcon } from "../content/contact";

const iconMap: Record<ContactInfoIcon, ComponentType<{ size?: number }>> = {
  mail: Mail, github: Github, clock: Clock,
};

export const ContactSection = () => (
  <ContactSplit.Root title={contact.title} description={contact.description}>
    <ContactSplit.Info>
      {contact.info.map((item) => {
        const Icon = iconMap[item.icon];
        return (
          <ContactSplit.InfoItem
            key={item.label}
            icon={<Icon size={18} />}
            label={item.label}
            value={item.value}
            href={item.href}
          />
        );
      })}
    </ContactSplit.Info>
    {/* ContactSplit.Form below */}
  </ContactSplit.Root>
);
```

Same shape applies to pricing tiers, FAQ items, real-world examples, stats — anything you'd put in a CMS.

---

## 3. Page composition order (B2B / devtool landing)

A proven order for the kind of product Vireya itself targets. Adapt to your funnel.

```
Hero               ← attention + value prop + 2 CTAs
Stats              ← thin proof bar (counts, license)
Catalog            ← what they get (blocks/charts/templates preview)
Templates          ← prebuilt screens
Real-world         ← who's already using it
Architecture       ← how it's built (3 packages, layered)
Theming            ← extend via tokens (illustrations)
Themes editor      ← interactive demo (peak engagement)
Install snippet    ← convinced? install
Pricing            ← OSS / Commercial / Enterprise
FAQ                ← objections, license, comparisons
Contact            ← talk to sales
```

**Why this order:** show before tell (catalog before architecture), peak engagement late (interactive theme editor right before the "install" ask), pricing/FAQ/contact at the bottom for users who scrolled all the way.

---

## 4. Hero CTA pair (primary + secondary)

```tsx
import { HeroCentered } from "@vireya/blocks/hero/centered";
import { Button } from "@vireya/ui/form/button";
import { Eyebrow } from "@vireya/ui/typography/eyebrow";
import Link from "next/link";

<HeroCentered.Root
  eyebrow={<Eyebrow variant="pill">120+ components</Eyebrow>}
  title={<>The design system that <span>ships</span></>}
  description="Built for speed, modularity, infinite customization."
>
  <Button size="lg" asChild>
    <Link href="/docs">Explore Documentation</Link>
  </Button>
  <Button variant="secondary" size="lg" surface="outlined" asChild>
    <Link href="#pricing">Commercial License</Link>
  </Button>
</HeroCentered.Root>
```

**Rules:**
- Never two `variant="primary"` buttons. Promote one (the actual decision), demote the other to `secondary` + `surface="outlined"` or `surface="ghosted"`.
- For accent-driven CTAs (promotional / branded moments), use `variant="accent"` instead of primary.
- Always wrap router links via `asChild + <Link>` — never invent an `href` prop on Button.

---

## 5. Pricing tier with a custom CTA per tier

```tsx
"use client";
import { PricingTiers } from "@vireya/blocks/pricing/tiers";
import { Button } from "@vireya/ui/form/button";
import Link from "next/link";

<PricingTiers.Root title="Pricing" description="...">
  <PricingTiers.Tier
    name="Open Source"
    price="Free"
    description="MIT licensed."
  >
    <PricingTiers.Feature>Full component library</PricingTiers.Feature>
    <PricingTiers.Feature>All blocks & charts</PricingTiers.Feature>
    <Button variant="secondary" surface="outlined" asChild>
      <Link href="#install">Get started</Link>
    </Button>
  </PricingTiers.Tier>

  <PricingTiers.Tier
    name="Commercial"
    price="Contact"
    highlighted
    highlightedLabel="Most popular"
    description="For teams shipping closed-source."
  >
    <PricingTiers.Feature>Everything in OSS</PricingTiers.Feature>
    <PricingTiers.Feature>Premium templates</PricingTiers.Feature>
    <Button variant="accent" surface="solid" asChild>
      <Link href="#contact">Talk to sales</Link>
    </Button>
  </PricingTiers.Tier>
</PricingTiers.Root>
```

**Notes:**
- Anything that isn't a `PricingTiers.Feature` becomes the tier's **action slot** — typically your CTA button.
- The highlighted tier auto-receives Root's `emphasis` (defaults to `accent`); pass `emphasis="primary"` on Root to mute the brand color.
- Use `highlighted` + `highlightedLabel` (NOT `popular`).

---

## 6. Install snippet — package-manager tabs + Shiki-rendered code

The install snippet section is server-side syntax-highlighted with Shiki, with a tiny client component for the copy interaction.

```ts
// app/lib/highlight.ts (or similar — server-only)
import { codeToHtml } from "shiki";

const cache = new Map<string, Promise<string>>();
export function highlightTsx(source: string) {
  const cached = cache.get(source);
  if (cached) return cached;
  const promise = codeToHtml(source, {
    lang: "tsx",
    themes: { light: "github-light", dark: "github-dark" },
    defaultColor: false,
  });
  cache.set(source, promise);
  return promise;
}
```

```tsx
// app/components/install.tsx (server async)
import { TabsRoot, TabsList, TabsTrigger, TabsContent } from "@vireya/ui/data/tab";
import { highlightTsx } from "../lib/highlight";
import { CopyButton } from "./copy-button";

export const InstallSection = async () => {
  const html = await highlightTsx(`import { Button } from "@vireya/ui/form/button";\nexport default () => <Button>Ship it</Button>;`);
  const commands = [
    { id: "pnpm", label: "pnpm", command: "pnpm add @vireya/ui" },
    { id: "npm",  label: "npm",  command: "npm install @vireya/ui" },
    { id: "yarn", label: "yarn", command: "yarn add @vireya/ui" },
  ];

  return (
    <TabsRoot defaultValue="pnpm" variant="group" layout={false}>
      <TabsList>{commands.map((c) =>
        <TabsTrigger key={c.id} value={c.id}>{c.label}</TabsTrigger>)}
      </TabsList>
      {commands.map((c) => (
        <TabsContent key={c.id} value={c.id}>
          <span>$ <code>{c.command}</code></span>
          <CopyButton value={c.command} />
        </TabsContent>
      ))}
      <div dangerouslySetInnerHTML={{ __html: html }} />
    </TabsRoot>
  );
};
```

```tsx
// app/components/copy-button.tsx
"use client";
import { useState } from "react";
import { Check, Copy } from "lucide-react";

export const CopyButton = ({ value }: { value: string }) => {
  const [copied, setCopied] = useState(false);
  return (
    <button onClick={async () => {
      await navigator.clipboard.writeText(value);
      setCopied(true);
      setTimeout(() => setCopied(false), 1600);
    }}>
      {copied ? <Check size={14} /> : <Copy size={14} />}
    </button>
  );
};
```

**Why this split:** Shiki is server-only (large bundle, Node-only). The async server section pre-renders the HTML; the copy button is a tiny client island. Same approach for any "code preview" block.

---

## 7. Stats bar (derived counts)

A thin, dense bar right after hero. Counts derive from data sources you already have (manifests, package.json, env), not hardcoded.

```tsx
import { manifest as blocksManifest } from "@vireya/blocks/showcase";

export const StatsSection = () => {
  const items = [
    { label: "Components", value: "120+" },
    { label: "Blocks",     value: `${blocksManifest.length}` },
    { label: "License",    value: "MIT + Commercial" },
  ];
  return (
    <dl className={styles.bar}>
      {items.map((item) => (
        <div key={item.label} className={styles.cell}>
          <dt><Text size={11} weight="medium">{item.label}</Text></dt>
          <dd><Text size={28} weight="semibold">{item.value}</Text></dd>
        </div>
      ))}
    </dl>
  );
};
```

```css
.bar {
  display: grid;
  grid-template-columns: repeat(4, 1fr);
  border: 1px solid var(--v-secondary-border);
  border-radius: var(--v-radius-12);
  background: var(--v-background-fill);
  overflow: hidden;
}
.cell {
  display: flex; flex-direction: column;
  gap: var(--v-height-4);
  padding: var(--v-height-20) var(--v-width-24);
  border-left: 1px solid var(--v-secondary-border);
}
.cell:first-child { border-left: none; }
@media (max-width: 720px) { .bar { grid-template-columns: repeat(2, 1fr); } }
```

---

## 8. One-off branded accent zone (without polluting the global accent)

Sometimes a single page (landing, marketing, illustration) wants a distinct accent that shouldn't bleed into the rest of the app. Don't override `--v-accent-*` globally — scope a parallel tier under a class.

```css
/* globals.css */
.landing {
  /* Landing-page-only example accent. Used by illustrations and example
     surfaces; never read by core Vireya components. */
  --v-accent-example-fill:       hsl(150 80% 38%);
  --v-accent-example-foreground: hsl(0 0% 98%);
  --v-accent-example-border:     hsl(150 50% 24%);
}
```

```tsx
<main className="landing">
  {/* sections inside can read var(--v-accent-example-fill, var(--v-accent-fill, var(--v-primary-fill))) */}
</main>
```

**Why a custom prefix (`--v-accent-example-*`) and not `--v-accent-*` directly:**
- Scoped to the landing class only — the rest of the app keeps whatever accent is set globally (or no accent at all).
- Doesn't break the cross-app `accent → primary` fallback contract.
- Easy to grep and remove later when the section hierarchy changes.

---

## 9. RSC + namespace block exports — when to mark `"use client"`

Vireya blocks expose a namespace object (`PricingTiers.Root`, `ContactSplit.Root`, `FAQTwoColumn.Root`). When that namespace is **defined inside a `"use client"` module** (because the block uses `createContext` or browser APIs), React's RSC layer rewrites the namespace into a client reference. Property access on a client reference from a Server Component returns `undefined` — so `PricingTiers.Root` becomes `undefined` and you get **"Element type is invalid... got: undefined"** at render.

**Fix:** mark the consumer section as `"use client"`. This puts both the consumer and the block's chunk in the same client boundary; namespace property access works normally.

```tsx
"use client";  // <- required, even for sections without state of their own
import { PricingTiers } from "@vireya/blocks/pricing/tiers";

export const PricingSection = () => (
  <PricingTiers.Root>...</PricingTiers.Root>
);
```

Rule of thumb: if the block uses `createContext` internally, your consumer section needs `"use client"`. When in doubt, just add it — the cost is one extra client component, the benefit is no mysterious render crash.

---

## 10. Catalog rail (illustration-first preview cards)

A rail-style preview of a category (blocks family, chart family, etc.). Illustration on top fills the visible area; meta below stays compact.

```tsx
<Link href={`/catalog/${family}`} className={styles.familyCard}>
  <div className={styles.familyVisual} aria-hidden>
    <div className={styles.familyIllustration}>
      <FamilyIcon />  {/* large SVG illustration, not a small icon */}
    </div>
  </div>
  <div className={styles.familyMeta}>
    <span className={styles.familyName}>{title}</span>
    <span className={styles.familyCount}>{count} variants</span>
  </div>
</Link>
```

```css
.familyCard {
  display: flex; flex-direction: column;
  border: 1px solid var(--v-secondary-border);
  border-radius: var(--v-radius-12);
  background: var(--v-background-fill);
  overflow: hidden; text-decoration: none; color: inherit;
  transition: border-color var(--v-motion-duration) var(--v-motion-timing),
              transform     var(--v-motion-duration) var(--v-motion-timing);
}
.familyCard:hover {
  border-color: var(--v-secondary-active);
  transform: translateY(-2px);
}
.familyVisual {
  aspect-ratio: 200 / 120;          /* match the SVG viewBox */
  background: var(--v-background-fill);
  border-bottom: 1px solid var(--v-secondary-border);
  display: flex; align-items: flex-end; justify-content: center;
  overflow: hidden;
}
.familyIllustration {
  width: 100%; aspect-ratio: 200 / 120;
  /* Fades the illustration into the bottom edge — feels less "cropped". */
  mask-image: linear-gradient(to top, transparent 0%, black 35%);
  -webkit-mask-image: linear-gradient(to top, transparent 0%, black 35%);
}
.familyIllustration > svg { display: block; width: 100%; height: 100%; }
.familyMeta {
  display: flex; align-items: center; justify-content: space-between;
  padding: var(--v-height-12) var(--v-width-14);
}
```

**Key:** the visual `aspect-ratio` MUST match the inner SVG's `viewBox` ratio, otherwise you get letterboxing or cropping. If your SVG is `viewBox="0 0 200 120"`, the container is `aspect-ratio: 200 / 120` (or `5 / 3`).

---

## 11. Theme editor / "copy theme JSON" close-out

After an interactive theme demo, give the user a **next step**: copy the current state as code they can paste into `createTheme()`.

```tsx
"use client";
import { Button } from "@vireya/ui/form/button";
import { Check, Copy } from "lucide-react";

const [tokens, setTokens] = useState(initialTokens);
const [copied, setCopied] = useState(false);

const handleCopy = async () => {
  await navigator.clipboard.writeText(JSON.stringify(tokens, null, 2));
  setCopied(true);
  setTimeout(() => setCopied(false), 1600);
};

<Button variant="accent" surface="solid" onClick={handleCopy}>
  {copied ? <Check size={16} /> : <Copy size={16} />}
  {copied ? "Copied!" : "Copy theme JSON"}
</Button>
```

Pattern applies to anything interactive — **always close the loop with an actionable artifact** (copy, download, email me) instead of leaving the user at a dead end.

---

## 12. Anchor IDs — section navigation that just works

Sections that may be linked from elsewhere on the page (or external) get an `id` matching the URL fragment. CTAs use `href="#pricing"` etc.

```tsx
<Section.Root id="pricing">...</Section.Root>
<Section.Root id="faq">...</Section.Root>
<Section.Root id="contact">...</Section.Root>
```

The Section primitive's `scroll-margin-top: var(--v-height-32)` keeps the anchored target from sliding under a sticky header. If your header is taller, set `scroll-margin-top` to the header height + a comfortable gap on `Section.Root`.
