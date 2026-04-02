---
title: UI Components
last-updated: 2026-04-02
primary-audience: frontend
source: consensus (audit/outside-in + audit/inside-out)
---

# UI Components

## Component Count Summary

| Category | Count | Location |
|----------|-------|----------|
| shadcn/ui primitives | 50 | `src/components/ui/` |
| Custom shared components | ~50 | `src/components/` (non-ui subfolders) |
| Feature components | ~80 | `src/features/*/components/` |
| Page components | 35 | `src/pages/` |
| **Total source files** | **326** | `src/` (.ts and .tsx) |


## shadcn/ui Primitives (50 files)

Location: `src/components/ui/`

accordion, alert-dialog, alert, aspect-ratio, avatar, badge, breadcrumb, button, calendar, card, carousel, chart, checkbox, collapsible, command, context-menu, dialog, drawer, dropdown-menu, form, hover-card, input-otp, input, label, menubar, multi-select (custom addition), navigation-menu, pagination, popover, progress, radio-group, resizable, scroll-area, select, separator, sheet, sidebar, skeleton, slider, sonner, switch, table, tabs, textarea, toast, toaster, toggle-group, toggle, tooltip, use-toast

Notable: `multi-select.tsx` is a custom addition not part of standard shadcn. `sidebar.tsx` exists as a shadcn primitive but the app uses a custom `AppSidebar.tsx` instead.


## Shared Components

### Auth (`src/components/auth/` - 6 files)

| Component | Purpose |
|-----------|---------|
| AuthLogo.tsx | Brand logo for auth pages |
| AuthMarketingPanel.tsx | Marketing content on auth page sidebar |
| GoogleOAuthButton.tsx | Google native sign-in with business email domain blocking |
| MicrosoftOAuthButton.tsx | Microsoft OAuth login button |
| InfinityLoop.tsx | Custom branded loading animation |
| ProtectedRoute.tsx | Auth guard component (**exists but is NOT used** - auth protection is in AppLayout) |

### Common (`src/components/common/` - 10 files)

| Component | Purpose | Usage Rules |
|-----------|---------|-------------|
| StatusBadge.tsx | Single source of truth for all status display. Supported: active, ready, completed, connected, inactive, disconnected, pending, processing, paused, running, syncing, building, error, failed. Input is `.toLowerCase()` normalized. Unknown falls back to inactive. | Never use raw `<Badge>` for status values. |
| PageHeader.tsx | Shared page title/description/actions pattern. Used on every page. | |
| EmptyState.tsx | Empty state with optional icon, title, description, action. Accepts className. | |
| InfinityLoader.tsx | Fullscreen/overlay loading indicator. Uses react-loader-spinner InfinitySpin. Default width=200, color="#2966BC". | Used for non-table loading (workspace switching, page transitions). |
| MosaicGrid.tsx | Responsive grid layout component | |
| PlatformLogo.tsx | Renders integration/platform logos | |
| RoleBadge.tsx | Displays user role badges | |
| TemplateLogoRenderer.tsx | Renders template-specific logos | |
| WorkspaceSwitchOverlay.tsx | Overlay during workspace transitions | |

### Credits (`src/components/credits/` - 2 components)

| Component | Purpose |
|-----------|---------|
| CreditsWidget.tsx | Credit balance display widget |
| OverageConfirmModal.tsx | Credit overage confirmation dialog |

### Folder (`src/components/folder/` - 6 files)

| Component | Purpose |
|-----------|---------|
| UploadCSVModal.tsx | CSV file upload dialog |
| GoogleSheetsImportModal.tsx | Google Sheets import with account/spreadsheet selection |
| HubSpotImportModal.tsx | HubSpot import configuration |
| SalesforceImportWizard.tsx | Multi-step Salesforce import via Polytomic adapter |
| LookalikesModal.tsx | Lookalike audience configuration |
| WebhookModal.tsx | Webhook setup (creates webhook + list) |

### Layout (`src/components/layout/` - 5 files)

| Component | Purpose |
|-----------|---------|
| AppLayout.tsx | App shell wrapper. Contains auth check (redirects to /login if unauthenticated). | 
| AppSidebar.tsx | Collapsible sidebar with workspace switcher, navigation items, folder tree. Permission-gated items. |
| TopBar.tsx | Header bar with search, notifications, user profile |
| CreditsPopover.tsx | Credit balance popover display |
| SidebarContext.tsx | Sidebar collapse state provider |

### List (`src/components/list/` - 8 files)

