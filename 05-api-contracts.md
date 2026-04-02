---
title: API Contracts
last-updated: 2026-04-02
primary-audience: all
source: consensus (audit/outside-in + audit/inside-out)
---

# API Contracts

## Service Topology

| Service | Base URL | Vite Dev Proxy | Scope |
|---------|----------|----------------|-------|
| **list-service** (apiClient) | `VITE_DATA_LIST_SERVICE_BASE_URL` / list-service.icustomer.ai/api/v1 | `/api/v1` | Auth, datasets, lists, data, columns, enrichment, validation, normalization, deduplication, payments, webhooks, onboarding, referrals, partners, API keys, tutorials, pixels |
| **gateway** (gatewayClient) | `VITE_GATEWAY_SERVICE_BASE_URL` / prod-aloop-api-gateway.icustomer.ai | `/gateway` (path rewrite strips /gateway) | Integration OAuth, connected accounts, export flows, audience sync, analytics |

Reference: `src/api/client.ts`, `vite.config.ts:8-30`


## API Client Architecture

Single `ApiClient` class (`src/api/client.ts:34-277`) with:
- fetch-based HTTP (no Axios)
- Request interceptors injecting `workspace-id` and `user-id` headers
- Token refresh on 401 + `TOKEN_EXPIRED` code with automatic retry
- `skipAuth: true` option for pre-auth calls (login, register, validate-email)
- `x-client-origin` header for dual-FE awareness

Two instances: `apiClient` (list-service) and `gatewayClient` (gateway). Both share token state via `apiClient.setToken()`.


## Response Convention

All responses follow:
```typescript
// Success
{ success: true, data: T, pagination?: { page: number, limit: number, total: number } }

// Error
{ success: false, error: string }
```

Response unwrapping pattern: `(res as any)?.data ?? res`
Gateway double unwrap: `response?.data?.data ?? response?.data`

> INVESTIGATE: Gateway response unwrapping is inconsistent across callsites. Some use `unwrapGatewayResponse<T>()`, others do inline unwrapping.


## Auth Endpoints (apiClient)

| Method | Path | Service | Purpose |
|--------|------|---------|---------|
| POST | /auth/login | authService.login | Email/password login |
| POST | /auth/register | authService.register | New user signup |
| POST | /auth/google | authService.googleAuth | Google OAuth login |
| POST | /auth/logout | authService.logout | Logout |
| GET | /auth/me | AuthContext | Validate session, get user |
| POST | /auth/refresh | ApiClient.tryRefreshToken | Token refresh (cookie-based) |
| POST | /auth/validate-email | authService.validateEmail | Check email validity |
| POST | /auth/send-otp | authService.sendOtp | Send OTP code |
| POST | /auth/verify-otp | authService.verifyOtp | Verify OTP code |
| POST | /auth/resend-otp | authService.resendOtp | Resend OTP |
| GET | /auth/otp-status/:userId | authService.getOtpStatus | OTP status check |
| POST | /auth/forgot-password | authService.forgotPassword | Request password reset |
| POST | /auth/verify-password-reset-otp | authService.verifyPasswordResetOtp | Verify reset OTP |
| POST | /auth/reset-password | authService.resetPassword | Set new password |
| GET | /auth/password-reset-otp-status/:emailHash | authService.getPasswordResetOtpStatus | Reset OTP status |
| POST | /auth/resend-password-reset-otp | authService.resendPasswordResetOtp | Resend reset OTP |
| POST | /auth/add-password | authService.addPassword | Add password to OAuth account |
| GET | /users/profile | authService.getProfile | Fetch user profile |
| PUT | /users/profile | authService.updateProfile | Update profile |


## Workspace Endpoints (apiClient)

| Method | Path | Service | Purpose |
|--------|------|---------|---------|
| GET | /workspaces | settingsService.listWorkspaces | List user workspaces |
| GET | /workspaces/:id | settingsService.getWorkspace | Get workspace details |
| POST | /workspaces | settingsService.createWorkspace | Create workspace |
| PUT | /workspaces/:id | settingsService.updateWorkspace | Update workspace |
| GET | /workspaces/:id/roles | settingsService.getWorkspaceRoles | Get role UUIDs |
| GET | /workspaces/:id/members | settingsService.listTeamMembers | List team members |
| POST | /workspaces/:id/invitations | settingsService.inviteMember | Invite member |
| PUT | /workspaces/:id/members/:userId/role | settingsService.updateMemberRole | Change role |
| DELETE | /workspaces/:id/members/:userId | settingsService.removeMember | Remove member |
| POST | /workspaces/:id/invitations/:invId/resend | invitationsService.resend | Resend invitation |
| DELETE | /workspaces/:id/invitations/:invId | invitationsService.cancel | Cancel invitation |


