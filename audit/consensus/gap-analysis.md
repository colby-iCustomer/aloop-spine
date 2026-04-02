# Gap Analysis - Outside-In vs Inside-Out Audit Consensus

Generated: 2026-04-02
Method: Systematic comparison of docs-derived (outside-in) and code-derived (inside-out) audits across all 11 topics.


## Topic 00: Product Context

**Agreements**
- Both confirm ALoop is a B2B audience data management / enrichment platform
- Both confirm template-driven workflow as the primary product pattern
- Both confirm folder/list data hierarchy with AG Grid interface
- Both confirm credit-based enrichment pricing
- Both confirm multi-platform export (LinkedIn, Meta, Google Ads, Reddit, HubSpot, SmartLeads, Mailshake, Google Sheets)
- Both confirm workspace-based multi-tenancy with role-based access
- Both confirm MCP (Model Context Protocol) support

**Divergences**
- Outside-In says "14 routes." Inside-Out found **34 lazy-loaded page components** across public and protected routes.
  - Code is correct. CLAUDE.md had a stale count. The app has grown significantly.
  - Recommended action: Update any references to route count.

- Outside-In lists 6 data entry sources. Inside-Out found 7: CSV, Google Sheets, HubSpot, Salesforce, Apollo/ICP, Klaviyo, Webhooks.
  - Code is correct. Salesforce import (via Polytomic adapter) exists in code but is not documented.
  - Recommended action: Document Salesforce import as a current capability.

- Outside-In describes 3 plan tiers (Free $0/200, Starter $99/2000, Pro $299/7000). Inside-Out does not challenge this but notes plan capabilities are hardcoded in frontend.
  - Both are correct. Docs describe plan tiers; code confirms hardcoded enforcement.
  - Recommended action: Note this as current state with the caveat that pricing is frontend-only.

**Missing From Docs**
- Microsoft OAuth support (code has MicrosoftOAuthButton, /auth/microsoft/complete route)
- Pixels & Tags feature (dedicated page and sidebar nav item)
- Salesforce import via Polytomic adapter with dedicated wizard
- Dev-only style guide route (/dev/style-guide)
- Dev bypass token mechanism for development auth
- Deduplication feature (find duplicates by criteria, merge with field-level control)
- Email validation via ZeroBounce
- Data normalization (contact/account entity types)
- Onboarding progress tracking API
- Tutorial/How-to-Use page with video tutorials
- Feedback page

**Missing From Code**
- No competitive landscape documentation (docs gap, not a code concern)
- No user personas or journey maps
- No product roadmap artifacts

**Regressions**
- ICP Prospecting and Lookalike Prospecting: Outside-In flags these as "not yet rebuilt." Inside-Out found `ICPProspectingModal`, `LookalikeForm`, `LookalikePreviewModal`, and `AlphaTemplateModal` components exist. These appear to have been rebuilt.
- Salesforce sync: Outside-In says "not mentioned in current docs." Inside-Out found SalesforceImportWizard, SalesforceSyncModal, and dedicated services. Rebuilt but undocumented.
- Web search agent: Outside-In flags as missing. Inside-Out found `WebSearchAgentModal` exists. Rebuilt but undocumented.
- Feedback page: Outside-In flags onboarding gap. Inside-Out found `FeedbackPage` route exists.
- Apollo prospecting: Both audits confirm this exists in code with full service layer.


## Topic 01: Tech Stack

**Agreements**
- React 18, Vite 5, TypeScript, Tailwind CSS 3, shadcn/ui, Lucide React, React Router v6
- AG Grid v35 Enterprise with Quartz JS theming API
- React Query / TanStack Query for server state
- Socket.IO client for real-time
- react-hook-form + zod for forms
- @react-oauth/google + jwt-decode v4 for auth
- @novu/react v3.10.1 for notifications
- Sonner for toast notifications (preferred, replacing Radix toast)
- Bun as package manager

**Divergences**
- Outside-In says "TypeScript (unspecified version)." Inside-Out found TypeScript ^5.8.3.
  - Code is correct with exact version.
  