| Component | Purpose |
|-----------|---------|
| EventCellDetailsPanel.tsx | Event data detail panel |
| GoogleSheetsSyncButton.tsx | Trigger Google Sheets sync |
| GoogleSheetsSyncModal.tsx | Google Sheets sync configuration |
| ICPProspectingModal.tsx | Apollo ICP prospecting interface |
| KlaviyoImportSheet.tsx | Klaviyo data import side sheet |
| SalesforceSyncModal.tsx | Salesforce sync configuration |
| SourceWebhookSheet.tsx | Webhook source configuration |
| WebSearchAgentModal.tsx | Web search agent interface |

### Settings (`src/components/settings/` - 9 files)

| Component | Visible To (roles) |
|-----------|-------------------|
| ProfileTab.tsx | owner, admin, editor, viewer |
| WorkspaceTab.tsx | owner, admin, editor, viewer |
| AIAgentsTab.tsx | owner, admin, editor |
| UsageTab.tsx | owner, admin, editor, viewer |
| BillingPlansTab.tsx | owner, admin, editor, viewer |
| MCPServerTab.tsx | owner, admin |
| APIKeysTab.tsx | owner, admin, editor |
| InvitePartnersTab.tsx | owner, admin, editor |
| TeamManagementTab.tsx | (within WorkspaceTab) |

Role-to-tab visibility is defined in `src/lib/permissions.ts:75-84`.

### Workspace (`src/components/workspace/` - 2 files)

| Component | Purpose |
|-----------|---------|
| WorkspaceSelector.tsx | Controlled dropdown for workspace switching. Splits owned vs shared workspaces. Role badges. Enforces Professional plan gating for new workspace creation. |
| CreateWorkspaceModal.tsx | New workspace creation dialog |

### Onboarding (`src/components/onboarding/` - 1 file)

| Component | Purpose |
|-----------|---------|
| GettingStartedCard.tsx | Onboarding checklist card on home page |


## Feature Components

### Grid (`src/features/grid/components/` - 30+ files)

The grid module is the largest feature module. Components organized into:

**Toolbar and Actions:**
- GridToolbar.tsx - toolbar above the grid
- ActionsPanel.tsx - side panel for export actions
- FilterDropdown.tsx - column filtering
- SortDropdown.tsx - column sorting

**Enrichment UI:**
- EnrichmentDropdown.tsx - scope selection for enrichment
- EnrichmentConfirmDialog.tsx - credit confirmation before enrichment
- EnrichmentProgressBar.tsx - progress during active enrichment
- EnrichableHeader.tsx - column header with enrichment indicators
- EnrichmentCellRenderer.tsx - custom cell renderer reading from singleton tracker

**Column Operations:**
- AddColumnButton.tsx - add new column
- AddColumnPopover.tsx - column type selection
- EditColumnPopover.tsx - column editing
- InsertColumnDialog.tsx - insert column at position
- TiptapColumnEditor.tsx - rich text formula/prompt editor (uses TipTap)
- UpdateColumnPromptPopover.tsx - AI prompt update for columns

**Row Operations:**
- AddRowsFooter.tsx - add rows at bottom of grid

**Data Quality:**
- DeduplicationDrawerContent.tsx - deduplication configuration drawer
- DeduplicationMergeDialog.tsx - merge duplicate records
- NormalizationDrawerContent.tsx - normalization configuration
- NormalizationFieldMapper.tsx - field mapping for normalization
- ValidationDrawerContent.tsx - email validation drawer
- EmailValidationCellRenderer.tsx - validation status in cells

**Specialized Renderers:**
- EventCellRenderer.tsx - event data cells
- EventColumnHeader.tsx - event column headers
- KlaviyoMetaCellRenderer.tsx - Klaviyo metadata cells
- SyncStatusCellRenderer.tsx - sync status display

**Audience Tracking:**
- AudienceTrackingPanel.tsx - audience tracking configuration

**Export Flows** (`actions/flows/` - 9 files):
download-csv, google-sheets-export, google-ads-export, linkedin-ads, meta-ads-export, hubspot-export, smartleads-export, reddit-ads-export, mailshake-export

### Templates (`src/features/templates/components/` - 12 files)

