---
title: Integrations
last-updated: 2026-04-02
primary-audience: all
source: consensus (audit/outside-in + audit/inside-out)
---

# Integrations

## Integration Registry

The app supports 12 named integrations organized into three connection types, plus 2 coming-soon providers.

Reference: `src/features/integrations/providers/registry.ts:1-117`


## OAuth Providers (6)

Redirect-based authentication flow.

| Integration | Provider Slug | Airbyte Setup | Onboarding Task | Notes |
|-------------|---------------|---------------|-----------------|-------|
| LinkedIn Ads | linkedin-ads | Yes | connect_linkedin | Pre-logout required before OAuth redirect |
| Meta Ads | meta-ads | Yes | - | Facebook/Instagram ads |
| Reddit Ads | reddit-ads | No | - | - |
| Google Ads | google-ads | No | - | Shows Google "unverified app" warning |
| Google Sheets | google-sheets | No | - | Import and export; shows "unverified app" warning |
| HubSpot | hubspot | No | connect_hubspot | - |

### OAuth Flow

1. Frontend builds OAuth URL: `{GATEWAY_BASE_URL}/api/integrations/{provider}/auth?userId=X&workspaceId=Y`
2. User redirected to provider's consent screen in popup/new tab
3. Callback returns to gateway, which stores tokens
4. Frontend detects completion via dual-layer detection:
   - **Primary**: `message` event listener for instant detection via `postMessage`
   - **Fallback**: Polling every 3 seconds for new connected account
   - Both cleaned up on success/timeout/unmount
5. For LinkedIn and Meta: additional Airbyte connection setup call (`POST /api/integrations/{provider}/connection/setup`)
6. If onboarding task exists: fire-and-forget `POST /onboarding/complete-task` (errors logged, not surfaced)

Reference: `src/features/integrations/utils/oauthPolling.ts`


## API Key Providers (4)

Key-based authentication with validate-then-connect pattern.

| Integration | Provider Slug | Notes |
|-------------|---------------|-------|
| SmartLead | smartleads | Email outreach platform |
| Apollo | apollo | B2B prospecting data; has "system" provider concept with cached provider ID |
| Klaviyo | klaviyo | Email marketing |
| Mailshake | mailshake | Sales engagement |

### API Key Flow

1. User enters API key in `IntegrationCredentialsDialog`
2. Frontend validates: `POST /api/integrations/{provider}/validate-key`
3. On success: `POST /api/integrations/{provider}/connect`
4. Account appears in connected accounts list
5. Cache invalidation triggers UI refresh

Reference: `src/features/integrations/utils/apiKeyFlow.ts`

### Apollo System Provider

Apollo has a "system" provider concept where a system-managed API key is auto-resolved. A `_systemProviderIdCache` in `prospectingService.ts` caches the Apollo system provider ID for the session. The UI shows "Onesource" as a display alias for the system-managed Apollo.


## Connection Providers (1 active, 2 coming soon)

Polytomic adapter-based connections.

| Integration | Provider Slug | Status | Notes |
|-------------|---------------|--------|-------|
| Salesforce | salesforce | Active | Dedicated import wizard, separate service layer |
| Instantly | instantly | Coming soon | Listed in connectionProviders.ts, no registry entry |
| Marketo | marketo | Coming soon | Listed in connectionProviders.ts, no registry entry |

### Salesforce Connection Flow

1. Uses Polytomic adapter pattern (separate from main OAuth and API key flows)
2. Frontend calls `/connections/*` endpoints
3. Dedicated `SalesforceImportWizard` component handles multi-step import
4. Separate services: `salesforceDataService` and `connectionsService`

Reference: `src/config/connectionProviders.ts:1-14`, `src/features/integrations/utils/polytomicOAuth.ts`


## Connected Account Management

| Method | Path | Purpose |
|--------|------|---------|
| GET | /api/integrations/connected-accounts | List all connections |
| DELETE | /api/integrations/connected-accounts/:id | Disconnect account |
| PUT | /api/integrations/connected-accounts/:id | Edit display name |
| GET | /api/integrations/connected-accounts/by-provider | Filter by provider |

Integration connections are workspace-scoped via query key `["integrations", workspaceId]`.

Backend naming inconsistencies handled by `INTEGRATION_NAME_OVERRIDES`: "reddit" -> "Reddit", "smartleads" -> "Smartlead".


## Export Flows (9 Destinations)

All export flows are lazy-loaded components in the grid's action drawer (`src/features/grid/components/actions/flows/`).

