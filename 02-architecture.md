---
title: Architecture
last-updated: 2026-04-02
primary-audience: all
source: consensus (audit/outside-in + audit/inside-out)
---

# Architecture

## Application Type

React single-page application (SPA) built with Vite 5. No server-side rendering. No React Server Components. Deployed as static files.


## Entry Point and Provider Hierarchy

The app boots from `src/main.tsx` and nests providers in this order (outermost to innermost):

```
main.tsx
  -> GoogleOAuthProvider (Google OAuth SDK)
    -> App.tsx
      -> QueryClientProvider (React Query, retry: 1, refetchOnWindowFocus: false)
        -> TooltipProvider (Radix tooltip context)
          -> Toaster + Sonner (dual toast renderers)
            -> AuthProvider (JWT auth, user session)
              -> WorkspaceProvider (active workspace, blocking gate)
                -> BrowserRouter (client routing)
                  -> SidebarStateProvider (sidebar collapse toggle)
                    -> AppErrorBoundary (class component)
                      -> Suspense (lazy loading fallback)
                        -> Routes
```

Key details:
- `WorkspaceProvider` is a **blocking gate** that does not render children until a workspace is resolved (`src/contexts/WorkspaceContext.tsx:1-439`)
- `NovuProviderWrapper` wraps only the `AppLayout` route, meaning notification context is unavailable on public pages
- `AppErrorBoundary` is a class component (required by React error boundary API)
- All 34 page components use `React.lazy()` with a shared `Suspense` fallback

Reference: `src/App.tsx:134-206`, `src/main.tsx:1-12`


## Source Directory Structure

```
src/
  api/                  - HTTP clients, service layer, type contracts
    client.ts           - ApiClient class (fetch-based, interceptors, token refresh)
    services.ts         - 14 service objects in 1400+ lines (maintenance concern)
    hooks.ts            - React Query hooks (600+ lines) with centralized query keys
    prospectingService.ts - Apollo/ICP prospecting
    types/              - TypeScript interfaces organized by domain
    mocks/              - Mock data for VITE_USE_MOCKS=true mode
  components/           - Shared components
    auth/               - Auth UI (login forms, OAuth buttons, ProtectedRoute - unused)
    common/             - Reusable primitives (EmptyState, PageHeader, StatusBadge, etc.)
    credits/            - Credit balance widgets
    folder/             - Folder-level modals (CSV, Sheets, HubSpot, Salesforce, etc.)
    layout/             - App chrome (AppLayout, AppSidebar, TopBar, SidebarContext)
    list/               - List-level components (sync, import, modals)
    onboarding/         - Getting started card
    settings/           - Settings page tabs (9 tab components)
    ui/                 - shadcn/ui primitives (50 components)
    workspace/          - Workspace selector, create modal
  config/               - Static configuration
    connectionProviders.ts - Polytomic adapter provider list
  contexts/             - React contexts
    AuthContext.tsx      - Auth state, login/signup/logout, JWT management (289 lines)
    WorkspaceContext.tsx - Workspace resolution, switching, blocking gate (439 lines)
  data/                 - Static data files
  features/             - Feature modules (domain-scoped, self-contained)
    batch-enrichment/   - Batch enrichment modal, hooks, services
    enrichment/         - Single-row enrichment (parallel to batch)
    grid/               - AG Grid feature (30+ components, most complex module)
    integrations/       - Integration catalog, provider registry, OAuth flows
    live-audiences/     - LinkedIn/Meta audience management
    notifications/      - Novu notification system
    templates/          - Template system (47 templates, mapping, enrichment)
  hooks/                - Global custom hooks (17 files)
  lib/                  - Utilities and services
    socket.ts           - Socket.IO singleton (362 lines)
    permissions.ts      - RBAC role/permission definitions
    utils.ts            - cn() helper
    services/           - Non-API services (user initialization)
  pages/                - Route-level page components (35 pages, all lazy-loaded)
  test/                 - Test setup (configured but no test files)
```

Reference: Directory listing from Inside-Out audit


## Feature Module Pattern

Each feature folder follows a consistent structure:

```
features/{name}/
  index.ts          - Barrel exports
  types.ts          - Domain types
  services.ts       - API calls (feature-scoped)
  stubs.ts          - Mock data (enrichment, batch-enrichment, grid)
  components/       - Feature-specific UI components
  hooks/            - Feature-specific React hooks
  utils/            - Feature-specific utilities
  contexts/         - Feature-level React contexts (templates only)
```