| Component | Purpose |
|-----------|---------|
| TemplateCard.tsx | Gallery card for template selection |
| TemplateCardSkeleton.tsx | Loading skeleton for template cards |
| TemplatePreviewModal.tsx | Detailed template preview with CTA buttons |
| ListPickerModal.tsx | Folder/list browser for "Use my List" flow |
| TemplateProgressOverlay.tsx | 4-step progress (Select, Creating List, Auto-Mapping, Loading Grid) |
| TemplateMappingPanel.tsx | Side panel for manual mapping adjustments |
| TemplateAgentGrid.tsx | Grid display for template agent results |
| AlphaTemplateModal.tsx | Alpha template (ICP/Lookalike) modal |
| IcpForm.tsx | ICP prospecting form within template flow |
| LookalikeForm.tsx | Lookalike audience form |
| LookalikePreviewModal.tsx | Preview lookalike results |
| LookalikeImportProgressModal.tsx | Import progress for lookalike data |

### Live Audiences (`src/features/live-audiences/` - 6 files)

| Component | Purpose |
|-----------|---------|
| AudienceTable.tsx | Table of uploaded audiences |
| SummaryStats.tsx | Audience summary statistics |
| AudienceActionsDropdown.tsx | Per-audience action menu |
| AddToCampaignModal.tsx | Attach LinkedIn audience to campaign |
| AddToMetaCampaignModal.tsx | Attach Meta audience to ad set |
| AttachedCampaignsModal.tsx | View attached campaigns |

### Integrations (`src/features/integrations/` - 4 files)

| Component | Purpose |
|-----------|---------|
| IntegrationDetailsDialog.tsx | Integration detail view |
| IntegrationCredentialsDialog.tsx | API key entry dialog |
| IntegrationCardSkeleton.tsx | Loading skeleton for integration cards |
| DeleteConfirmDialog.tsx | Confirm integration disconnection |

### Notifications (`src/features/notifications/` - 5 files)

| Component | Purpose |
|-----------|---------|
| NotificationBell.tsx | Bell icon with unread badge count + popover trigger |
| NotificationPopover.tsx | Popover with enrichment progress cards + Novu notifications |
| EnrichmentProgressCard.tsx | Active enrichment job progress display |
| NovuNotificationItem.tsx | Single Novu notification display |
| NovuProviderWrapper.tsx | Wraps authenticated routes in Novu provider |

### Batch Enrichment (`src/features/batch-enrichment/` - 1 file)

| Component | Purpose |
|-----------|---------|
| BatchEnrichModal.tsx | Batch enrichment configuration and execution modal |


## Card Family System

Four card families with consistent visual rules:

| Family | Use | Key Variants |
|--------|-----|-------------|
| 1 - Action Card | Navigable/clickable cards | Full (Quick Actions), Compact (New List From), Data (Recently Used), Info (Pixel features) |
| 2 - Stat Card | Metric display (icon + title + number) | Featured (24px number), Compact (22px number) |
| 3 - Connect Card | Integration cards with CTA button | 44px icon, 1-line title, 2-line desc, pill + button footer |
| 4 - Gallery Card | Template cards (unique) | rounded-3xl, decorative circles, field tags |

**Shadow rules:** Clickable cards get `shadow-sm` at rest + larger on hover. Non-clickable cards get no shadow. Shadow communicates interactivity.

**Border rules:** All borders 1px, never 2px. Full Action Cards on gradient bg: transparent at rest, colored on hover. Compact on white bg: 40% opacity at rest, 100% on hover.


## Loading Patterns

Two distinct patterns:
1. **Data Skeleton** - shown while API data loads. Skeleton IS the loading state. Shimmer cards matching expected layout. Unmounts when data arrives.
2. **Embed Overlay Skeleton** - cosmetic cover over third-party embeds (iframes). Real content mounted underneath. Fades out after iframe `onLoad`.

Skeleton components: `IntegrationCardSkeleton.tsx`, `TemplateCardSkeleton.tsx`


## Naming Conventions

| Suffix | Meaning |
|--------|---------|
| `*Modal.tsx` | Dialog/modal components |
| `*Sheet.tsx` | Side sheet components |
| `*Drawer*.tsx` | Drawer components |
| `*CellRenderer.tsx` | AG Grid custom cell rendering |
| `*Skeleton.tsx` | Loading state skeletons |
| `*Tab.tsx` | Settings page tab components |
| `*Page.tsx` | Route-level page components |


## Known Concerns

- `ProtectedRoute.tsx` exists but is not used anywhere in routing. Auth protection is in `AppLayout.tsx` via `useAuth()` check.
- `src/pages/Index.tsx` exists but is never referenced in any route (dead file).
- The grid feature has 30+ components and may benefit from further sub-modularization.
- 9 export flow components likely share significant code (each handles account selection, field mapping, confirmation).
- 3 analytics pages (LinkedIn, Meta, SmartLead) exist as separate components with likely 80%+ shared logic.
