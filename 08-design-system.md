---
title: Design System
last-updated: 2026-04-02
primary-audience: frontend
source: consensus (audit/outside-in + audit/inside-out)
---

# Design System

## Configuration

- **shadcn/ui**: style=default, base-color=slate, css-variables=true, rsc=false
- **Font**: Inter (Google Fonts, weights: 400, 500, 600, 700)
- **Dark mode**: class-based via `next-themes`, configured in `tailwind.config.ts:4` as `darkMode: ["class"]`
- **Tailwind**: v3.4.17 with typography plugin

Reference: `components.json:1-20`, `tailwind.config.ts:1-94`, `src/index.css:1-617`


## Color System (HSL CSS Variables)

All design values live as CSS custom properties in `:root` (`src/index.css`). Never hardcode hex or HSL values in JSX or component files.

### Semantic Tokens (theme-aware)

| Token | Light Value | Hex Approx | Role |
|-------|------------|------------|------|
| --background | 213 40% 98% | ~#f5f7fb | Page background (blue-tinted) |
| --foreground | 220 30% 12% | ~#171f2b | Default text |
| --card | 0 0% 100% | #ffffff | Card surfaces |
| --card-foreground | 220 30% 12% | ~#171f2b | Card text |
| --popover | 0 0% 100% | #ffffff | Popover backgrounds |
| --popover-foreground | 220 30% 12% | ~#171f2b | Popover text |
| --primary | 213 97% 56% | ~#2085fc | Brand blue |
| --primary-foreground | 0 0% 100% | #ffffff | Text on primary |
| --secondary | 213 50% 93% | Light blue tint | Secondary surfaces |
| --secondary-foreground | 213 70% 32% | Dark blue | Secondary text |
| --muted | 213 40% 92% | Blue-tinted surface | Muted backgrounds |
| --muted-foreground | 213 18% 46% | Gray-blue | Muted text |
| --accent | 213 50% 93% | Same as secondary | Accent surfaces |
| --accent-foreground | 213 70% 32% | Same as secondary | Accent text |
| --destructive | 0 84% 60% | Red | Error/danger |
| --success | 155 45% 37% | Green | Success state |
| --warning | 38 92% 50% | Amber | Warning state |
| --border | 213 35% 87% | Blue-tinted | Default borders |
| --input | 213 35% 87% | Same as border | Input borders |
| --ring | 213 60% 72% | Medium blue | Focus ring (intentionally lightened) |

### Palette Primitives (fixed, not theme-aware)

| Token | HSL Value | Hex Approx | Use |
|-------|-----------|------------|-----|
| --accent-blue | 213 64% 45% | #2966bc | Category/icon accents |
| --accent-green | 142 71% 45% | Tailwind green-500 | Status accents |
| --accent-purple | 271 91% 65% | #a855f7 | Category/icon accents |
| --accent-teal | 172 76% 40% | #14b8a6 | Category/icon accents |

**Naming rule:** Semantic tokens by role (--primary, --success). Palette primitives by color (--accent-blue). Never name palette primitives by role.


### Dark Mode Values

Full dark mode theme in `.dark` class:
- Background: 224 14% 10% (dark gray)
- Card: 224 14% 13% (slightly lighter)
- Primary: 213 97% 62% (brighter blue for dark)
- Border: 224 14% 20% (subtle dark borders)
- All sidebar variables adjusted for dark backgrounds

> INVESTIGATE: `next-themes` is imported for theme toggling but no dark mode toggle button was found in the UI code. Dark mode may not be user-accessible yet.


### Panel-Specific Tokens

| Token | HSL Value | Use |
|-------|-----------|-----|
| --gs-bg | 138 76% 97% | Getting Started panel background |
| --gs-border | 141 54% 84% | Getting Started panel border |
| --gs-item-border | 141 54% 91% | Getting Started step item borders |
| --qa-gradient | linear-gradient(135deg, hsl(213 78% 96%), hsl(228 68% 94%), hsl(245 60% 93%)) | Quick Actions panel gradient |


