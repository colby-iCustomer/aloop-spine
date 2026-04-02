---
title: Tech Stack
last-updated: 2026-04-02
primary-audience: all
source: consensus (audit/outside-in + audit/inside-out)
---

# Tech Stack

## Frontend Core

| Technology | Version | Purpose | Reference |
|-----------|---------|---------|-----------|
| React | ^18.3.1 | UI framework | `package.json` |
| React DOM | ^18.3.1 | DOM renderer | `package.json` |
| TypeScript | ^5.8.3 | Type system | `package.json` |
| Vite | ^5.4.19 | Build tool and dev server | `vite.config.ts` |
| React Router DOM | ^6.30.1 | Client-side routing (all routes lazy-loaded) | `src/App.tsx` |


## UI Component Layer

| Technology | Version | Purpose | Reference |
|-----------|---------|---------|-----------|
| Tailwind CSS | ^3.4.17 | Utility-first CSS via HSL token system | `tailwind.config.ts` |
| shadcn/ui | (no version - copy-paste) | Component library (50 primitives in `components/ui/`) | `components.json` |
| Radix UI | Various ^1.x-^2.x | Primitives underlying shadcn (28 packages) | `package.json` |
| Lucide React | ^0.462.0 | Icon library | `package.json` |
| class-variance-authority | ^0.7.1 | Variant-based component styling | `package.json` |
| clsx | ^2.1.1 | Conditional classname joining | `package.json` |
| tailwind-merge | ^2.6.0 | Tailwind class conflict resolution | `package.json` |
| Framer Motion | ^12.34.4 | Animation library | `package.json` |
| cmdk | ^1.1.1 | Command palette (shadcn command) | `package.json` |
| vaul | ^0.9.9 | Drawer component | `package.json` |
| embla-carousel-react | ^8.6.0 | Carousel component | `package.json` |
| react-resizable-panels | ^2.1.9 | Resizable panel layouts | `package.json` |


## Data Grid

| Technology | Version | Purpose | Reference |
|-----------|---------|---------|-----------|
| ag-grid-community | ^35.1.0 | AG Grid core | `package.json` |
| ag-grid-enterprise | ^35.1.0 | Enterprise features (requires license key) | `package.json` |
| ag-grid-react | ^35.1.0 | React wrapper | `package.json` |

AG Grid uses the v35 JS theming API (`themeQuartz.withParams()`) - NOT the legacy CSS approach. Theme configuration is in `src/features/grid/theme.ts`.


## State Management and Data Fetching

| Technology | Version | Purpose | Reference |
|-----------|---------|---------|-----------|
| @tanstack/react-query | ^5.83.0 | Server state management, caching, mutations | `src/api/hooks.ts` |
| socket.io-client | ^4.8.1 | WebSocket for real-time enrichment updates | `src/lib/socket.ts` |
| React Context | (built-in) | Auth state, workspace state, sidebar state | `src/contexts/` |
| localStorage | (built-in) | JWT token, user ID, workspace persistence | `src/contexts/AuthContext.tsx` |
| sessionStorage | (built-in) | Pending template state for cross-page flows | Templates feature |

No Redux, Zustand, Jotai, or other external state libraries.


## Authentication

| Technology | Version | Purpose | Reference |
|-----------|---------|---------|-----------|
| @react-oauth/google | ^0.13.4 | Google Sign-In button and OAuth provider | `src/components/auth/GoogleOAuthButton.tsx` |
| jwt-decode | ^4.0.0 | Decode Google JWT for email domain validation | `src/contexts/AuthContext.tsx` |

Microsoft OAuth is also supported via a custom flow (`/auth/microsoft/complete` route).


## Notifications

| Technology | Version | Purpose | Reference |
|-----------|---------|---------|-----------|
| @novu/react | ^3.10.1 | In-app notification SDK (headless hooks, custom UI) | `src/features/notifications/` |
| Sonner | ^1.7.4 | Toast notifications (preferred) | `package.json` |
| @radix-ui/react-toast | ^1.2.14 | Toast notifications (legacy, being phased out) | `src/components/ui/toast.tsx` |

Two toast systems run in parallel. Sonner is preferred for new code; Radix toast is legacy.


## Rich Text

| Technology | Version | Purpose | Reference |
|-----------|---------|---------|-----------|
| @tiptap/react | ^3.20.1 | Rich text editor framework | `package.json` |
| @tiptap/starter-kit | ^3.20.1 | TipTap base extensions | `package.json` |
| @tiptap/extension-mention | ^3.20.1 | @mention support in editor | `package.json` |
| @tiptap/extension-placeholder | ^3.20.1 | Placeholder text in editor | `package.json` |
| @tiptap/pm | ^3.20.1 | ProseMirror core for TipTap | `package.json` |
| tippy.js | ^6.3.7 | Tooltip/popover for TipTap mentions | `package.json` |

TipTap is used specifically for the column formula/prompt editor in the grid feature (`src/features/grid/components/TiptapColumnEditor.tsx`).


## Visualization

