---
title: Routing
last-updated: 2026-04-02
primary-audience: frontend
source: consensus (audit/outside-in + audit/inside-out)
---

# Routing

## Overview

All routes are defined in `src/App.tsx:145-194`. The app uses React Router v6 with a flat route structure. All 34 page components are lazy-loaded via `React.lazy()` with a single shared `Suspense` fallback (centered spinner).

Lazy imports: `src/App.tsx:90-123`


## Public Routes (No Auth Required)

These routes are accessible without authentication:

| Path | Component | Purpose |
|------|-----------|---------|
| `/login` | AuthPage | Login form (email/password + Google + Microsoft) |
| `/signup` | AuthPage | Signup form (same component as login) |
| `/auth` | AuthRedirect | Legacy redirect: partner code -> /signup, else -> /login |
| `/verify-otp` | VerifyOTPPage | 6-digit OTP entry form |
| `/verify` | EmailVerificationPage | Magic-link email verification (token in ?user= param) |
| `/forgot-password` | ForgotPasswordPage | Password reset request |
| `/reset-password` | ResetPasswordPage | New password entry (after OTP) |
| `/auth/callback` | OAuthCallbackPage | OAuth provider callback handler |
| `/auth/microsoft/complete` | MicrosoftCompletePage | Microsoft OAuth completion |
| `/accept-invitation` | AcceptInvitationPage | Workspace invitation acceptance |
| `/mcp-auth` | McpAuthPage | MCP (Model Context Protocol) authentication |
| `/mcp-signup` | McpSignupPage | MCP signup flow |
| `/payment/success` | PaymentSuccessPage | Stripe success redirect |
| `/payment/cancel` | PaymentCancelPage | Stripe cancel redirect |
| `*` | NotFound | 404 catch-all |

Reference: `src/App.tsx:145-170`


## Protected Routes (Inside AppLayout)

All protected routes are children of `<Route element={<NovuProviderWrapper><AppLayout /></NovuProviderWrapper>}>`. Authentication is enforced by `AppLayout.tsx:13-17`: checks `isAuthenticated`, shows loader during `isLoading`, redirects to `/login` when unauthenticated. `WorkspaceProvider` acts as a secondary gate, blocking rendering until workspace is resolved.

| Path | Component | Purpose |
|------|-----------|---------|
| `/` | HomePage | Dashboard with quick actions, stats, getting started |
| `/datasets` | DatasetsPage | Folder listing (all folders) |
| `/datasets/upload` | DatasetsUploadPage | CSV upload flow |
| `/datasets/:folderId` | FolderPage | Single folder view with lists |
| `/datasets/:projectId/:datasetId/original` | ListViewPage | AG Grid data view of a list |
| `/datasets/:projectId/:datasetId/dedupe` | DedupeRedirect | Legacy redirect -> /original |
| `/datasets/:projectId/:listId/template/:templateId/mapping/:mappingId` | TemplateViewPage | Template-mapped grid view |
| `/integrations` | IntegrationsPage | Integration catalog and connected accounts |
| `/live-audiences` | LiveAudiencePage | Audience management (LinkedIn, Meta) |
| `/live-audiences/:audienceName/analytics` | AudienceAnalyticsPage | Per-audience analytics |
| `/linkedin-analytics` | LinkedInAnalyticsPage | LinkedIn ad analytics |
| `/meta-analytics` | MetaAnalyticsPage | Meta ad analytics |
| `/smartleads-analytics` | SmartLeadsAnalyticsPage | SmartLead analytics |
| `/pixels-and-tags` | PixelsTagsPage | Tracking pixel setup |
| `/refer-and-earn` | ReferEarnPage | Referral program |
| `/templates` | TemplatesPage | Template gallery (47 templates) |
| `/welcome/template` | TemplateLinkHandler | Template deep link handler (marketing/onboarding links) |
| `/how-to-use` | TutorialPage | Tutorial/documentation page |
| `/settings` | SettingsPage | User/workspace settings (9 tabs) |
| `/pricing` | PricingPage | Pricing plans |
| `/feedback` | FeedbackPage | User feedback |
| `/dev/style-guide` | StyleGuidePage | Dev-only style reference (only when `import.meta.env.DEV` is true) |

Reference: `src/App.tsx:172-194`


## Redirects

