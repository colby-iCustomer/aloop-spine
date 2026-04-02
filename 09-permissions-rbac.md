---
title: Permissions & RBAC
last-updated: 2026-04-02
primary-audience: all
source: consensus (audit/outside-in + audit/inside-out)
---

# Permissions & RBAC

## Role Hierarchy

```
Owner > Admin > Editor > Viewer
```

Reference: `src/lib/permissions.ts:1-98`


## Role Definitions

| Role | Type Definition | Description | Assignable |
|------|----------------|-------------|------------|
| owner | `"owner"` | Workspace billing owner. One per workspace. Cannot be removed or demoted. | No (transferred, not assigned) |
| admin | `"admin"` | Workspace team manager. Invites, roles, settings. | Yes |
| editor | `"editor"` | Audience creator. Builds, edits, exports. Cannot manage team. | Yes |
| viewer | `"viewer"` | Read-only access. Views audiences and settings. | Yes |

TypeScript type: `export type UserRole = "owner" | "admin" | "editor" | "viewer"` (`src/api/types/auth.ts:7`)

### Legacy "member" Role

The backend may still return `"member"` as a role for older workspaces. The `normalizeRole()` function (`src/lib/permissions.ts:92-97`) maps:
- `"member"` -> `"editor"` (silent upgrade)
- `null`/`undefined`/unknown -> `"editor"` (permissive default)

> INVESTIGATE: Unknown roles default to `editor` rather than `viewer`. This is a permissive default that grants create/edit/export access to unrecognized roles.


## Permission Matrix

| Permission String | Owner | Admin | Editor | member* | Viewer |
|-------------------|:-----:|:-----:|:------:|:-------:|:------:|
| workspace.manage | Y | Y | - | - | - |
| folder.view | Y | Y | Y | Y | Y |
| folder.create | Y | Y | Y | Y | - |
| folder.edit | Y | Y | Y | Y | - |
| folder.delete | Y | Y | Y | Y | - |
| list.create | Y | Y | Y | Y | - |
| list.delete | Y | Y | Y | Y | - |
| list.edit | Y | Y | Y | Y | - |
| data.export | Y | Y | Y | Y | - |
| credits.allocate | Y | - | - | - | - |
| tutorial.manage | Y | Y | - | - | - |

*`member` has identical permissions to `editor`

Reference: `src/lib/permissions.ts:44-67`


## Permission Check Implementation

### useHasPermission Hook (`src/hooks/use-has-permission.ts:1-17`)

Two-tier resolution:
1. Gets current user from AuthContext and current workspace from WorkspaceContext
2. If workspace has a `permissions` array (from API) with entries, checks if permission is included
3. If permissions array is empty, falls back to role-based lookup in static `ROLE_PERMISSIONS` map

This means the backend can override the frontend's role-based defaults by returning a permissions array per workspace.


### useFeatureCheck Hook (`src/hooks/use-feature-check.ts:1-16`)

**Stub implementation.** Always returns `{ enabled: true, loading: false }`.

Has a TODO comment about wiring to `GET /features/{key}/check` backend endpoint. No actual feature gating exists in the frontend.


## Where Permissions Are Checked

### Sidebar (`src/components/layout/AppSidebar.tsx:212-215`)
- `useHasPermission("folder.view")` - folder section visibility
- `useHasPermission("folder.create")` - create folder button
- `useHasPermission("folder.edit")` - folder edit actions
- `useHasPermission("folder.delete")` - folder delete actions

### Settings Page (`src/lib/permissions.ts:75-84`)

Tab visibility by role (SETTINGS_TAB_ROLES):

| Tab | Visible To |
|-----|-----------|
| profile | owner, admin, editor, viewer |
| workspace | owner, admin, editor, viewer |
| ai-agents | owner, admin, editor |
| usage | owner, admin, editor, viewer |
| billing | owner, admin, editor, viewer |
| mcp | owner, admin |
| api-keys | owner, admin, editor |
| partners | owner, admin, editor |

### Route Level