## Folder/Project Endpoints (apiClient)

| Method | Path | Service | Purpose |
|--------|------|---------|---------|
| GET | /workspaces/:wsId/projects | datasetsService.listFolders | List folders |
| GET | /workspaces/:wsId/projects/:id | datasetsService.getFolder | Get folder |
| POST | /workspaces/:wsId/projects | datasetsService.createFolder | Create folder |
| PUT | /workspaces/:wsId/projects/:id | datasetsService.renameFolder | Rename folder |
| DELETE | /workspaces/:wsId/projects/:id | datasetsService.deleteFolder | Delete folder |


## List Endpoints (apiClient)

| Method | Path | Service | Purpose |
|--------|------|---------|---------|
| GET | /:folderId/lists | datasetsService.listLists | List datasets in folder |
| POST | /:folderId/lists/empty | datasetsService.createList | Create empty list |
| POST | /:folderId/lists | datasetsService.uploadCSVFile | Upload CSV (multipart) |
| POST | /:projectId/lists/from-google-sheets | datasetsService.createListFromGoogleSheets | Import from Sheets |
| GET | /lists/:id | datasetsService.getList | Get list metadata |
| PUT | /lists/:id | datasetsService.renameList | Rename list |
| POST | /lists/:id/duplicate | datasetsService.duplicateList | Duplicate list |
| DELETE | /:projectId/lists/:id | datasetsService.deleteList | Delete list |
| GET | /lists/:id/data | datasetsService.getRawListColumns | Raw list data |
| GET | /lists/:id/original-data | datasetsService.getListData | Template-mapped data |
| POST | /lists/:id/data | datasetsService.insertRow | Insert empty row |
| PATCH | /lists/:id/records/:recordId | datasetsService.updateListData | Update cell value |
| POST | /lists/:id/records/bulk-delete | datasetsService.deleteRows | Bulk delete rows |
| POST | /lists/:id/add-rows | datasetsService.addRows | Add N empty rows |
| GET | /recently-used-sheets | datasetsService.getRecentlyUsedSheets | Recent lists |


## Enrichment Endpoints (apiClient)

| Method | Path | Service | Purpose |
|--------|------|---------|---------|
| POST | /lists/:id/enrich | enrichList | Start enrichment job |
| POST | /lists/:id/enrichment/estimate | estimateEnrichment | Estimate credit cost |
| GET | /lists/:id/enrichment/jobs | getEnrichmentJobs | List enrichment jobs |
| GET | /enrichment/jobs/active | getActiveJobs | Active jobs globally |
| GET | /enrichment/jobs/:jobId | getEnrichmentJob | Get job status |
| GET | /enrichment/jobs/:jobId/results | getEnrichmentJobResults | Get job results |
| POST | /enrichment/jobs/:jobId/cancel | cancelEnrichmentJob | Cancel job |
| DELETE | /enrichment/jobs/:jobId | deleteEnrichmentJob | Delete job record |
| POST | /lists/:id/fill-column-formula | fillColumnFormula | Fill column via formula |


## Validation and Normalization Endpoints (apiClient)

| Method | Path | Service | Purpose |
|--------|------|---------|---------|
| POST | /lists/:id/validate-emails | datasetsService.validateEmails | Start email validation |
| GET | /validation/credits-status | datasetsService.getValidationCreditsStatus | ZeroBounce credits |
| GET | /lists/:id/validation-stats | datasetsService.getValidationStats | Validation statistics |
| POST | /validation/jobs/:jobId/cancel | datasetsService.cancelValidationJob | Cancel validation |
| DELETE | /lists/:id/invalid-emails | datasetsService.deleteInvalidEmails | Remove invalid emails |
| POST | /lists/:id/normalize | datasetsService.normalizeData | Start normalization |
| POST | /normalization/jobs/:jobId/cancel | datasetsService.cancelNormalizationJob | Cancel normalization |


## Deduplication Endpoints (apiClient)

| Method | Path | Service | Purpose |
|--------|------|---------|---------|
| GET | /projects/:pid/lists/:lid/deduplication/stats | datasetsService.getDeduplicationStats | Dedup statistics |
| POST | /projects/:pid/lists/:lid/deduplication/find | datasetsService.findDuplicates | Find duplicate groups |
| POST | /projects/:pid/lists/:lid/deduplication/merge | datasetsService.mergeDeduplicatedRecords | Merge duplicates |


## Column Config Endpoints (apiClient)