### Sidebar Tokens

| Token | Light Value | Purpose |
|-------|------------|---------|
| --sidebar-background | 0 0% 100% | White sidebar |
| --sidebar-foreground | 215 28% 20% | Sidebar text |
| --sidebar-primary | 213 97% 56% | Sidebar primary (brand blue) |
| --sidebar-accent | 213 60% 94% | Sidebar hover |
| --sidebar-accent-foreground | 213 80% 38% | Sidebar hover text |
| --sidebar-border | 213 30% 91% | Sidebar dividers |


### Layout Variables

| Token | Value | Purpose |
|-------|-------|---------|
| --topbar-height | 56px | Top navigation bar height |
| --page-max-width | 1800px | Max content width |


## Typography Hierarchy

| Level | Class | Use |
|-------|-------|-----|
| L1 - Page Title | `text-3xl font-bold text-foreground` | PageHeader only, one per page |
| L2 - Card/Section Title | `text-base font-semibold text-foreground` | Card headers, section titles |
| L3 - Item Title | `text-sm font-medium text-foreground` | Card item labels, list names |
| L4 - Description | `text-xs text-muted-foreground` | Help text, subtitles, metadata |

Headings have `tracking-tight` applied globally.

Reference: Tailwind config `fontFamily: { sans: ['Inter', 'system-ui', 'sans-serif'] }`


## Border Radius

Base: `--radius: 0.625rem` (10px). Intentionally tighter than shadcn default. Do not increase.

| Token | Value |
|-------|-------|
| lg | var(--radius) = 10px |
| md | calc(var(--radius) - 2px) = 8px |
| sm | calc(var(--radius) - 4px) = 6px |


## Focus Rings

`--tw-ring-offset-width: 0px` set globally. `--ring` lightened to `213 60% 72%` to reduce visual weight. Do not revert.


## Background Gradient

The body has a subtle fixed gradient background:
```css
background: linear-gradient(145deg, hsl(213, 25%, 98.5%) 0%, hsl(220, 10%, 97.5%) 100%);
background-attachment: fixed;
```
Left side: barely-blue tint. Right side: near-neutral gray. Keep right endpoint low-saturation.


## Card Family System

Four card families with CSS classes in `src/index.css`:

| Family | CSS Class(es) | Use |
|--------|--------------|-----|
| 1 - Action Card | `.card-action-full`, `.card-action-compact` | Navigable/clickable cards |
| 2 - Stat Card | `.card-stat`, `.card-stat-featured` | Metric display (icon + title + number) |
| 3 - Connect Card | `.card-connect` | Integration cards with CTA button |
| 4 - Gallery Card | (inline styles) | Template cards (rounded-3xl, decorative circles) |

Additional card classes: `.card-data`, `.card-info`, `.card-step`

**Card sizing variables:**
| Variable | Value | Purpose |
|----------|-------|---------|
| --card-action-min-w | 230px | Action card minimum width |
| --card-action-compact-min-w | 240px | Compact card min width |
| --card-action-compact-max-w | 370px | Compact card max width |
| --card-data-min-w | 230px | Data card min width |
| --card-data-max-w | 332px | Data card max width |

**Shadow rules:**
- Clickable card: `shadow-sm` at rest + larger shadow on hover
- Non-clickable card: no shadow at rest or hover
- Shadow communicates interactivity

**Border rules:**
- All borders 1px, never 2px
- Full Action Cards on gradient bg: transparent at rest, colored on hover
- Compact Action Cards on white bg: 40% opacity at rest, 100% on hover


## PLG Utility Classes

### Base Classes
| Class | Purpose |
|-------|---------|
| `.plg-card` | Standard card with shadow + hover shadow |
| `.plg-card-lift` | Card with lift-on-hover effect |

