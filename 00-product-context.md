---
title: Product Context
last-updated: 2026-04-02
primary-audience: all
source: consensus (audit/outside-in + audit/inside-out)
---

# Product Context

## What Is Audience Loop

Audience Loop (ALoop) is a **B2B audience data management and enrichment platform** built by iCustomer. It is a multi-tenant SaaS product following a product-led growth (PLG) model. Users upload or create contact/company lists, enrich them using AI-powered data agents, and export enriched audiences to advertising platforms, outreach tools, CRMs, and spreadsheets.

The product is a rewrite of an older frontend codebase called "Precise." Multiple code comments reference "old FE," "old app," and "dual-FE support," and the `/auth` redirect component maps old-FE partner links to new-FE paths (`src/App.tsx:22-36`).


## Core Data Model

### Hierarchy

```
Workspace (isolation boundary)
  -> Folder (organizational container, called "project" in the API)
    -> List (dataset - rows of contact/company data in AG Grid)
      -> Columns (typed fields: text, email, phone, url, number, boolean, date, paragraph)
      -> Template Mappings (activation overlays that add enrichment capability)
```

### Key Entities

| Entity | Description | Code Reference |
|--------|-------------|----------------|
| **Workspace** | Top-level isolation boundary. Every resource belongs to exactly one workspace. Users access workspaces through membership. | `src/contexts/WorkspaceContext.tsx` |
| **Folder** | Organizational container for lists. Called "project" in API endpoints. | `src/api/services.ts` (datasetsService) |
| **List** | The core data unit. Spreadsheet-like rows in AG Grid Enterprise. | `src/pages/ListViewPage.tsx` |
| **Template** | Defines column structure and enrichment capabilities. 47 templates across 18 categories. | `src/features/templates/data/templates.ts` |
| **Template Mapping** | Links a template to a list, mapping template fields to list columns. | `src/features/templates/services/templateMappingService.ts` |
| **Live Audience** | An uploaded audience segment on an ad platform with campaign attachment. | `src/features/live-audiences/services.ts` |
| **Connected Account** | An OAuth or API-key-based connection to an external service. | `src/features/integrations/providers/registry.ts` |


## Template System

Templates are the primary product differentiator. They are activation overlays, not standalone entities. A list can have multiple template activations. Templates are organized by channel, source, and workflow.

**Template categories** (`src/features/templates/types.ts:38-57`):
prospecting, enrichment, outreach, analytics, management, lookalike, events, community, newsletter, social, retargeting, content, conversion, product, abm, suppression, seed-audiences, segmentation

**Template-driven workflow:**
1. User picks a template from the gallery (e.g., LinkedIn Prospecting, Contact Enrichment)
2. Template maps fields to list columns via AI field matching (three-tier: n8n AI agent, pre-defined mappings, heuristic fallback)
3. User enriches data (credits consumed per row)
4. User exports to destination platform

**Template count:** 47 marketing templates (`src/features/templates/data/templates.ts`)


## Data Entry Sources

| Source | Component | Status |
|--------|-----------|--------|
| CSV Upload | `UploadCSVModal` | Fully wired |
| Google Sheets | `GoogleSheetsImportModal` | Wired with sync scheduling |
| HubSpot | `HubSpotImportModal` | Background job import |
| Salesforce | `SalesforceImportWizard` | Multi-step wizard via Polytomic |
| Apollo/ICP Prospecting | `ICPProspectingModal` | Company and people search |
| Klaviyo | `KlaviyoImportSheet` | Side sheet import |
| Webhook | `WebhookModal` | Creates webhook + list |


## Export Destinations

Nine export flows exist as lazy-loaded components in the grid's action drawer (`src/features/grid/components/actions/flows/`):

| Destination | Flow File |
|-------------|-----------|
| CSV Download | download-csv.tsx |
| Google Sheets | google-sheets-export.tsx |
| Google Ads | google-ads-export.tsx |
| LinkedIn Ads | linkedin-ads.tsx |
| Meta Ads | meta-ads-export.tsx |
| HubSpot | hubspot-export.tsx |
| SmartLeads | smartleads-export.tsx |
| Reddit Ads | reddit-ads-export.tsx |
| Mailshake | mailshake-export.tsx |


## Enrichment Engine

- Per-row and batch enrichment via agent prompts
- Fill-column formula capability
- Enrichment job tracking with Socket.IO real-time progress
- Credit-based billing (credits consumed per enrichment operation)
- Singleton `cellStatusTracker` manages per-cell enrichment status outside React for performance


## Data Quality Operations

| Operation | Description | Status |
|-----------|-------------|--------|
| Email Validation | ZeroBounce integration | Backend wired, UI components exist |
| Normalization | Contact and account entity types | Wired with socket progress |
| Deduplication | Find duplicates by criteria field, merge with field-level control | Drawer UI with dedicated endpoints |


## User Types

| Type | Email Domain | Signup Behavior | Roles Available |
|------|-------------|----------------|-----------------|
| Full User | Business domain | Own workspace created on signup | Owner, Admin, Editor, Viewer |
| Guest Viewer | Personal domain (gmail, yahoo, etc.) | Dormant personal workspace, read-only | Viewer only (Pro plan invitation required) |


## Pricing and Credits

| Plan | Price | Credits | Team Size |
|------|-------|---------|-----------|
| Free | $0 | 200 | 1 |
| Starter | $99 | 2,000 | 3 |
| Professional | $299 | 7,000 | 10 |

> INVESTIGATE: Plan capabilities are hardcoded in the frontend (`src/hooks/use-feature-check.ts` always returns true). No backend endpoint enforces plan restrictions. MCP callers bypass all plan limits.


## Who Uses It

B2B marketers and sales teams who need to:
- Build targeted prospect lists from multiple sources
- Enrich contact data (emails, phones, company info, social profiles) using AI agents
- Validate and normalize contact information
- Deduplicate contact lists
- Export enriched audiences to advertising and outreach platforms
- Track audience performance across platforms (LinkedIn, Meta, SmartLeads analytics)


## MCP Access

The product supports Model Context Protocol (MCP) access with dedicated authentication pages (`/mcp-auth`, `/mcp-signup`). The API is designed to be self-describing enough for programmatic callers. MCP auth flow is separate from the standard Google/Microsoft OAuth flow.


## Sidebar Navigation

The app sidebar (`src/components/layout/AppSidebar.tsx:52-64`) shows:
Home, Templates, Live Audience, Folders, Integrations, Pixels & Tags, Refer & Earn, How To Use, Feedback


## Known Gaps

- No error tracking service (highlight.run was in old repo, not replaced)
- No product analytics service (PostHog was in old repo, not replaced)
- No support widget (Intercom was in old repo, not replaced)
- No competitive landscape documentation
- No user personas or journey maps
- No product roadmap beyond sprint-level planning