- Outside-In says TipTap was in old repo and "not mentioned in new repo docs." Inside-Out found TipTap v3 IS in the new repo (TiptapColumnEditor for column formula/prompt editing).
  - Code is correct. TipTap was carried forward but docs did not track it.
  - Recommended action: Document TipTap as part of current stack.

- Outside-In says Recharts was in old repo only. Inside-Out found Recharts ^2.15.4 in package.json.
  - Code is correct. Recharts was carried forward for analytics pages.
  - Recommended action: Document Recharts as current dependency.

- Outside-In says dnd-kit was old repo only. Inside-Out did not find dnd-kit in package.json.
  - Both agree dnd-kit was dropped. Confirmed.

- Outside-In says react-querybuilder was old repo only. Inside-Out did not find it.
  - Confirmed dropped.

**Missing From Docs**
- Framer Motion (^12.34.4) for animation
- TipTap v3 for rich text editing in column formula editor
- Recharts for charts on analytics pages
- react-markdown + remark-gfm for markdown rendering
- next-themes for dark mode (unusual for non-Next.js app)
- embla-carousel-react for carousel
- react-resizable-panels for panel layouts
- react-day-picker for date selection
- cmdk for command palette
- vaul for drawer component
- input-otp for OTP input
- lovable-tagger dev dependency
- Vitest + @testing-library/react for testing (configured but no tests written)
- Exact versions of all dependencies (only available from package.json)

**Missing From Code**
- Backend stack details (Node/Express, PostgreSQL, Drizzle, Redis/Bull, Stripe) are backend-only; frontend repo cannot verify

**Regressions**
- highlight.run (error tracking): Old repo had it, neither current docs nor code have it. **No error tracking exists.**
- PostHog (analytics): Old repo had it, neither current docs nor code have it. **No product analytics exists.**
- Intercom (support widget): Old repo had it, neither current docs nor code have it. **No support widget exists.**
- ReactFlow (flow diagrams): Old repo had it, not in new repo. Confirmed dropped.
- three.js: Old repo had it, documented as "not in scope." Confirmed dropped.


## Topic 02: Architecture

**Agreements**
- React SPA with Vite, feature-based directory structure
- Two API backends: apiClient (list-service) and gatewayClient (gateway)
- All routes lazy-loaded via React.lazy + Suspense
- Feature module pattern: types, hooks, components, services per feature
- React Query for all server state
- Socket singleton for real-time events
- fetch-based ApiClient (not Axios)

**Divergences**
- Outside-In shows NovuProviderWrapper wrapping AuthProvider. Inside-Out shows NovuProviderWrapper wrapping only the AppLayout route (not the full tree).
  - Code is correct. Novu context is only available on authenticated routes, not public pages.
  - Recommended action: Use the Inside-Out provider hierarchy as authoritative.

- Outside-In says "WorkspaceContext OR workspace state in AuthContext." Inside-Out found WorkspaceContext is a **separate, dedicated context** with its own file (439 lines), blocking gate, and full state management.
  - Code is correct. WorkspaceContext is definitively separate from AuthContext.
  - Recommended action: Eliminate the ambiguity in documentation.

- Outside-In mentions 14 routes. Inside-Out found 34 lazy-loaded pages.
  - Code is correct. 34 pages, not 14.

**Missing From Docs**
- SidebarStateProvider context (simple collapse toggle)
- TemplateHeaderContext (feature-level context for template view)
- AppErrorBoundary (class component between providers and routes)
- TooltipProvider wrapping the app
- Dual toast systems both rendered in App.tsx (Toaster + Sonner)
- The enrichment and batch-enrichment features have nearly identical structures (potential refactor target)
- Vite proxy configuration details (list-service, gateway, socket.io)
- QueryClient configuration (retry: 1, refetchOnWindowFocus: false)

**Missing From Code**
- Backend architecture details (Bull queues, Python enrichment agent, Airbyte) are backend-side only