**There are no route-level permission checks.** Auth protection is handled by `AppLayout.tsx` (checks isAuthenticated), but all authenticated users can navigate to any protected route URL. Role/permission restrictions only apply at the component/button level.

`ProtectedRoute.tsx` exists in `src/components/auth/` but is NOT used in routing.


## User Types

| Type | Email Domain | Signup Behavior | Workspace | Available Roles |
|------|-------------|----------------|-----------|-----------------|
| Full User | Business domain | Own workspace created on signup | Full access | All roles |
| Guest Viewer | Personal domain (gmail, yahoo, etc.) | Dormant personal workspace (Viewer in own ws) | Hidden from selector | Viewer only in shared workspaces |

Guest Viewers require Pro plan invitation to join a shared workspace.


## Plan Gating

Three plan tiers with frontend-enforced feature gating:

| Feature | Free | Starter | Professional |
|---------|------|---------|-------------|
| Team members | 1 | 3 | 10 |
| Templates | 3 specific UUIDs | All | All |
| Integrations | 5 specific names | All | All |
| Pixels and Tags | No | Yes | Yes |
| Multiple workspaces | No | No | Yes |

> INVESTIGATE: Plan capabilities are **hardcoded in the frontend**. No API endpoint exposes "what can this plan access?" Consequences:
> - Frontend is the ONLY enforcer for free-plan restrictions
> - MCP callers cannot check feature access without hardcoding same values
> - If template UUIDs or integration names change, frontend breaks silently
> - A motivated user could bypass plan limits via direct API calls


## Workspace Authorization

### How It Works

- `workspace-id` header is injected on every API request via interceptor (`src/api/client.ts`)
- `WorkspaceContext` resolves active workspace before any data loads
- Switching workspace clears all React Query cache
- Workspace role comes from the workspace object, not the user object

### Security Gap

> INVESTIGATE: No `X-Workspace-Id` header middleware exists on the backend. All workspace-scoped endpoints should validate workspace membership before processing. Without this, users can potentially access resources across workspaces by ID. The frontend navigates to `/` on workspace switch as a stopgap but this is not a security boundary.


## Workspace Creation Gating

The `WorkspaceSelector` enforces Professional plan gating for new workspace creation. However, the sidebar's create button does NOT check plan, creating an inconsistency.


## Auth Flow Protection Layers

| Layer | Mechanism | What It Protects |
|-------|-----------|-----------------|
| Route | AppLayout checks `isAuthenticated`, redirects to /login | Unauthenticated access |
| Workspace | WorkspaceProvider blocks rendering until workspace resolves | Data access without workspace |
| API | JWT token on all requests, 401 triggers token refresh | Server-side auth enforcement |
| UI | useHasPermission per component | Role-inappropriate actions |

Missing layer: **No route-level role checks.** A viewer can navigate to any route URL; they only see restricted buttons/menus hidden.


## What Is Missing (From Old Repo)

| Feature | Old Repo | New Repo |
|---------|----------|----------|
| PrivateRoute with requiredPermission | Route-level permission gating | Not rebuilt; component-level only |
| Resource sharing (GET/POST/DELETE /workspaces/:id/shares/*) | Shared resources between workspaces | Not implemented |
| Leave workspace (POST /workspaces/:id/leave) | Self-removal from workspace | Endpoint not available |
| Layer 2 Supabase app roles | Legacy parallel system | Correctly not carried forward |


## Known Concerns

1. **No feature gating**: useFeatureCheck always returns true. No per-plan or per-workspace feature restrictions on the frontend.
2. **No route-level RBAC**: A viewer can navigate to any URL; restrictions are UI-element level only.
3. **Viewer can see billing**: Settings tab visibility shows viewers can access billing and usage tabs, potentially exposing sensitive cost data.
4. **Permissive role default**: Unknown roles default to `editor` instead of `viewer`.
5. **No audit trail**: Permission checks are purely UI-side with no logging.
6. **Frontend-only plan enforcement**: Plan limits are hardcoded and enforced only in the browser.
