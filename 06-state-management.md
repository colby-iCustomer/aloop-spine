---
title: State Management
last-updated: 2026-04-02
primary-audience: frontend
source: consensus (audit/outside-in + audit/inside-out)
---

# State Management

## Architecture Overview

The app uses a three-layer state model with no external state libraries (no Redux, Zustand, or Jotai):

1. **React Context** - Identity and workspace state (who am I, which workspace)
2. **React Query** - All server/API state (what data do I have)
3. **Socket.IO** - Real-time event-driven state updates (enrichment progress, validation, etc.)


## React Contexts

### AuthContext (`src/contexts/AuthContext.tsx:1-289`)

**State:**
```typescript
{ user: UserProfile | null, isAuthenticated: boolean, isLoading: boolean }
```

**Actions:** `login`, `signup`, `loginWithGoogle`, `loginWithMicrosoft`, `logout`, `updateProfile`, `setSession`

**Persistence:**
- JWT stored in `localStorage.jwt_token`
- User ID stored in `localStorage.user_id`

**Token refresh:** Handled by ApiClient (401 -> POST /auth/refresh -> retry original request)

**Socket lifecycle:** Calls `useSocketConnection(token, userId)` to connect/disconnect socket on auth changes.

**Dev bypass:** `dev-bypass-token` in localStorage skips auth and uses a mock user.

**HMR safe:** Uses global context key `__AUTH_CONTEXT__` on `globalThis` to persist across hot reloads.


### WorkspaceContext (`src/contexts/WorkspaceContext.tsx:1-439`)

**State:**
```typescript
{
  workspaces: Workspace[],
  currentWorkspace: Workspace | null,
  defaultWorkspaceId: string | null,
  isWorkspaceReady: boolean,
  isWorkspaceSwitching: boolean,
  workspaceError: string | null
}
```

**Actions:** `switchWorkspace`, `setDefaultWorkspace`, `retryWorkspaceLoad`

**Blocking gate:** Does not render children until a workspace is resolved. Shows loading or error UI. This prevents routes from rendering without workspace context.

**Resolution priority:** lastUsedWorkspaceId -> defaultWorkspaceId -> API isDefault -> first workspace

**Persistence:**
- `localStorage.lastUsedWorkspaceId` - active workspace
- `localStorage.defaultWorkspaceId` - default preference
- Legacy migration: converts old `currentWorkspaceId` key to `lastUsedWorkspaceId`

**On switch:**
- Clears ALL React Query cache via `queryClient.clear()`
- Redirects resource-specific pages to home

This is a **separate context** from AuthContext, contrary to some documentation that suggested workspace state lived in AuthContext. The workspace role comes from the workspace object, not the user object.


### SidebarStateProvider (`src/components/layout/SidebarContext.tsx:1-24`)

**State:** `{ collapsed: boolean }`
**Actions:** `toggleSidebar`

Simple toggle. No persistence (resets on page refresh).


### TemplateHeaderContext (`src/features/templates/contexts/TemplateHeaderContext.tsx`)

Feature-level context for template view header state. Scoped to TemplateViewPage.


## React Query

### Configuration (`src/App.tsx:125-132`)

```
QueryClient defaults:
  retry: 1
  refetchOnWindowFocus: false
```

### Centralized Query Keys (`src/api/hooks.ts:75-112`)

All query keys are defined in a single location for cache management:

profile, integrations, audiences, campaigns, folders, lists, listData, workspace, team, apiKeys, creditBreakdown, list, recentSheets, columnConfigs, subscription, referralStats, referralCode, userReferrals, myReferrer, couponStats, coupons, pixelStatus, tutorials, tutorialDocuments, templateMappings, allTemplateMappings, templateRegistry, onboardingProgress, salesforceAccounts, salesforceFields

### Stale Times

| Data | Stale Time | Reasoning |
|------|-----------|-----------|
| Profile | 5 min | Rarely changes |
| Integrations | 2 min | Moderate change frequency |
| Folders | 2 min | Moderate change frequency |
| Workspace | 5 min | Rarely changes |
| Team | 60 sec | May change with invitations |
| Audiences | 30 sec | Platform sync status changes |
| Campaigns | 60 sec | Campaign data changes |
| OTP status | 10 sec | Polling |
| Most others | 0 (default) | Always refetch |

### Cache Invalidation Patterns

- **Mutations:** All mutations invalidate related query keys via `qc.invalidateQueries()`
- **Workspace switch:** `queryClient.clear()` (nuclear, clears everything including non-workspace data)
- **User initialization:** `queryClient.refetchQueries()` on folders key
- **No optimistic updates:** Despite React Query supporting them, no mutations use optimistic updates

### Workspace-Scoped Query Keys

Query keys that include workspace ID: `["folders", wsId]`, `["folder", wsId, id]`, `["workspace", wsId]`, `["team", wsId]`, `["pixel-status", wsId]`

Non-workspace query keys: `["subscription"]`, `["profile"]`, `["credit-breakdown"]`

The `enabled: !!wsId` pattern prevents queries from firing before auth resolves.


## Socket.IO State

### Socket Singleton (`src/lib/socket.ts:1-362`)

Single `SocketService` instance shared across the app. Hooks register/deregister listeners.

**Listener registry:** Survives disconnect/reconnect cycles by design (stored in `listenerRegistry` array). The registry grows over time and is only cleaned when individual listeners call their cleanup function.

### Socket Hooks (17 hooks in `src/hooks/`)

