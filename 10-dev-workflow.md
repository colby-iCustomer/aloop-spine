---
title: Dev Workflow
last-updated: 2026-04-02
primary-audience: all
source: consensus (audit/outside-in + audit/inside-out)
---

# Dev Workflow

## Package Scripts

| Script | Command | Purpose |
|--------|---------|---------|
| `bun run dev` | `vite` | Start dev server (port 8080, host ::) |
| `bun run build` | `vite build` | Production build with TypeScript checking |
| `bun run build:dev` | `vite build --mode development` | Dev-mode build (preserves dev tools, console.log, dev-bypass-token) |
| `bun run lint` | `eslint .` | Run ESLint |
| `bun run preview` | `vite preview` | Preview production build locally |
| `bun run test` | `vitest run` | Run tests once |
| `bun run test:watch` | `vitest` | Run tests in watch mode |

Reference: `package.json:6-14`


## Dev Server Configuration

| Setting | Value | Reference |
|---------|-------|-----------|
| Host | `::` (all interfaces, IPv6) | `vite.config.ts` |
| Port | 8080 | `vite.config.ts` |
| HMR overlay | disabled (`overlay: false`) | `vite.config.ts` |
| React plugin | `@vitejs/plugin-react-swc` (SWC, not Babel) | `vite.config.ts` |
| Dev plugin | `lovable-tagger` componentTagger() (dev only) | `vite.config.ts:34` |
| Path alias | `@` -> `./src` | `vite.config.ts` |

### Proxy Configuration

| Path | Target | Notes |
|------|--------|-------|
| `/api/v1` | `https://list-service.icustomer.ai` | API backend |
| `/socket.io` | `https://list-service.icustomer.ai` | WebSocket (ws: true) |
| `/gateway` | `https://prod-aloop-api-gateway.icustomer.ai` | Path rewrite strips /gateway |

> INVESTIGATE: The dev server proxies to **production** backend services. Local development hits real production APIs. There is no local or staging backend option documented.


## Environment Files

| File | Purpose |
|------|---------|
| `.env` | Shared defaults |
| `.env.development` | Dev-specific overrides |
| `.env.production` | Production values |

### Known Environment Variables

| Variable | Purpose |
|----------|---------|
| `VITE_GOOGLE_CLIENT_ID` | Google OAuth client ID |
| `VITE_DATA_LIST_SERVICE_BASE_URL` | API base URL |
| `VITE_GATEWAY_SERVICE_BASE_URL` | Gateway base URL |
| `VITE_SOCKET_URL` | Socket.IO server URL (optional) |
| `VITE_NOVU_APPLICATION_IDENTIFIER` | Novu notification app ID |
| `VITE_USE_MOCKS` | Enable mock data (true/false) |


## ESLint Configuration (`eslint.config.js`)

- Format: Flat config (ESLint v9)
- Extends: `@eslint/js` recommended + `typescript-eslint` recommended
- Plugins: `react-hooks`, `react-refresh`
- Files: `**/*.{ts,tsx}`
- Ignores: `dist/`

Notable rule settings:
- `react-hooks` recommended rules: **enabled**
- `react-refresh/only-export-components`: warn (allows constant exports)
- `@typescript-eslint/no-unused-vars`: **off** (disabled, dead imports/variables accumulate)

Reference: `eslint.config.js:1-26`


## TypeScript Configuration

### tsconfig.app.json

| Setting | Value | Impact |
|---------|-------|--------|
| `noImplicitAny` | false | Any types allowed implicitly |
| `strictNullChecks` | false | Null reference bugs hidden |
| `noUnusedParameters` | false | Dead parameters not flagged |
| `noUnusedLocals` | false | Dead variables not flagged |
| `skipLibCheck` | true | Library type errors skipped |
| `allowJs` | true | JavaScript files allowed |

**Path alias:** `@/*` -> `./src/*`

Reference: `tsconfig.json:1-16`, `tsconfig.app.json`


## Test Configuration (`vitest.config.ts`)

- Runner: Vitest
- Environment: jsdom
- Globals: true (test/expect/describe available without imports)
- Setup file: `./src/test/setup.ts`
- Include pattern: `src/**/*.{test,spec}.{ts,tsx}`
- Plugin: `@vitejs/plugin-react-swc`

**No test files exist.** Vitest is configured but zero `*.test.ts` or `*.spec.ts` files have been written.

Reference: `vitest.config.ts:1-16`


## Git Conventions

### Branching Strategy