**Regressions**
- Old repo had 5 HTTP client abstractions; new repo consolidated to 2 (confirmed improvement)
- Old repo had Supabase direct queries; no Supabase found in new code (confirmed removed)
- Old repo had Jotai atom cascade; replaced by WorkspaceContext + React Query (confirmed improvement)
- Old repo ListView was 2,954 lines; new repo decomposed into feature hooks and 30+ grid components (confirmed improvement, though grid module complexity noted)


## Topic 03: UI Components

**Agreements**
- 50 shadcn/ui primitives in components/ui/
- StatusBadge, PageHeader, EmptyState, InfinityLoader as shared components
- Card family system (Action, Stat, Connect, Gallery)
- Grid feature module with toolbar, enrichment UI, cell renderers
- Template feature with card, preview, mapping, progress components
- Notification feature with bell, popover, Novu items
- Skeleton loading patterns

**Divergences**
- Outside-In lists 6 auth components. Inside-Out found 6 files including MicrosoftOAuthButton.tsx and InfinityLoop.tsx (not mentioned in docs).
  - Code has more auth components than documented.

- Outside-In says "Settings: WorkspaceTab, TeamManagementTab, BillingPlansTab, UsageTab, APIKeysTab, IntegrationsTab." Inside-Out found 9 tabs: AIAgentsTab, APIKeysTab, BillingPlansTab, InvitePartnersTab, MCPServerTab, ProfileTab, TeamManagementTab, UsageTab, WorkspaceTab.
  - Code has 3 more settings tabs than docs describe (AIAgents, InvitePartners, MCPServer, Profile).
  - Recommended action: Document all 9 settings tabs.

**Missing From Docs**
- MicrosoftOAuthButton component
- InfinityLoop animation component
- SalesforceImportWizard in folder components
- LookalikesModal in folder components
- WorkspaceSwitchOverlay component
- PlatformLogo, RoleBadge, TemplateLogoRenderer shared components
- MosaicGrid shared component
- NavLink root component
- EventCellDetailsPanel, EventCellRenderer, EventColumnHeader
- DeduplicationDrawerContent, DeduplicationMergeDialog
- NormalizationDrawerContent, NormalizationFieldMapper
- ValidationDrawerContent, FilterDropdown, SortDropdown
- AlphaTemplateModal, IcpForm, LookalikeForm, LookalikeImportProgressModal, LookalikePreviewModal
- TemplateAgentGrid, TemplateCardSkeleton
- BatchEnrichModal (separate batch-enrichment feature)
- 8 export flow sub-components under grid/components/actions/flows/
- multi-select.tsx custom shadcn addition

**Missing From Code**
- Nothing identified. All documented components exist.

**Regressions**
- Old repo had 309 components across 37 directories. New repo has ~326 source files (comparable, reorganized into feature modules).
- Old repo had 35 grid components (8,499 lines). New repo has 30+ grid components. Density reduced but complexity still high.
- Old repo had overlapping credit dialogs. Inside-Out found EnrichmentConfirmDialog + OverageConfirmModal. Need to verify these are not duplicative.
- Old repo per-platform analytics pages with 80%+ shared logic: Inside-Out confirms 3 separate analytics pages still exist as separate routes (LinkedInAnalyticsPage, MetaAnalyticsPage, SmartLeadsAnalyticsPage). **Duplication likely persists.**


## Topic 04: Routing

**Agreements**
- All routes in App.tsx with React.lazy + Suspense
- Public auth routes (login, signup, verify, forgot-password, reset-password)
- Protected routes behind AppLayout auth check
- Template navigation with mappingId query param
- Style guide at /dev/style-guide (dev only)
- TemplateLinkHandler at /welcome/template
- Analytics routes (linkedin, meta, smartleads)

**Divergences**
- Outside-In says "14 routes." Inside-Out found 15 public + 20 protected = **35 total routes** (plus dev-only style guide).
  - Code is correct. The 14 count in CLAUDE.md is severely outdated.
  - Recommended action: Document actual route count.