| Method | Path | Service | Purpose |
|--------|------|---------|---------|
| POST | /lists/:id/mappings/:mid/columns | columnConfigService.create | Create column |
| GET | /lists/:id/mappings/:mid/columns | columnConfigService.getByListId | Get columns |
| PATCH | /lists/:id/mappings/:mid/columns/:cid | columnConfigService.update | Update column |
| DELETE | /lists/:id/mappings/:mid/columns/:cid | columnConfigService.delete | Delete column |
| PUT | /lists/:id/columns/:cid/change-type | columnConfigService.changeType | Change column type |
| POST | /lists/:id/columns/:cid/validate-conversion | columnConfigService.validateConversion | Validate type change |

Column types: `text | email | phone | url | number | boolean | date | paragraph`


## Template Endpoints (apiClient)

| Method | Path | Service | Purpose |
|--------|------|---------|---------|
| GET | /templates/registry | templateService.getRegistry | Template availability |
| GET | /lists/:id/template-mappings | templateService.getTemplateMappings | List mappings |
| POST | /lists/:id/template-mappings | templateService.createTemplateMapping | Create mapping |
| GET | /lists/:id/template-mappings/:mid | templateService.getTemplateMapping | Get mapping |
| PUT | /lists/:id/template-mappings/:mid | templateService.updateTemplateMapping | Update mapping |
| DELETE | /lists/:id/template-mappings/:mid | templateService.deleteTemplateMapping | Delete mapping |
| POST | /lists/:id/template-mappings/:mid/auto-map | templateService.autoMapTemplate | AI auto-mapping |
| POST | /lists/:id/template-mappings/:mid/field-mapping | templateService.getFieldMappingSuggestions | Field mapping AI |
| POST | /lists/:id/template-mappings/:mid/columns | templateMappingService.addTemplateColumn | Add template column |
| PUT | /lists/:id/template-mappings/:mid/columns/:fid | templateMappingService.editTemplateColumn | Edit column |
| DELETE | /lists/:id/template-mappings/:mid/columns/:fid | templateMappingService.deleteTemplateColumn | Delete column |


## Payment Endpoints (apiClient)

| Method | Path | Service | Purpose |
|--------|------|---------|---------|
| GET | /payments/subscription | paymentsService.getSubscription | Current subscription |
| GET | /payments/transactions-enriched | paymentsService.getEnrichedTransactions | Transaction history |
| GET | /payments/credits-breakdown | paymentsService.getCreditBreakdown | Credit breakdown |
| POST | /payments/checkout-session | paymentsService.createCheckoutSession | Stripe checkout |
| POST | /payments/create-portal-session | paymentsService.createPortalSession | Stripe billing portal |
| POST | /payments/preview-upgrade | paymentsService.previewUpgrade | Preview plan change |
| POST | /payments/upgrade-subscription | paymentsService.upgradeSubscription | Upgrade plan |
| POST | /payments/buy-credits | paymentsService.buyCredits | Purchase credits |
| POST | /payments/consume-credits | paymentsService.consumeCredits | Consume credits |
| POST | /payments/cancel-subscription | paymentsService.cancelSubscription | Cancel subscription |
| POST | /payments/enable-overage | paymentsService.enableOverage | Enable credit overage |


## Referral and Partner Endpoints (apiClient)

| Method | Path | Service | Purpose |
|--------|------|---------|---------|
| GET | /referrals/stats | referralsService.getStats | Referral statistics |
| GET | /referrals/my-code | referralsService.getMyCode | User referral code |
| GET | /referrals/my-referrals | referralsService.getUserReferrals | List of referrals |
| GET | /referrals/my-referrer | referralsService.getMyReferrer | Who referred user |
| POST | /referrals/track | referralsService.trackReferral | Record a referral |
| POST | /partner/invite | partnerService.invite | Invite partner |
| POST | /promo/generate-batch | partnerService.generateBatch | Generate coupon batch |
| GET | /promo/my-coupons | partnerService.listCoupons | List coupons |
| GET | /promo/stats | partnerService.getCouponStats | Coupon statistics |
| DELETE | /promo/revoke/:id | partnerService.revokeCoupon | Revoke coupon |
| POST | /promo/validate | partnerService.validatePromo | Validate coupon code |
| POST | /promo/apply-to-existing | partnerService.applyToExisting | Apply code to user |


## Other Endpoints (apiClient)