| Hook | File | Purpose |
|------|------|---------|
| use-socket-connection | hooks/ | Lifecycle (connect on auth, disconnect on logout) |
| use-dataset-socket | hooks/ | Subscribe/unsubscribe to dataset rooms |
| use-import-socket | hooks/ | Import progress events |
| use-column-config-socket | hooks/ | Column config change events |
| use-normalization-socket | hooks/ | Normalization progress events |
| use-validation-socket | hooks/ | Email validation progress events |
| use-webhook-socket | hooks/ | Webhook data received events |
| features/batch-enrichment/hooks/useEnrichmentSocket | features/ | Enrichment progress |
| features/notifications/hooks/useEnrichmentNotifications | features/ | Notification events |

**Socket-to-state flow:**
1. Component mounts, registers listener via `socketService.on()` or `socketService.onAny()`
2. Server emits event with payload
3. Hook callback updates local state or invalidates React Query cache
4. Component re-renders

Socket events primarily trigger React Query invalidation rather than maintaining parallel state.


## Enrichment State Architecture

The enrichment system uses a strict separation to avoid re-renders during high-frequency updates:

**User edits (React pathway):**
```
onCellValueChanged -> cellEditQueue (debounced) -> PATCH API
```

**Enrichment updates (non-React pathway):**
```
socket events -> cellStatusTracker (module-level singleton) -> applyTransaction() -> grid refresh
```

These pathways are structurally isolated. `applyTransaction()` does NOT fire `onCellValueChanged`, making it impossible for enrichment to reach the edit queue.

The `cellStatusTracker` uses a publish/subscribe pattern with `requestAnimationFrame`-batched notifications. Subscribers (like `useEnrichmentRefresh`) receive batched updates and call `gridApi.refreshCells()` to trigger cell renderer re-reads.


## localStorage Keys

| Key | Value | Purpose |
|-----|-------|---------|
| `jwt_token` | JWT string | API authentication |
| `user_id` | User ID | Session identity |
| `lastUsedWorkspaceId` | Workspace ID | Persist active workspace across reloads |
| `defaultWorkspaceId` | Workspace ID | Default workspace preference |
| `currentWorkspaceId` | Workspace ID | Legacy key (migrated to lastUsedWorkspaceId) |
| `has_logged_in_before` | `"true"` | First-login detection |
| `pending_referral_code` | Code string | Referral tracking during auth |
| `pending_partner_code` | Code string | Partner code during auth |
| `columnMapping_{listId}_{templateId}` | JSON | Template field mapping cache |
| `dev-bypass-token` | Token string | Dev-only auto-authentication |


## sessionStorage Keys

| Key | Value | Purpose |
|-----|-------|---------|
| `pendingTemplateId` | Template UUID | Cross-page template activation flow |
| `pendingTemplateName` | Template name | Cross-page template activation flow |


## URL State

| Parameter | Route | Purpose |
|-----------|-------|---------|
| `?mappingId=xyz` | List view | Template context for column logos |
| `?createnewlist=true` | Datasets | Auto-open create list modal |
| `?partner=CODE` | Auth | Referral code |
| React Router `state` | List view | `templateMapped`, `templateId`, `mappingId` |


## Custom Hooks (Global - 17 files in `src/hooks/`)

| Hook | Purpose |
|------|---------|
| use-auto-step | Auto-advance stepper logic |
| use-column-config-socket | Column config socket events |
| use-dataset-data | Dataset data fetching |
| use-dataset-socket | Dataset room subscription |
| use-feature-check | Feature flag check (**stub - always returns true**) |
| use-has-permission | RBAC permission check (two-tier: API permissions, then role fallback) |
| use-import-socket | Import progress socket events |
| use-mobile | Mobile viewport detection |
| use-modal-state | Modal open/close helpers (replaces multiple useState(false)) |
| use-mosaic-grid | MosaicGrid layout helpers |
| use-normalization-socket | Normalization socket events |
| use-socket-connection | Socket lifecycle management |
| use-toast | Toast notification helper (Radix-based, legacy) |
| use-user-initialization | New user default setup (3-second delay before folder creation check) |
| use-validation-socket | Validation socket events |
| use-webhook-socket | Webhook data socket events |
| use-workspace-id | Current workspace ID shortcut |


## Toast/Notification State

Two systems running side-by-side:
- **Sonner** (preferred) - `import { toast } from "sonner"` - clean pill notifications. Used by user initialization and newer code.
- **shadcn Radix toast** (legacy) - `import { toast } from "@/hooks/use-toast"` - wider box style. Used by most existing hooks.
- Migration plan: move all to Sonner, remove Radix toast infrastructure.


## Known Concerns

1. **Feature check stub**: `use-feature-check.ts` always returns `{ enabled: true, loading: false }`. No actual feature gating exists in the frontend.
2. **Nuclear cache clear**: Workspace switch calls `queryClient.clear()` which is destructive; non-workspace data (subscription, profile) is unnecessarily cleared and must refetch.
3. **Socket listener growth**: The `listenerRegistry` grows over time with no global cleanup mechanism.
4. **No optimistic updates**: All mutations show loading state even for likely-successful operations.
5. **Session validation gap**: `isAuthenticated` in AuthContext has no server-side validation interval; a revoked session stays "authenticated" until the next API call returns 401.
6. **Role defaults to editor**: Unknown roles in `normalizeRole()` default to `editor` rather than the more restrictive `viewer`.
