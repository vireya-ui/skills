# Vireya skills for Claude Code

Claude Code skill for the [**Vireya Design System**](https://github.com/vireya-ui/vireya.mapree.dev) — a token-first React design system with 67 components, 26 pre-composed blocks, theme provider, and motion presets.

Install once and Claude Code becomes an expert at building UIs with `@vireya/core`, `@vireya/ui`, `@vireya/next`, and `@vireya/blocks` — correct subpath imports, `--v-*` token compliance, `forwardRef` + `asChild` + CSS Module conventions, Next.js setup with `ThemeProvider` + `AppProvider`, and the design decision tree (which color tier, when to use `Dialog` vs `Sheet`, spacing rhythm, elevation hierarchy, etc.).

> **This repo is auto-published** from the upstream monorepo — do not open PRs here. See [Source of truth](#source-of-truth) below.

---

## What's inside

```
vireya/
├── SKILL.md          # always loaded when the skill triggers
└── references/
    ├── design.md     # decision guide — color hierarchy, spacing, modals, anti-patterns
    ├── tokens.md     # full --v-* catalog (scales, composite tokens, color sub-tokens)
    ├── setup.md      # Next.js boilerplate (layout.tsx + AppProvider + globals.css)
    ├── blocks.md     # 26 blocks compound API reference
    └── gotchas.md    # institutional traps (RSC + use client, animation flicker, etc.)
```

`SKILL.md` is loaded into context every time the skill activates. The five `references/*` files are loaded on-demand by the model based on the routing table inside `SKILL.md`.

---

## When the skill activates

Mention any of these in a request and Claude will load the skill automatically:

- "vireya", "@vireya/core", "@vireya/ui", "@vireya/next", "@vireya/blocks"
- "build a landing page with vireya"
- "design with vireya tokens"
- "create a vireya component"
- "should this be a Dialog or a Sheet?" (design-decision queries)

You can also force-invoke with `/vireya` in the Claude Code prompt.

---

## Install

### Manual (works everywhere today)

Clone the repo and copy the `vireya/` folder into your Claude skills directory.

**User-scope (available across every project):**
```bash
git clone https://github.com/vireya-ui/skills.git /tmp/vireya-skills
cp -r /tmp/vireya-skills/vireya ~/.claude/skills/
```

**Project-scope (only in one project):**
```bash
git clone https://github.com/vireya-ui/skills.git /tmp/vireya-skills
cp -r /tmp/vireya-skills/vireya .claude/skills/
```

**Symlink (recommended if you want to track upstream updates):**
```bash
git clone https://github.com/vireya-ui/skills.git ~/code/vireya-skills
ln -s ~/code/vireya-skills/vireya ~/.claude/skills/vireya
# pull updates anytime:
cd ~/code/vireya-skills && git pull
```

### Via skills.sh (when supported)

```bash
npx skills add vireya-ui/skills
```

> Check the [skills.sh](https://skills.sh/) entry once published.

---

## Verify install

Open Claude Code in any project and ask:

> Configure a new Next.js app with Vireya from scratch.

Expected behavior — Claude should:
- Import `@vireya/next/styles` first in `app/layout.tsx`
- Wrap with `<ThemeProvider>` and a client `<AppProvider>` that injects `locale` + `icons` map
- Define `--v-page-padding` and `--v-page-max-width` in `globals.css`
- Use per-component subpath imports (`@vireya/ui/form/button`, never `@vireya/ui`)
- Use `var(--v-*)` tokens everywhere — no hardcoded `px`/`#hex`

If any of those are missing, the skill probably isn't loaded. Check `~/.claude/skills/vireya/SKILL.md` exists and has the YAML frontmatter intact.

---

## What the skill teaches

**Non-negotiable rules** (always in context):
- Every CSS visual property → `--v-*` token. Hardcoded `px`/`#hex`/`rgb()` is a bug.
- `primary` (neutral-strong) ≠ `accent` (brand-vivid). Always chain `var(--v-accent-*, var(--v-primary-*))`.
- React 19.2.1 pinned everywhere — never bump in isolation.
- Per-component subpath imports only. Never barrel.
- `:focus-visible` (never `:focus`). Disabled = `opacity: 0.5; cursor: not-allowed`.

**Catalogs** (flat lists with import paths, no prop tables — open the source for those):
- 67 components across 8 groups (form, data, feedback, overlay, navigation, layout, typography, common)
- 26 blocks across 7 families (navbar, hero, feature, pricing, cta, faq, testimonials, contact, footer + eyebrow utility)

**Design decisions** (in `references/design.md`):
- Color hierarchy and `primary` vs `accent` policy
- Typography rhythm (size > weight > color)
- Spacing ladder (when `--v-height-12` vs `-24` vs `-32`)
- Radius hierarchy (children take smaller radius than parent)
- Elevation philosophy (low-elevation; borders + tone over shadows)
- Modal taxonomy (Dialog vs Sheet vs Drawer vs Popover vs Tooltip)
- Composition rules (slots over configuration, compound API split, `asChild`)
- Anti-patterns gallery

**Setup boilerplate** (in `references/setup.md`):
- Copy-pasteable `app/layout.tsx`, `app/layout.client.tsx`, `globals.css`
- Required `AppProvider` icon map (14 lucide-react names) — without it, components like `DateField`, `Calendar`, `Select` indicator break silently
- `createTheme` usage, `useTheme` hook, motion presets

---

## Source of truth

This repo is **auto-published** by a GitHub Action in the Vireya monorepo. The canonical version of every file here lives at:

[`vireya-ui/vireya.mapree.dev/skills/vireya/`](https://github.com/vireya-ui/vireya.mapree.dev/tree/master/skills/vireya) (skill content)
[`vireya-ui/vireya.mapree.dev/.github/distribution/skills/`](https://github.com/vireya-ui/vireya.mapree.dev/tree/master/.github/distribution/skills) (this README)

Every push to `master` in the monorepo that touches those paths triggers a sync to this repo. Each commit message here references the upstream source SHA.

**Do not edit files here directly** — open a PR against the monorepo instead.

---

## Versioning

This skill tracks the `@vireya/*` packages on npm. When the design system ships a new component, block, or token, the skill is updated to match. Pin a specific commit if you want a stable snapshot:

```bash
cd ~/code/vireya-skills && git checkout <commit-sha>
```

Compatible with **`@vireya/*` 0.0.0+**. The skill doesn't depend on a specific version — it teaches the canonical patterns documented in [`DESIGN.md`](https://github.com/vireya-ui/vireya.mapree.dev/blob/master/DESIGN.md) and [`CLAUDE.md`](https://github.com/vireya-ui/vireya.mapree.dev/blob/master/CLAUDE.md). If those rules change, this skill changes too.

---

## Mobile / native?

This skill is **web-focused** (`@vireya/ui` + `@vireya/blocks` + `@vireya/next` for Next.js). A separate mobile skill covering the WebView ↔ native bridge and platform integration patterns is planned.

---

## License

MIT. See the parent design system at [vireya-ui/vireya.mapree.dev](https://github.com/vireya-ui/vireya.mapree.dev) for full project info.