### Action Variants
| Class | Purpose |
|-------|---------|
| `.plg-action-blue` | Blue border on hover |
| `.plg-action-purple` | Purple border on hover |
| `.plg-action-teal` | Teal border on hover |
| `.plg-action-active` | Active/selected state |
| `.plg-action-tilt` | Subtle rotation during auto-cycle (not on manual hover) |

### Utility Classes
| Class | Purpose |
|-------|---------|
| `.status-dot` | Tiny inline status indicator dot |
| `.data-label` | Label above form fields |
| `.mono-value` | Monospace display for IDs, API keys, codes |
| `.page-container` | Page content padding (responsive) |
| `.page-constrained` | Max-width constraint |
| `.grid-responsive` | Auto-fill responsive grid |
| `.grid-fixed` | Fixed-width card grid |
| `.hide-scrollbar` | Hidden scrollbar with scroll |
| `.thin-scrollbar` | Visible thin scrollbar |

### Pill Badges
| Class | Data Type |
|-------|-----------|
| `.pill-green` | Record counts |
| `.pill-blue` | Account counts, tags |
| `.pill-amber` | Empty/disconnected states |


## Animation Definitions

### Tailwind keyframes (`tailwind.config.ts`)
- `accordion-down/up`: height transition, 0.2s ease-out

### CSS keyframes (`src/index.css`)
- `modal-in/out`: opacity + translateY(6px), smooth dialog entrance/exit
- `overlay-in/out`: opacity transition for dialog overlays
- `grid-item-enter`: opacity + translateY(8px) + scale(0.98), 0.4s ease-out, staggered via nth-child delays


## AG Grid Theme (`src/features/grid/theme.ts`)

Custom Quartz theme built with `themeQuartz.withParams()`:

| Property | Value |
|----------|-------|
| Font family | Inter |
| Font size (body) | 14px |
| Font size (headers) | 13px |
| Background | #fcfcfc |
| Header background | #f8f9fc |
| Border color | #edeff3 |
| Row height | 42px |
| Cell horizontal padding | 12px |
| Cell vertical padding | 0px |
| Accent color | #0084e7 |
| Row hover | rgba(0, 132, 231, 0.04) |
| Scrollbar thumb | #dee2e6 |
| Scrollbar track | #f8f9fa |

Additional AG Grid CSS overrides in `src/index.css` handle borders, selection, scrollbars, and pinned columns using `!important`.


## Color Usage Rules

1. **Always use tokens.** If a color is not in the token system, add it to `src/index.css` first.
2. **Tailwind usage:** `bg-primary`, `text-success`, `border-border`
3. **CSS variables not in Tailwind:** `bg-[hsl(var(--gs-bg))]` or inline `style`
4. **Per-element accent colors:** Pass CSS variable name as string, resolve at render time
5. **Never:** `bg-[#2966bc]`, `style={{ color: '#f0fdf4' }}`, hardcoded HSL strings


## Style Guide Page

Route: `/dev/style-guide` (dev-only, gated behind `import.meta.env.DEV`)
Tabs: Cards, Typography, Colors and Tokens, Buttons and Pills
Purpose: Single source of truth for all design elements, rendered live with actual tokens.


## Known Violations (from styling decisions doc)

| Location | Violation | Fix |
|----------|-----------|-----|
| TemplatesPage template button | `text-[13px]` | Should be `text-sm` |
| FolderPage source pill | `text-[11px]` | Should be `text-xs` |
| HomePage stat values | `text-blue-600`, `text-green-600` | Should use semantic tokens |


## Known Concerns

1. **index.css is 617 lines** mixing global resets, component classes, AG Grid overrides, and TipTap styles in a single file. Consider splitting.
2. **Two scrollbar strategies**: Global thin scrollbars (4px) vs AG Grid wider scrollbars (via theme params).
3. **AG Grid CSS uses many `!important` overrides** making the cascade fragile.
4. **`.mosaic-grid` suppresses all CSS animations** on children with `!important` to prevent conflicts with Framer Motion.
5. **Some card classes use `@apply`** which compiles away at build time but makes CSS harder to trace at runtime.