| Flow | File | Destination | Account Required |
|------|------|-------------|-----------------|
| CSV Download | download-csv.tsx | Local file | No |
| Google Sheets | google-sheets-export.tsx | Google Sheets | Google Sheets OAuth |
| Google Ads | google-ads-export.tsx | Google Ads audiences | Google Ads OAuth |
| LinkedIn Ads | linkedin-ads.tsx | LinkedIn Ads audiences | LinkedIn OAuth |
| Meta Ads | meta-ads-export.tsx | Meta Ads audiences | Meta OAuth |
| HubSpot | hubspot-export.tsx | HubSpot CRM | HubSpot OAuth |
| SmartLeads | smartleads-export.tsx | SmartLead campaigns | SmartLead API key |
| Reddit Ads | reddit-ads-export.tsx | Reddit Ads audiences | Reddit OAuth |
| Mailshake | mailshake-export.tsx | Mailshake campaigns | Mailshake API key |

Sync status write-back: all flows write to `POST /lists/{listId}/meta-columns` after successful sync.

> INVESTIGATE: Export flows are 9 separate components with likely significant code duplication (each handles account selection, field mapping, confirmation). A shared abstraction could reduce maintenance burden.


## Import Sources (7)

| Source | Component | Service Method | Status |
|--------|-----------|---------------|--------|
| CSV Upload | UploadCSVModal | datasetsService.uploadCSVFile | Fully wired (multipart form) |
| Google Sheets | GoogleSheetsImportModal | datasetsService.createListFromGoogleSheets | Wired with sync scheduling |
| HubSpot | HubSpotImportModal | datasetsService.startHubSpotImport | Background job |
| Salesforce | SalesforceImportWizard | salesforceDataService | Multi-step wizard |
| Apollo/ICP | ICPProspectingModal | prospectingService | Company and people search |
| Klaviyo | KlaviyoImportSheet | (implied) | Side sheet import |
| Webhook | WebhookModal | datasetsService.createWebhookWithTable | Creates webhook + list |


## Live Audiences (Platform Sync)

| Platform | Capabilities | Service |
|----------|-------------|---------|
| LinkedIn Ads | Upload audiences, sync, attach to campaigns | liveAudiencesService |
| Meta Ads | Upload audiences, sync, attach to ad sets | liveAudiencesService |

The old `audiencesService` in `src/api/services.ts` is fully deprecated (all methods throw errors). The replacement is `src/features/live-audiences/services.ts`, which only covers LinkedIn and Meta.

Reference: `src/features/live-audiences/services.ts:1-253`


## External AI Service

`POST https://n8n.icustomer.ai/webhook/field-mapping-agent` - external n8n webhook for AI-powered field mapping.

Three-tier matching:
1. AI Agent (n8n webhook) - primary
2. Pre-defined mappings (for known sample datasets) - fallback
3. Heuristic fallback (normalized substring matching) - last resort


## Third-Party Services (Non-Integration)

| Service | Connection Method | Usage |
|---------|------------------|-------|
| Google OAuth | @react-oauth/google SDK | User authentication |
| Microsoft OAuth | Custom flow via /auth/microsoft/complete | User authentication |
| Novu | @novu/react SDK (@3.10.1) | Push notifications (custom UI, not pre-built Inbox) |
| Stripe | Redirect-based checkout | Payments and billing |
| ZeroBounce | Backend-only (validation/credits-status) | Email validation |
| Socket.IO | socket.io-client | Real-time updates |


## Notification Integration (Novu)

Provider: `NovuProviderWrapper` with:
- `applicationIdentifier` from `VITE_NOVU_APPLICATION_IDENTIFIER` env var
- `subscriberId`: `audience-loop-{email}`

Custom UI (not pre-built Inbox component) because of hybrid notification system: Novu handles general notifications while enrichment progress comes via WebSocket. Enrichment workflow IDs are filtered out of the Novu notification list.


## Integration Categories (UI Display)

| Category | Integrations |
|----------|-------------|
| Marketing & Ad Platforms | LinkedIn Ads, Meta Ads, Reddit Ads, Google Ads |
| CRM & Customer Data Platforms | HubSpot, Salesforce |
| Databases & Data Sources | Google Sheets |
| Outreach | SmartLead, Mailshake |
| Data Enrichment | Apollo, Klaviyo |


## What Is Missing (From Old Repo)

| Integration | Old Repo Status | New Repo Status |
|-------------|----------------|-----------------|
| Intercom | Support widget | **Not replaced** |
| PostHog | Analytics tracking | **Not replaced** |
| highlight.run | Error tracking | **Not replaced** |

These three services provided customer support, product analytics, and error tracking respectively. None have been replaced in the new codebase.