| Branch Pattern | Purpose |
|---------------|---------|
| `main` | Stable + Lovable commits (Lovable always commits to main) |
| `dev` | Development integration |
| `staging` | Staging environment |
| `production` | Production release |
| `feature/{name}` | Feature work |
| `fix/{name}` | Bug fixes |
| `spike/{name}` | Exploration/research |
| `docs/{name}` | Documentation |
| `colby-{name}` | Personal branches |
| `worktree-agent-{name}` | Automated agent worktrees |
| `lovable` | Lovable platform branch |

**Workflow:** Create branch from latest main -> work -> squash merge to main -> delete feature branch.

### Commit Format

```
[category] description
```

Categories: `feat`, `fix`, `refactor`, `design`, `docs`

Examples:
- `[feat] Add Supabase Auth with Google OAuth`
- `[fix] StatusBadge - use success token instead of green-600`

### Pre-commit Checks (Manual)

- Run `bun run build` to verify no TypeScript errors
- Check that no hardcoded hex colors or raw Tailwind color literals were introduced
- No automated enforcement (no husky, lint-staged, or pre-commit hooks)


## CI/CD

**No CI/CD pipeline exists.** There is no `.github/` directory, no GitHub Actions workflows, no automated testing on push, no automated build verification, no automated linting.

Deployment is handled externally. A `_redirects` file in `public/` suggests Vercel or Netlify-style SPA deployment with all paths redirecting to index.html.

Three deployment targets implied by remote branches: dev, staging, production.


## Lovable Compatibility

The repo is iterated locally and synced back to the Lovable platform:
- Avoid non-standard build plugins or Vite config changes
- Keep component structure flat and readable (Lovable's editor parses JSX)
- shadcn/ui is the preferred primitive layer, no other UI libraries
- `lovable-tagger` plugin tags components in dev mode for Lovable's visual editor
- `tmp/` folder for scratch files during migration (not committed)


## Planning System (Superpowers)

**Active specs:** `docs/superpowers/specs/` - brainstorming outputs and design specs
**Active plans:** `docs/superpowers/plans/` - implementation step plans
**Archive:** `docs/archive/plans/` - pre-superpowers era plans

Completed plans automatically generate a completion entry in `docs/current-repo/decisions/` with decisions, deviations, design fixes, and lessons learned.


## Working Docs System

Living reference documents in `docs/current-repo/working-docs/`:

| Document | Purpose |
|----------|---------|
| API Endpoint Map | Every endpoint with validation status |
| Styling Decisions | Visual/typographic decisions with alternatives tested |
| Troubleshooting | Known issues with index and per-issue files |

Rules: check before touching a domain, update when making decisions, update when reverting.


## Port Feature Pipeline

Multi-agent pipeline for porting features from old app. Agents: gatherer, frontend, reviewer, wiring, spot-check.

12 features ported through the pipeline:
1. Heading bar actions drawer
2. LinkedIn Ads
3. Integration infrastructure
4. Wire platform connections
5. Export action flows
6. Template mapping
7. Notifications
8. Live audiences
9. Template system
10. Template card
11. Template CTA wiring
12. Getting started

Each feature port produces a log in `docs/port-feature-pipeline/logs/{NN}-{feature}/`.


## Decision Journal

`docs/current-repo/decisions/` contains 25+ post-completion entries covering architectural decisions, API wiring notes, and plan deviations. Each entry linked to its source plan.


## Static Assets (`public/`)

| File | Purpose |
|------|---------|
| `_redirects` | Deployment SPA routing rules |
| `aloop-docs-banner.avif` | Documentation banner image |
| `favicon.ico` | Browser favicon |
| `loopFavicon.png` | Collapsed sidebar icon |
| `new-aloop-logo.png` | Full logo (expanded sidebar) |
| `new_Contact.csv` | Sample CSV file for template sample data feature |
| `placeholder.svg` | Placeholder image |
| `robots.txt` | Search engine directives |


## What Is Missing

| Concern | Status |
|---------|--------|
| **Automated testing** | Vitest configured, zero tests written |
| **CI/CD** | No GitHub Actions, no automated checks |
| **Pre-commit hooks** | No husky, lint-staged, or similar |
| **Error tracking** | highlight.run was in old repo, not replaced |
| **Product analytics** | PostHog was in old repo, not replaced |
| **Monitoring/observability** | No logging, error tracking, or performance monitoring |
| **Security practices** | No dependency scanning, no secret management |
| **Code review process** | Only "squash merge after code review" documented |
| **Deployment documentation** | No environment promotion or rollback procedures |
| **Console.log control** | No lint rule prevents console.log in production code |