Feature modules in the codebase:
- `grid/` - AG Grid (30+ components, largest module), toolbar, columns, editing, exports
- `templates/` - Template definitions (47 templates), mapping services, field mapping
- `enrichment/` - Single-row enrichment hooks and services
- `batch-enrichment/` - Batch enrichment modal, hooks, services
- `live-audiences/` - Live audience queries, services, platform tabs
- `integrations/` - OAuth polling, API key flow, integration catalog
- `notifications/` - Novu integration, notification bell, popover

> INVESTIGATE: The `enrichment` and `batch-enrichment` features have nearly identical file structures (types.ts, services.ts, stubs.ts, hooks/*). This suggests a refactor that duplicated rather than consolidated.


## Data Flow Pattern

### API Data Flow
```
ApiClient (fetch wrapper with interceptors)
  -> services.* (typed methods on stateless objects)
    -> React Query hooks (caching, invalidation, mutations)
      -> Page components
        -> Feature components
          -> UI primitives
```

### Real-time Data Flow
```
socketService (Socket.IO singleton)
  -> use-*-socket.ts hooks (register/deregister listeners)
    -> React Query invalidation OR local state update
      -> Component re-render
```

### Auth Flow
```
JWT in localStorage
  -> ApiClient.setToken() (in-memory)
    -> Authorization header on all requests
    -> workspace-id header via interceptor
    -> user-id header via interceptor
```

Reference: `src/api/client.ts:34-277`, `src/api/services.ts`, `src/lib/socket.ts`


## Two API Backends

| Service | Variable | Base URL | Vite Proxy | Handles |
|---------|----------|----------|------------|---------|
| list-service | `VITE_DATA_LIST_SERVICE_BASE_URL` | list-service.icustomer.ai/api/v1 | `/api/v1` | Auth, datasets, lists, enrichment, validation, normalization, payments, webhooks, onboarding |
| gateway | `VITE_GATEWAY_SERVICE_BASE_URL` | prod-aloop-api-gateway.icustomer.ai | `/gateway` (path rewrite strips /gateway) | Integrations, OAuth, audience sync, export flows |

Both clients share token state via `apiClient.setToken()` called from AuthContext. Both inject `workspace-id` and `user-id` headers via request interceptors.

Response convention: `{ success: true, data: T, pagination?: {...} }` or `{ success: false, error: string }`

Gateway responses need double unwrap: `response?.data?.data ?? response?.data` (inconsistent across callsites).

Reference: `src/api/client.ts`, `vite.config.ts:8-30`


## Enrichment Pipeline (Full Stack)

```
User clicks Enrich
  -> REST POST /lists/:listId/enrich
    -> Bull Queue (Redis) on backend
      -> Worker picks job
        -> Streams to Python Agent (SSE)
          -> Row-by-row results via SSE events
            -> Socket.IO emits to subscribed FE clients
              -> Two event channels:
                 Grid events (row_enriched, enrichment_progress) -> AG Grid applyTransaction()
                 Notification events (enrichment:started/progress/completed) -> Progress UI
```

The enrichment system uses strict pathway separation:
- **User edits**: onCellValueChanged -> cellEditQueue -> PATCH API (React pathway)
- **Enrichment updates**: socket events -> cellStatusTracker -> applyTransaction() (non-React pathway)

These pathways are structurally isolated. `applyTransaction()` does NOT fire `onCellValueChanged`.

Reference: `src/features/batch-enrichment/`, `src/features/enrichment/`, `src/lib/socket.ts`


## Mock Data Strategy

Toggle: `VITE_USE_MOCKS=true` environment variable.

Mock data lives in `api/mocks/` and feature `stubs.ts` files. The swap pattern: create `services.ts` with same signatures, change imports in hooks, delete stubs.


## Key Architecture Concerns

1. **services.ts monolith** - 1400+ lines, 14 service objects in a single file. Feature-level services (like `live-audiences/services.ts`) demonstrate the better pattern. (`src/api/services.ts`)

2. **Deprecated code still present** - `audiencesService` is fully deprecated (all methods throw errors) but still exported and has React Query hooks wrapping it. `ProtectedRoute.tsx` exists but is unused. `src/pages/Index.tsx` exists but is not routed.

3. **Dual toast systems** - Both Toaster (Radix) and Sonner render in App.tsx. Migration to Sonner-only is planned but incomplete.

4. **No error boundaries per feature** - Single AppErrorBoundary catches all errors. No feature-level error isolation.

5. **Dev proxy to production** - The dev server proxies to production backend services (`list-service.icustomer.ai`, `prod-aloop-api-gateway.icustomer.ai`). Local development hits real production APIs.