- Outside-In says ProtectedRoute component is used for auth gating. Inside-Out found ProtectedRoute.tsx exists but is **NOT used** - auth gating is in AppLayout.tsx.
  - Code is correct. ProtectedRoute is dead code.
  - Recommended action: Note that ProtectedRoute is unused; AppLayout handles auth.

- Outside-In mentions `/datasets` as FolderPage. Inside-Out found `/datasets` is DatasetsPage (folder listing), `/datasets/:folderId` is FolderPage (single folder).
  - Code has an additional nesting level not captured in docs.

**Missing From Docs**
- /signup route (same AuthPage component as /login)
- /verify-otp route (OTP entry)
- /verify route (magic-link email verification)
- /auth/callback (OAuth callback handler)
- /auth/microsoft/complete (Microsoft OAuth)
- /accept-invitation route
- /mcp-auth and /mcp-signup routes
- /payment/success and /payment/cancel routes
- /datasets/upload route
- /datasets/:projectId/:datasetId/dedupe redirect
- /datasets/:projectId/:listId/template/:templateId/mapping/:mappingId (template view)
- /integrations page
- /live-audiences and /live-audiences/:audienceName/analytics
- /pixels-and-tags page
- /refer-and-earn page
- /how-to-use (tutorial) page
- /pricing page
- /feedback page
- NotFound (404) catch-all route
- URL param naming inconsistency (projectId vs folderId, datasetId vs listId)

**Missing From Code**
- Nothing. All documented routes exist plus many more.