| Technology | Version | Purpose | Reference |
|-----------|---------|---------|-----------|
| Recharts | ^2.15.4 | Charts for analytics pages | `package.json` |
| react-markdown | ^10.1.0 | Markdown rendering | `package.json` |
| remark-gfm | ^4.0.1 | GitHub Flavored Markdown support | `package.json` |


## Forms and Validation

| Technology | Version | Purpose | Reference |
|-----------|---------|---------|-----------|
| react-hook-form | ^7.61.1 | Form state management | `package.json` |
| @hookform/resolvers | ^3.10.0 | Form validation resolvers | `package.json` |
| zod | ^3.25.76 | Schema validation | `package.json` |


## Utilities

| Technology | Version | Purpose | Reference |
|-----------|---------|---------|-----------|
| date-fns | ^3.6.0 | Date utilities | `package.json` |
| next-themes | ^0.3.0 | Dark mode toggle (class-based) | `tailwind.config.ts` |
| react-day-picker | ^8.10.1 | Date picker component | `package.json` |
| react-loader-spinner | ^8.0.2 | Loading spinners (InfinitySpin) | `package.json` |
| input-otp | ^1.4.2 | OTP input component | `package.json` |

> INVESTIGATE: `next-themes` is unusual for a non-Next.js app. It works but couples to the Next.js ecosystem unnecessarily.


## Dev Dependencies

| Technology | Version | Purpose | Reference |
|-----------|---------|---------|-----------|
| @vitejs/plugin-react-swc | ^3.11.0 | SWC-based React plugin (faster than Babel) | `vite.config.ts` |
| tailwindcss | ^3.4.17 | Utility-first CSS framework | `package.json` |
| @tailwindcss/typography | ^0.5.16 | Typography plugin | `package.json` |
| autoprefixer | ^10.4.21 | CSS vendor prefixing | `package.json` |
| postcss | ^8.5.6 | CSS processing | `package.json` |
| eslint | ^9.32.0 | Linter (flat config format) | `eslint.config.js` |
| typescript-eslint | ^8.38.0 | TypeScript ESLint parser | `package.json` |
| vitest | ^3.2.4 | Test framework (configured, no tests written) | `vitest.config.ts` |
| @testing-library/react | ^16.0.0 | React testing utilities | `package.json` |
| @testing-library/jest-dom | ^6.0.0 | Jest-DOM matchers | `package.json` |
| jsdom | ^20.0.3 | DOM simulation for tests | `package.json` |
| lovable-tagger | ^1.1.13 | Component tagging (Lovable platform, dev only) | `vite.config.ts:34` |


## Backend Services (not in this repo)

| Technology | Purpose | Source |
|-----------|---------|--------|
| Node.js / Express | Backend API server | Docs only |
| PostgreSQL | Database (multi-schema, multi-tenant) | Docs only |
| Drizzle ORM | Database ORM | Docs only |
| Redis / Bull | Job queues for enrichment | Docs only |
| Socket.IO (server) | Real-time event broadcasting | Docs only |
| Stripe | Payment processing | Docs only |
| Novu (server) | Notification workflows | Docs only |
| ZeroBounce | Email validation | Docs only |
| Polytomic | Salesforce adapter | Code (type references only) |


## External Services

| Service | Base URL | Purpose |
|---------|----------|---------|
| list-service | list-service.icustomer.ai/api/v1 | Core data operations |
| gateway | prod-aloop-api-gateway.icustomer.ai | Integrations, OAuth, audience sync |
| n8n | n8n.icustomer.ai/webhook/field-mapping-agent | AI field mapping |


## Build and Development

| Tool | Purpose | Reference |
|------|---------|-----------|
| Bun | Package manager and build runner | `package.json` scripts |
| Lovable | UI iteration platform (app synced back to Lovable) | `lovable-tagger` plugin |
| SWC | React compilation (faster than Babel) | `@vitejs/plugin-react-swc` |


## What Is NOT in This Stack

The following were in the old repo but are confirmed absent from the current codebase:

| Technology | Old Purpose | Status |
|-----------|------------|--------|
| Jotai | State management | Replaced by React Context + React Query |
| Ant Design (antd) | UI components | Replaced by shadcn/ui |
| Axios / AuthClient | HTTP client | Replaced by single fetch-based ApiClient |
| AG Grid v32 | Data grid | Upgraded to v35 with JS theming API |
| Supabase | Direct DB queries from hooks | Fully removed |
| highlight.run | Error tracking | **Not replaced** |
| PostHog | Analytics tracking | **Not replaced** |
| Intercom | Support widget | **Not replaced** |
| three.js / @react-three | 3D visuals | Documented as out of scope |
| ReactFlow | Flow diagrams | Dropped |
| dnd-kit | Drag and drop | Dropped |
| react-querybuilder | Filter rules UI | Dropped |


## TypeScript Configuration Concerns

The TypeScript configuration is notably loose (`tsconfig.app.json`):
- `strictNullChecks: false` - potential null reference bugs hidden at compile time
- `noImplicitAny: false` - any types allowed implicitly
- `noUnusedParameters: false` - dead parameters not flagged
- `noUnusedLocals: false` - dead variables not flagged
- `skipLibCheck: true` - library type errors skipped
- `@typescript-eslint/no-unused-vars: off` - dead imports/variables not caught by linter