| Method | Path | Service | Purpose |
|--------|------|---------|---------|
| GET/POST/DELETE | /keys, /keys/:id, /keys/:id/revoke, /keys/:id/permanent | settingsService | API key CRUD |
| GET | /keys/workspace/:wsId/members | settingsService | Team key members |
| POST | /keys/admin/generate-for-user | settingsService | Admin generate key |
| GET | /onboarding/progress | onboardingService.getProgress | Onboarding checklist |
| POST | /onboarding/complete-task | onboardingService.completeTask | Mark task done |
| GET | /pixels/workspace/:wsId/status | pixelService.getPixelStatus | Pixel install status |
| GET | /tutorials/videos | tutorialsService.list | List video tutorials |
| GET | /tutorial-documents | tutorialDocumentsService.list | List documents |
| POST | /webhooks/projects/:pid/create-webhook-table | datasetsService | Create webhook + list |
| GET | /apollo/filter-options | prospectingService | ICP filter dropdowns |
| POST | /apollo/preview | prospectingService | Preview search results |
| POST | /apollo/import | prospectingService | Start prospecting import |
| POST | /hubspot-export/start | datasetsService | Start HubSpot import |
| POST | /lists/:id/sync/now | datasetsService | Trigger immediate sync |
| POST | /lists/:id/sync/schedule | datasetsService | Set sync schedule |
| DELETE | /lists/:id/sync/schedule | datasetsService | Cancel sync |


## Gateway Integration Endpoints (gatewayClient)

| Method | Path | Service | Purpose |
|--------|------|---------|---------|
| GET | /api/integrations/connected-accounts | integrationsService | List all connections |
| DELETE | /api/integrations/connected-accounts/:id | integrationsService | Disconnect account |
| PUT | /api/integrations/connected-accounts/:id | integrationsService | Edit account name |
| POST | /api/integrations/:provider/validate-key | integrationsService | Validate API key |
| POST | /api/integrations/:provider/connect | integrationsService | Connect via API key |
| GET | /api/integrations/:provider/auth | integrationsService | Get OAuth redirect URL |
| POST | /api/integrations/:provider/connection/setup | integrationsService | Airbyte setup |
| GET | /api/integrations/connected-accounts/by-provider | Various | Accounts by provider |
| GET/POST | /api/integrations/linkedin-ads/* | liveAudiencesService | LinkedIn audience CRUD |
| GET/POST | /api/integrations/meta-ads/* | liveAudiencesService | Meta audience CRUD |
| GET | /api/integrations/google-sheets/accounts/* | exportService | Google Sheets accounts |


## Socket Events

### Client-Emitted Events
| Event | Payload | Purpose |
|-------|---------|---------|
| subscribe_dataset | { datasetId, userId } | Join dataset room |
| unsubscribe_dataset | { datasetId } | Leave dataset room |

### Server Events - Typed (SocketEventMap in `src/lib/socket.ts`)

**Grid events (underscore format):**
| Event | Purpose |
|-------|---------|
| row_enriched | Single row enrichment complete |
| enrichment_progress | Batch enrichment progress |
| row_enrichment_queued | Row queued for enrichment |
| row_enrichment_processing | Row being processed |
| row_enrichment_completed | Row enrichment done |
| validation_progress | Email validation progress |
| row_validation_updated | Single row validated |
| normalization_progress | Normalization progress |
| column:config_updated | Column config change |
| webhook_data_received | New webhook data |
| webhook:column_mapped | Webhook column mapping |
| apollo_import_progress | Prospecting import progress |
| apollo_import_complete | Prospecting import done |
| apollo_import_cancelled | Prospecting import cancelled |
| import:progress | Klaviyo import progress |
| import:completed | Klaviyo import done |

**Notification events (colon format):**
| Event | Purpose |
|-------|---------|
| enrichment:started | Notification: enrichment started |
| enrichment:progress | Notification: enrichment progress |
| enrichment:completed | Notification: enrichment done |
| enrichment:failed | Notification: enrichment failed |
| enrichment:cancelled | Notification: enrichment cancelled |
| row:enrichment:status | Notification: row status |

### Untyped Events
Template view uses `job:update`, `validation:*`, `normalization:*` events not in the shared SocketEventMap. These are handled via `socketService.onAny()`.

> INVESTIGATE: Socket event types are split between typed (SocketEventMap) and untyped (onAny) with no central registry for the untyped ones.


## Missing/Requested Endpoints

The following endpoints were identified as needed but not yet available:

1. `GET /plans` or `GET /pricing/plans` - plan prices are hardcoded in frontend
2. `GET /features/{key}/check` - feature flag check (frontend stub always returns true)
3. `POST /workspaces/:id/leave` - leave workspace (no endpoint available)
4. `X-Workspace-Id` header middleware - workspace authorization boundary on backend (security gap)
5. `DELETE /workspaces/:id` - workspace deletion not supported