**Regressions**
- Old repo had PrivateRoute with requiredPermission prop. New repo has NO route-level permission checks. All role/permission checks are component-level only. This is a downgrade in security posture.
- Old repo had 32 page components. New repo has 35 pages (all lazy-loaded, which is an improvement over old repo's synchronous loading).


## Topic 05: API Contracts

**Agreements**
- Two-service topology (apiClient for list-service, gatewayClient for gateway)
- Standard response format: { success, data, pagination? } or { success: false, error }
- Token refresh on 401 with retry
- Socket events split between grid (underscore format) and notification (colon format)
- Auth, workspace, folder, list, enrichment, export, payment endpoints present in both audits
- Integration OAuth + API key flows documented and confirmed in code

**Divergences**
- Outside-In says workspace endpoints use path `/workspaces/:id`. Inside-Out shows BOTH `/workspaces/:id` patterns (settings) AND `/workspaces/:wsId/projects` (folder listing).
  - Both are correct. Different endpoint groups use different scoping patterns.

- Outside-In flags `POST /auth/add-password` as missing. Inside-Out found `authService.addPassword` exists in code.
  - Code is correct. The endpoint was ported but not documented.

- Outside-In flags referral endpoints as "status unclear." Inside-Out found full referral service (stats, my-code, my-referrals, my-referrer, track).
  - Code is correct. Referrals are fully wired.

- Outside-In says invitations use `POST /invitations` (body-scoped). Inside-Out shows `POST /workspaces/:id/invitations` (path-scoped).
  - Code is authoritative. The endpoint is workspace-path-scoped.

- Outside-In says `GET /workspaces/:id/roles` is a "missing/requested endpoint." Inside-Out found `settingsService.getWorkspaceRoles` calling this endpoint.
  - Code is correct. Endpoint exists and is wired.

**Missing From Docs**
- Salesforce data service endpoints (/connections/*)
- Polytomic adapter endpoints
- Onboarding endpoints (progress, complete-task)
- Pixel status endpoint
- Tutorial and tutorial-documents endpoints
- Google Sheets sync endpoints (sync/now, sync/schedule, cancel)
- Deduplication endpoints (stats, find, merge)
- Validation endpoints (validate-emails, credits-status, validation-stats, cancel, delete-invalid)
- Normalization endpoints (normalize, cancel)
- Column config endpoints (CRUD, change-type, validate-conversion)
- Template registry endpoint (GET /templates/registry)
- Template mapping CRUD endpoints
- Webhook endpoint (create-webhook-table)
- Apollo prospecting endpoints (filter-options, preview, import)
- HubSpot import endpoint
- Consume-credits endpoint
- Cancel-subscription endpoint
- Connection provider endpoints (Salesforce, Instantly, Marketo)
- Multiple additional socket events (apollo_import_*, import:*, column:config_updated, webhook_data_received, webhook:column_mapped)
- Partner invite endpoint
- Promo validate and apply-to-existing endpoints

**Missing From Code**
- None identified. All documented endpoints exist plus many more.

**Regressions**
- `POST /payments/consume-credits`: Old repo had it as dead code. Inside-Out found `paymentsService.consumeCredits` exists in new repo. Wired but usage unclear.
- Resource sharing endpoints (GET/POST/DELETE /workspaces/:id/shares/*): Not found in new repo code. Confirmed missing.
- Workspace deletion (DELETE /workspaces/:id): Not found in either audit. Confirmed not available.


## Topic 06: State Management

**Agreements**
- Three-layer model: React Context (auth/workspace) + React Query (server state) + Socket.IO (real-time)
- No Redux, Zustand, or Jotai
- JWT and workspace ID in localStorage
- sessionStorage for pending template state
- queryClient.clear() on workspace switch (nuclear cache clear)
- useHasPermission hook with two-tier resolution (API permissions, then role fallback)
- useFeatureCheck as always-true stub
- Dual toast systems (Sonner preferred, Radix legacy)

**Divergences**
- Outside-In was ambiguous about WorkspaceContext vs workspace-in-AuthContext. Inside-Out confirms they are **separate contexts** - WorkspaceContext (439 lines) is distinct from AuthContext.
  - Code is correct. These are separate, with clear responsibilities.

- Outside-In documents cellStatusTracker and cellEditQueue as module-level singletons. Inside-Out does not specifically call these out but confirms the socket-to-state flow pattern.
  - Both are correct. Outside-In has more detail on the singleton pattern; Inside-Out focuses on the hook layer.

**Missing From Docs**
- TemplateHeaderContext (feature-level context)
- SidebarStateProvider context
- HMR-safe context pattern (globalThis key persistence)
- Socket listener registry that survives disconnect/reconnect
- use-user-initialization hook (3-second delay for default folder creation)
- Centralized query key definitions (src/api/hooks.ts:75-112)
- Full list of stale times per query type
- Dev bypass token mechanism in AuthContext
- Legacy workspace key migration (currentWorkspaceId -> lastUsedWorkspaceId)
- defaultWorkspaceId in localStorage
- pending_referral_code, pending_partner_code in localStorage
- Full list of 17 custom hooks

**Missing From Code**
- None identified.

**Regressions**
- Old repo Jotai atoms replaced by Context + React Query (confirmed improvement)
- Old repo CreditsProvider: Not found as a provider in new code; replaced by hooks (confirmed improvement)
- Old repo per-row enrichment status (_isEnriching magic prefixes): Not found in new code. cellStatusTracker singleton confirmed as replacement.
- Old repo no React Query / manual useEffect+useState: Fully replaced by React Query (confirmed improvement)
- Old repo dummyProgress (fake 0-15% enrichment progress): Not explicitly found in Inside-Out audit. **Needs investigation** to confirm whether blended progress is implemented in notification module.


## Topic 07: Integrations

**Agreements**
- 6 OAuth providers (LinkedIn, Meta, Reddit, Google Ads, Google Sheets, HubSpot)
- 4 API key providers (SmartLeads, Apollo, Klaviyo, Mailshake)
- OAuth flow: popup redirect, postMessage + polling fallback
- API key flow: validate-then-connect
- 8+ export destinations (LinkedIn, Meta, Reddit, Google Ads, Google Sheets, HubSpot, SmartLeads, Mailshake)
- Airbyte setup for LinkedIn and Meta post-connect
- Novu for push notifications
- Stripe for payments
- n8n webhook for AI field mapping

**Divergences**
- Outside-In says import flows for Google Sheets, HubSpot, Klaviyo are "stub UI only." Inside-Out found actual service methods for all three (createListFromGoogleSheets, startHubSpotImport, KlaviyoImportSheet with implied service).
  - Inside-Out is more current. At minimum, Google Sheets and HubSpot imports have wired services.
  - Recommended action: Re-evaluate import flow status; some may have been wired after docs were written.

- Outside-In does not mention Salesforce as an integration. Inside-Out found Salesforce as a Connection Provider with Polytomic adapter, dedicated import wizard, and sync modal.
  - Code is correct. Salesforce integration exists but was never documented.
  - Recommended action: Document Salesforce integration.

- Outside-In mentions "Onesource" as UI alias for system-managed Apollo. Inside-Out confirms Apollo has a "system" provider concept with cached provider ID.
  - Both describe the same thing from different angles.

**Missing From Docs**
- Salesforce integration (Polytomic adapter, import wizard, sync)
- Instantly and Marketo as "coming soon" connection providers
- Polytomic as the adapter layer name
- Google "Unverified App" warning tracking for Google Sheets and Google Ads
- LinkedIn pre-logout flag before OAuth
- CSV download as an export flow (9th export, not 8)
- Microsoft OAuth as a third-party service
- ZeroBounce as backend email validation service
- INTEGRATION_NAME_OVERRIDES mapping

**Missing From Code**
- Intercom (support widget) from old repo
- PostHog (analytics) from old repo
- highlight.run (error tracking) from old repo

**Regressions**
- Salesforce sync: Outside-In flags as missing. Inside-Out found it exists. **Not a regression; documentation gap.**
- Apollo prospecting: Both audits confirm it exists. **Not a regression.**
- Klaviyo import: Partially wired (API key connection + import UI exists, full wiring status needs investigation).
- Intercom: **Confirmed regression** - no support widget in new app.
- PostHog: **Confirmed regression** - no analytics tracking in new app.
- highlight.run: **Confirmed regression** - no error tracking in new app.


## Topic 08: Design System

**Agreements**
- HSL CSS variable system in index.css
- Semantic tokens (--background, --foreground, --primary, --success, etc.)
- Palette primitives (--accent-blue, --accent-purple, --accent-teal)
- Blue-branded color scheme (213 hue family)
- Border radius base: 0.625rem (10px)
- Focus ring lightened (213 60% 72%)
- Card family system (Action, Stat, Connect, Gallery/Info/Step)
- PLG utility classes (.plg-card, .status-dot, .data-label, .mono-value)
- Pill pattern (green, blue, amber)
- Background gradient (145deg, blue-tinted)
- Quick Actions gradient (--qa-gradient)
- Getting Started panel tokens (--gs-bg, --gs-border)
- Dark mode via .dark class on root
- Style guide at /dev/style-guide

**Divergences**
- Outside-In does not mention font system. Inside-Out found Inter as primary font with specific weights (400, 500, 600, 700).
  - Code is authoritative. Inter is the font.

- Outside-In documents typography hierarchy (L1-L4). Inside-Out confirms tracking-tight on headings but does not enumerate the hierarchy.
  - Docs provide better organization of the hierarchy; code confirms the values.

**Missing From Docs**
- Inter font specification and Google Fonts import
- Full dark mode HSL values (Inside-Out enumerated all of them)
- Sidebar-specific CSS variables (7 variables)
- Layout variables (--topbar-height: 56px, --page-max-width: 1800px)
- Card family CSS variables (min/max widths)
- Animation keyframes (modal-in/out, overlay-in/out, grid-item-enter)
- Accordion animation keyframes in Tailwind config
- PLG action color variants (.plg-action-blue, .plg-action-purple, .plg-action-teal)
- .plg-action-tilt for auto-cycle rotation
- .page-container, .page-constrained, .grid-responsive, .grid-fixed utility classes
- Scrollbar utility classes (.hide-scrollbar, .thin-scrollbar)
- AG Grid theme details (font sizes, row height, cell padding, accent color, scrollbar colors)
- TipTap editor styles in index.css
- .mosaic-grid animation suppression with !important
- index.css is 617 lines (maintenance concern)
- --accent-green variable (not in docs)

**Missing From Code**
- Nothing. All documented design tokens exist.

**Regressions**
- Old repo hardcoded hex in toast styles: Needs code verification, but new repo has token system.
- Old repo hardcoded hex in export modals: Export flows exist as new components; likely rebuilt with tokens.
- Old repo inline style blocks: TipTap styles are in index.css (centralized, not inline). Improvement confirmed.
- 3 remaining violations from docs (text-[13px], text-[11px], text-blue-600/text-green-600): **Status unknown from Inside-Out audit.** Need code verification.


## Topic 09: Permissions and RBAC

**Agreements**
- Four roles: Owner > Admin > Editor > Viewer
- "member" legacy alias maps to "editor" via normalizeRole()
- Permission hook: useHasPermission with API permissions primary, role fallback secondary
- useFeatureCheck stub (always true)
- Permission strings: workspace.manage, folder.view/create/edit/delete, list.create/edit/delete, data.export, credits.allocate
- Owner-only: credits.allocate, billing, workspace creation (Pro plan)
- No route-level permission guards (only component-level)
- Three plan tiers with feature gating

**Divergences**
- Outside-In says UserRole type includes "member." Inside-Out found the type is `"owner" | "admin" | "editor" | "viewer"` with member handled by normalizeRole().
  - Code is correct. The type itself does not include "member"; normalization handles it.

- Outside-In lists 6 settings tabs visible to various roles. Inside-Out found 8 tabs in SETTINGS_TAB_ROLES including AIAgents, MCP, and Partners.
  - Code has more tabs than docs describe.
  - Recommended action: Document all 8 settings tab visibility rules.

- Outside-In says AppSidebar is missing folder.view and other permission gates. Inside-Out found AppSidebar DOES check folder.view, folder.create, folder.edit, folder.delete.
  - Code is correct. The RBAC audit doc (Outside-In source) may be outdated; gates have been implemented.
  - Recommended action: Re-audit sidebar permission gates against current code.

**Missing From Docs**
- tutorial.manage permission (owner + admin only)
- Settings tab visibility matrix (8 tabs with role whitelist)
- Role normalization: unknown/null roles default to editor (permissive default concern)
- Workspace role comes from workspace object, not user object
- Assignable roles limited to admin, editor, viewer (owner excluded)

**Missing From Code**
- Resource sharing endpoints (old repo had /workspaces/:id/shares/*)
- Leave workspace endpoint (POST /workspaces/:id/leave)
- PrivateRoute with requiredPermission (old repo pattern, not rebuilt)

**Regressions**
- PrivateRoute with permission-gated routing: **Confirmed not rebuilt.** Component-level checks are the only enforcement. This is less secure than route-level gating.
- Layer 2 Supabase app roles: Confirmed removed (improvement - no dual system).
- Resource sharing: **Confirmed not rebuilt.** No sharing endpoints or UI found.
- Viewer access to billing tab: **New concern.** Viewers can see billing/usage tabs, potentially exposing cost data.


## Topic 10: Dev Workflow

**Agreements**
- Bun as package manager
- Vite 5 for dev server and builds
- Feature branch workflow (feature/, fix/, spike/ prefixes)
- Squash merge to main
- Lovable compatibility as constraint
- Proxy-based development against real backend services
- Port feature pipeline with multi-agent approach
- Decision journal in docs/current-repo/decisions/
- Working docs system (API endpoint map, styling decisions, troubleshooting)

**Divergences**
- Outside-In says "No testing framework or test files documented." Inside-Out found Vitest + @testing-library/react configured with test scripts in package.json, but **no actual test files exist**.
  - Both are partly correct. Framework is configured; tests are not written.
  - Recommended action: Document that testing infrastructure exists but no tests are written.

- Outside-In says "No linting configuration documented." Inside-Out found ESLint v9 flat config with TypeScript + React hooks rules, but @typescript-eslint/no-unused-vars is disabled.
  - Inside-Out is correct. ESLint exists but with weak rules.
  - Recommended action: Document linting config and its limitations.

- Outside-In says "No CI/CD pipeline documented." Inside-Out confirms no .github/ directory, no CI configuration at all.
  - Both agree. No CI/CD exists.

**Missing From Docs**
- Dev server runs on port 8080 (host ::)
- HMR overlay is disabled
- build:dev script for development-mode builds
- ESLint flat config details (v9, what's enabled/disabled)
- Vitest config (jsdom environment, globals, setup file)
- TypeScript strictness settings (noImplicitAny: false, strictNullChecks: false)
- Environment file structure (.env, .env.development, .env.production)
- All VITE_* environment variable names
- Dev branches: dev, staging, production remote branches exist
- _redirects file in public/ (deployment routing)
- Static assets in public/ (logos, sample CSV, favicon)
- lovable-tagger dev plugin details
- SWC compiler usage (not Babel)

**Missing From Code**
- Pre-commit checks (bun run build before commit) documented but no husky/lint-staged enforcement
- Color audit check documented but no automated enforcement

**Regressions**
- highlight.run (error tracking): **Confirmed not replaced.** No error tracking in new app.
- PostHog (analytics): **Confirmed not replaced.** No product analytics in new app.
- Console.log in production: No lint rule prevents it (no-console not configured).
- No lazy loading in old repo was a known issue: **Fixed.** All routes lazy-loaded.
- No React Query in old repo: **Fixed.** React Query throughout.


---


## Cross-Cutting Findings

### Critical Issues

1. **No Error Tracking** - highlight.run was in the old repo. Nothing replaced it. Production errors are invisible unless a user reports them. This affects all topics.

2. **No Product Analytics** - PostHog was in the old repo. Nothing replaced it. No visibility into feature usage, conversion, or retention. Critical for a PLG product.

3. **No CI/CD Pipeline** - No automated build verification, no automated testing, no automated linting on push. Every deployment relies on manual checks.

4. **No Tests Written** - Vitest is configured but zero test files exist. For a 326-file codebase with enterprise grid, real-time enrichment, and multi-platform integrations, this is significant risk.

5. **Frontend-Only Plan Enforcement** - Plan capabilities (template access, integration limits, team size) are hardcoded in frontend. No backend enforcement. MCP callers bypass all restrictions. A motivated user could bypass plan limits via API.

6. **No Workspace Authorization Middleware** - No X-Workspace-Id header validation on the backend. Cross-workspace data access may be possible via direct API calls.

7. **Loose TypeScript** - strictNullChecks: false, noImplicitAny: false. Potential null reference bugs are hidden at compile time.

### Documentation Debt

8. **Stale Route Count** - CLAUDE.md says 14 routes; actual count is 35. This mismatch could mislead agents and developers.

9. **Many Features Undocumented** - Salesforce integration, Microsoft OAuth, deduplication, email validation, normalization, onboarding, tutorials, pixels/tags, and 20+ routes have no documentation.

10. **Socket Event Registry Incomplete** - Typed SocketEventMap covers only enrichment events. Template view, validation, normalization, and import events use untyped onAny handlers.

### Architecture Concerns

11. **Dual Toast Systems** - Sonner (preferred) and Radix toast run in parallel. Migration plan exists but is incomplete.

12. **services.ts Monolith** - 1400+ lines, 14 service objects in a single file. Feature-level services (live-audiences, templates) show the better pattern.

13. **Deprecated Code Not Removed** - audiencesService is fully deprecated (all methods throw), but still exported with React Query hooks wrapping it. ProtectedRoute.tsx exists but is unused. Index.tsx page exists but is not routed.

14. **Export Flow Duplication** - 9 separate export flow components likely share significant code (account selection, field mapping, confirmation patterns).

15. **No Optimistic Updates** - Despite React Query supporting them, no mutations use optimistic updates. This means every mutation shows loading state even for likely-successful operations.