| From | To | Mechanism |
|------|----|-----------|
| `/auth` | `/signup?partnercode=CODE` or `/login` | AuthRedirect component (`src/App.tsx:22-36`) |
| `/datasets/:projectId/:datasetId/dedupe` | `/datasets/:projectId/:datasetId/original` | DedupeRedirect component (`src/App.tsx:9-12`) |


## Route Guard Implementation

The app does NOT use the `ProtectedRoute` component despite it existing in `src/components/auth/ProtectedRoute.tsx`.

Instead, auth protection works through three layers:

1. **Route level**: `AppLayout.tsx:13-17` checks `isAuthenticated` from AuthContext. Shows loader during `isLoading`. Redirects to `/login` when unauthenticated.
2. **Workspace level**: `WorkspaceProvider` blocks rendering until a workspace is resolved. Shows loading/error UI during resolution.
3. **Component level**: Individual components use `useHasPermission()` to gate UI elements (buttons, menus, tabs).

There are **no route-level role or permission checks**. A viewer can navigate to any protected route URL; restrictions only apply at the UI element level.


## URL Parameters

| Route | Param | Purpose |
|-------|-------|---------|
| `/datasets/:projectId/:datasetId/original` | `:projectId` | Folder ID |
| `/datasets/:projectId/:datasetId/original` | `:datasetId` | List ID |
| `/datasets/:projectId/:datasetId/original` | `?mappingId=` | Template context for column logos |
| `/datasets/:projectId/:listId/template/:templateId/mapping/:mappingId` | `:templateId`, `:mappingId` | Template mapping context |
| `/datasets` | `?createnewlist=true` | Auto-open create list modal |
| `/auth` | `?partner=CODE` | Partner referral code |
| `/linkedin-analytics` | `?audienceId=`, `?connectedAccountId=` | Analytics context |

> INVESTIGATE: URL param naming is inconsistent. `:projectId` and `:folderId` refer to the same concept (a folder/project). `:datasetId` and `:listId` also refer to the same entity. This creates confusion when reading code.


## Navigation Patterns

### Template Activation
1. User selects template in gallery (`/templates`)
2. Navigates to `/datasets/:folderId/:listId/original?mappingId=xyz` with React Router state (`templateMapped`, `templateId`, `mappingId`)
3. Grid loads with template-mapped columns

### Template Pending Flow
Uses `sessionStorage` to survive cross-page navigation:
1. User selects template but needs a new list
2. `pendingTemplateId` and `pendingTemplateName` saved to sessionStorage
3. Navigate to `/datasets?createnewlist=true`
4. After list creation, return to template flow

### Workspace Switch
1. User selects new workspace in sidebar
2. `queryClient.clear()` wipes entire React Query cache
3. Navigate to `/` (home)
4. All data refetches for new workspace context

### Template Deep Links
`/welcome/template` handles `GET /welcome/initialize-template?email=` for welcome email template initialization.


## Page Components (35 total)

AcceptInvitationPage, AudienceAnalyticsPage, AuthPage, DatasetsPage, DatasetsUploadPage, EmailVerificationPage, FeedbackPage, FolderPage, ForgotPasswordPage, HomePage, Index (unused), IntegrationsPage, LinkedInAnalyticsPage, ListViewPage, LiveAudiencePage, McpAuthPage, McpSignupPage, MetaAnalyticsPage, MicrosoftCompletePage, NotFound, OAuthCallbackPage, PaymentCancelPage, PaymentSuccessPage, PixelsTagsPage, PricingPage, ReferEarnPage, ResetPasswordPage, SettingsPage, SmartLeadsAnalyticsPage, StyleGuidePage, TemplateLinkHandler, TemplateViewPage, TemplatesPage, TutorialPage, VerifyOTPPage

Reference: `src/pages/` (35 files), `src/App.tsx:90-123` (lazy imports)


## Known Concerns

- **Stale CLAUDE.md count**: CLAUDE.md says "14 page routes" but there are 35 pages and 35+ route paths. This mismatch could mislead AI agents.
- **Dead file**: `src/pages/Index.tsx` exists but is never referenced in routing.
- **Dead component**: `ProtectedRoute.tsx` exists but is unused.
- **No route-level RBAC**: All role/permission enforcement happens at the component level, not the route level. A URL in the browser bar bypasses visual restrictions.
- **Three separate analytics pages**: LinkedIn, Meta, SmartLead each have dedicated page components. These likely share 80%+ of their logic and could be parameterized.
- **Dedupe redirect**: The deduplication feature was once a standalone page but is now a drawer within ListViewPage. The redirect exists for backward compatibility.
