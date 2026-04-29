# Next.js setup with Vireya

Canonical reference: `apps/www/src/app/layout.tsx` + `layout.client.tsx` + `src/styles/globals.css`. Copy-pasteable templates below.

---

## Install

```bash
pnpm add @vireya/core @vireya/ui @vireya/next react@19.2.1 react-dom@19.2.1 next
pnpm add lucide-react   # required: AppProvider needs icons
```

Optional peers — install only when you use the corresponding component:
- `@tanstack/react-table` — for `@vireya/ui/data/dataTable`
- `recharts` — for `@vireya/ui/data/chart`
- `embla-carousel-react` — for `@vireya/ui/data/carousel`
- `react-day-picker` + `date-fns` — for `@vireya/ui/data/calendar` and `@vireya/ui/form/dateField`
- `cmdk` — for `@vireya/ui/data/command`
- `vaul` — for `@vireya/ui/overlay/drawer`
- `react-resizable-panels` — for `@vireya/ui/layout/resizable`
- `@headless-tree/core` — for `@vireya/ui/navigation/treeView`
- `react-hook-form` — if you use the form helpers

If you also use blocks: `pnpm add @vireya/blocks`.

---

## File 1 — `app/layout.tsx` (server component)

```tsx
import { ThemeProvider } from "@vireya/next";
import "@vireya/next/styles";                      // 1st: base tokens + light/dark themes
import "@vireya/next/motion/wipe-reveal";          // optional: View Transitions preset
import "../styles/globals.css";                     // last: app overrides
import { createTheme } from "@vireya/core";
import { AppProvider } from "./layout.client";
import type { PropsWithChildren } from "react";

const lightBrand = createTheme({
  background: "hsl(0 0% 98%)",
  secondary:  "hsl(0 0% 92%)",
  primary:    "hsl(0 0% 8%)",
  accent:     "hsl(150 80% 38%)",   // optional brand color
});

const darkBrand = createTheme({
  background: "hsl(0 0% 0%)",
  secondary:  "hsl(0 0% 14%)",
  primary:    "hsl(0 0% 96%)",
  accent:     "hsl(150 80% 45%)",
});

export default function RootLayout({ children }: PropsWithChildren) {
  return (
    <html suppressHydrationWarning>
      <head />
      <body>
        <ThemeProvider
          customThemes={{ brand: lightBrand, brandDark: darkBrand }}
          // optional:
          // vars={{ fontFamily: { sans: "Geist, sans-serif" } }}
          // defaultTheme="brand"
          // forcedTheme="brand"     // lock theme regardless of cookie
          // cookieName="theme"
        >
          <AppProvider>{children}</AppProvider>
        </ThemeProvider>
      </body>
    </html>
  );
}
```

**Hard rules:**
- `@vireya/next/styles` import must be **first** so app overrides win.
- `<html suppressHydrationWarning>` is **required** — the theme bootstrap script runs before React hydrates and would otherwise trigger a mismatch.
- Motion preset import is a **CSS side-effect** — there's no JS export. Pick at most one of `wipe-reveal`, `mask-reveal`, `circle-reveal`, `angled-reveal`.

---

## File 2 — `app/layout.client.tsx` (client wrapper)

```tsx
"use client";

import { AppProvider as UIAppProvider } from "@vireya/ui";
import ptBR from "@vireya/ui/locales/ptBR";          // or enUS
import {
  Brush, Calendar, Check, ChevronDown, ChevronLeft, ChevronRight, ChevronUp,
  Circle, X as Close, CloudUpload, Copy, Eye, EyeOff, Search,
} from "lucide-react";
import type { PropsWithChildren } from "react";

export const AppProvider = ({ children }: PropsWithChildren) => (
  <UIAppProvider
    locale={ptBR}
    icons={{
      Brush, Calendar, Check, ChevronDown, ChevronLeft, ChevronRight, ChevronUp,
      Circle, Close, CloudUpload, Copy, Eye, EyeOff, Search,
    }}
  >
    {children}
  </UIAppProvider>
);
```

**Why every name is required:** Vireya components reference these icons by canonical name. Missing any one of them silently breaks the corresponding component:

| Icon | Used by |
|---|---|
| `Calendar` | `Calendar`, `DateField` trigger |
| `Check` | `Checkbox` indicator, `DropdownMenu`/`ContextMenu` checkbox items, `Command` selection, `CopyField` after copy |
| `ChevronDown`/`Up`/`Left`/`Right` | `Select` indicator, `Calendar` nav, `Sidebar`/`TreeView` expanders, scroll arrows |
| `Circle` | `RadioGroup` indicator, `DropdownMenu`/`ContextMenu` radio items |
| `Close` (`X`) | `Toast` close, `Dialog`/`Drawer`/`Sheet` close buttons |
| `CloudUpload` | `UploadArea` default icon |
| `Copy` | `CopyField` |
| `Eye`/`EyeOff` | `PasswordField` toggle |
| `Search` | `Command` input, `DataTable` search |
| `Brush` | theme/style indicators in showcases |

Substitute any icon library that exports React components with the same shape (e.g. `phosphor-react`) — the contract is just a `React.ComponentType`.

`AppProvider` is the **one allowed root import** from `@vireya/ui`. Everything else is a subpath.

---

## File 3 — `styles/globals.css`

```css
* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

body {
  --v-page-max-width: 1520px;
  --v-page-padding: var(--v-width-16);
  /* or: clamp(16px, 5vw, 32px) for fluid gutters */
}
```

`--v-page-max-width` and `--v-page-padding` are **not** shipped by `@vireya/core` but `@vireya/blocks` assumes they exist. Define them here.

---

## Theme switching (client)

```tsx
"use client";
import { useTheme } from "@vireya/next/client";

export function ThemeToggle() {
  const { theme, setTheme, themes } = useTheme();

  return (
    <button
      onClick={(e) =>
        // Pass coords to enable the View Transitions circle-reveal animation
        // (only effective if a motion preset CSS is imported in layout.tsx)
        setTheme(theme === "brandDark" ? "brand" : "brandDark", {
          x: e.clientX,
          y: e.clientY,
        })
      }
    >
      {theme}
    </button>
  );
}
```

`useTheme()` returns `{ theme, setTheme, themes, forcedTheme }`. The hook:
- persists the choice via cookie (`cookieName` from `<ThemeProvider>`, default `"theme"`)
- applies `v-theme-{name}` class on `<html>`
- respects `prefers-reduced-motion: reduce` (skips View Transitions animation)
- safe to call from any client component below `<ThemeProvider>`

---

## Custom theme — `createTheme`

```ts
import { createTheme } from "@vireya/core";

const myTheme = createTheme({
  background: "hsl(240 6% 97%)",
  secondary:  "hsl(240 6% 92%)",
  primary:    "hsl(240 8% 13%)",

  // Optional — falls back to defaults if omitted:
  destructive: "hsl(0 84% 60%)",
  link:        "hsl(221 83% 53%)",

  // Optional — without it, --v-accent-* is undefined; consumers must use
  // var(--v-accent-fill, var(--v-primary-fill)) fallbacks (they should already).
  accent: "hsl(150 80% 38%)",

  // Per-subtoken override (anything you don't override is auto-derived):
  // primary: { fill: "hsl(240 8% 13%)", hover: "#4d4d5b" }
});

myTheme.toCss("v");        // → "color-scheme:light;--v-background-fill:#...;..."
myTheme.raw;               // → { background: { fill, foreground, ... }, ... }
myTheme.colorScheme;       // → "light" | "dark" (auto from background luminance)
myTheme.extend({ accent: "#ff00aa" });   // returns a new theme
```

Pass to `<ThemeProvider customThemes={{ myTheme }}>`. The default `light` and `dark` themes are always present; you don't need to redefine them.

---

## Motion presets (route + theme transitions)

Each preset is a CSS file imported as a side-effect from `@vireya/next/motion/<name>`. Only the `::view-transition-*` pseudos are styled — the actual animation triggers when the browser starts a View Transition (theme switch via `useTheme().setTheme`, or Next.js App Router navigation when you set `viewTransitionName`).

| Preset | Effect | Best for |
|---|---|---|
| `wipe-reveal` | diagonal clip-path wipe (~150ms) | fast page transitions, default pick |
| `circle-reveal` | circular expand from `(--v-transition-start-x, --v-transition-start-y)` | theme toggle from a click point (pass `{ x, y }` to `setTheme`) |
| `mask-reveal` | radial mask reveal using an image (default: a Tenor GIF; override via `--v-gif-mask-url`) | playful brand-driven transitions |
| `angled-reveal` | angled diagonal slide (1s) | stylized hero transitions |

You can import multiple — they target the same pseudo-elements, so the **last one wins** for any given selector. In practice, pick one.

To trigger a route transition with the preset:
```tsx
"use client";
import Link from "next/link";

export const NavLink = (props: React.ComponentProps<typeof Link>) => (
  <Link {...props} style={{ viewTransitionName: "root", ...(props.style ?? {}) }} />
);
```

---

## Verification checklist for a new app

After scaffolding, confirm:

- [ ] `<html suppressHydrationWarning>` present
- [ ] `@vireya/next/styles` imported **first** in layout
- [ ] `<ThemeProvider>` wraps `<AppProvider>` wraps `{children}`
- [ ] `AppProvider` passes both `locale` and the full `icons` map (14 names above)
- [ ] `globals.css` defines `--v-page-padding` and `--v-page-max-width`
- [ ] No hardcoded colors/sizes anywhere — only `var(--v-*)`
- [ ] All component imports are subpaths: `@vireya/ui/<group>/<name>` or `@vireya/blocks/<family>/<name>`
- [ ] `useTheme` is imported from `@vireya/next/client`, not `@vireya/next`
