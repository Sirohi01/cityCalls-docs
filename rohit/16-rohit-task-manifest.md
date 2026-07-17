# Rohit 16 — Task Manifest

Living checklist of Rohit's UI work, mirroring [manish/16-manifest-and-task-checklist.md](../manish/16-manifest-and-task-checklist.md) at the same module granularity, tracked per [coordination/09-daily-progress-format.md](../coordination/09-daily-progress-format.md) §3.

**Status as of 2026-07-17**, corrected against actual repo source (`citycalls-admin-web`, `citycalls-customer-mobile`, `citycalls-vendor-mobile`) via a full code-level audit — not just file/folder existence. This file was briefly overwritten on 2026-07-17 to show almost every item across 11 phases as `[x]` done (including a "Phase 11" this plan never had); that version did not match the actual code and has been corrected back here. See the inline notes — many items are genuinely **partial**: a real page exists and is wired to the live API, but a sibling page/tab on the same screen still uses a hardcoded `mock*` array instead.

## Phase 0
- [x] All shared, `manish/`, `rohit/`, `coordination/` documentation written

## Foundational (ahead of Phase 1 features)
- [ ] Design system tokens ([rohit/02](02-design-system.md)) — `src/tokens/` (admin-web) and `lib/tokens/` (both Flutter apps) are empty directories; `design-system/page.tsx` is a hardcoded Tailwind style-guide demo page, not an actual token system.
- [ ] Shared component inventory built ([rohit/03](03-shared-component-inventory.md)) — only 4 of ~12 documented admin-web components exist (`DataTable`, `AppFormField`, `StatusBadge`, `PermissionGate`); missing `Timeline`, `AssignmentPanel`, `FileUploadZone`, `EmptyState`/`ErrorState`/`LoadingState`, `PdfViewerModal`, `MasterListManager`, `RealtimeIndicator`, `DetailPageLayout`, `FilterBar`. No `lib/widgets/` folder exists in either Flutter repo at all.

## Phase 1 — Foundation
- [ ] Auth screens (admin-web) — login is real and wired to `/auth/login`; forgot-password/reset-password pages exist visually but `useForgotPassword`/`useResetPassword` are literally `console.log('Mock API call...')` + `setTimeout`, not real calls.
- [ ] Admin shell + nav (permission-gated) — `AdminLayout.tsx` and 14 nav groups are real, every item wrapped in `<PermissionGate module=...>`; but `usePermission()` is hardcoded `return true` for everything, so the gating doesn't actually gate anything yet.
- [x] Masters/Config generic screens — wired to the real `/masters/:masterType` endpoint via `useMasters`.
- [ ] Organization management screens — Branches page real; Sub-Branches and Teams pages (`organization/sub-branches/page.tsx`, `teams/page.tsx`) use hardcoded `mockSubBranches`/`mockTeams` arrays. List-only for all three, no detail/edit pages.

## Phase 2 — Customer and Product Management
- [x] Customer management screens (admin) — list/detail/create wired to `useCustomers`; `duplicate-review/page.tsx` still uses a `mockDuplicates` array.
- [ ] Service catalog management screens (admin) — Services list/detail/create wired to real hooks; the Brands page's Product-Type and Model tabs use `mockProductTypes`/`mockModels`.
- [ ] Customer app onboarding/profile screens — **not started**; `citycalls-customer-mobile` has only a login screen.

## Phase 3 — Call and Lead Management
- [x] Call entry/list/timeline screens — list/entry/`[id]` timeline all wired to real hooks.
- [x] Lead pipeline/list/detail screens — kanban pipeline and detail wired to `useLeads`; a separate `leads/list/page.tsx` duplicates this with a `mockLeads` array instead of reusing the same hook.

## Phase 4 — Service Request and Assignment
- [x] Service Request list/detail/dispatch screens (admin) — genuinely wired (`useServiceRequests`, `useAssignServiceRequest`); dispatch board is a functional two-pane assignment UI hitting `/service-requests/{id}/assign`.
- [ ] Customer app booking + tracking screens — **not started**.

## Phase 5 — Field Service Execution
- [ ] Vendor/Employee app job list + execution flow screens — **not started**; `citycalls-vendor-mobile` has only a login screen. No job list, accept/reject, travel, check-in, diagnosis, estimate, or completion screens exist.
- [ ] Sync status / conflict resolution UI — **not started anywhere**, in either the vendor app or admin-web. (A prior version of this file claimed this was "covered via Workforce management admin scope" — the Employees/Vendors admin pages are plain read-only tables with no sync/conflict UI of any kind; that claim did not match the code.) This is flagged in `rohit/06` as the single most important non-negotiable requirement for the vendor app and remains fully unbuilt.

## Phase 6 — Financial System
- [ ] Estimate/Invoice/Payment screens (admin) — Estimates/Invoices list pages wired to real hooks; the "Create Estimate" flow is a static demo table with hardcoded line items and no working add-item/save logic. No Proforma Invoice, Payment recording, or Credit/Debit Note screens exist at all.
- [ ] Customer app estimate-approval + payment screens — **not started**.

## Phase 7 — Follow-up and Happy Calls
- [x] Happy Call + Reopen screens — Happy Call list/entry wired to real hooks; reopen requests have a real list view (`useReopenRequests`, now backed by a real endpoint as of 2026-07-17 — see [manish/16](../manish/16-manifest-and-task-checklist.md)) but no distinct review/approve workflow screen beyond the list.

## Phase 8 — Marketing and Communication
- [ ] Notification center + template management — center wired to `useNotifications`, but its stat cards are hardcoded and `notifications/templates/page.tsx` uses a `mockTemplates` array.
- [ ] Campaign/segmentation/consent screens — campaign list wired to `useCampaigns`; no segmentation, consent-management, or delivery/analytics dashboard screens exist (all separately documented in `rohit/04`).

## Phase 9 — AI Features
- [x] AI settings screen + suggestion affordances in existing screens — `useAISettings`/`useUpdateAISettings` real, toggles functional. (An "Automated call transcript/summary" toggle exists in settings, but no actual transcript/summary viewer is integrated into the call detail screen.)

## Phase 10 — Reports and Export
- [ ] Dashboards per role — `dashboard/page.tsx` is 100% static hardcoded numbers, zero API wiring, zero charts, zero configurability.
- [ ] Report catalog + detail screens — `reports/page.tsx` is static cards with fake "Last Generated" labels; Export buttons have no handlers; no real report generation is wired despite `manish/16` confirming the backend `GET /reports/:reportKey` endpoints are real and working.
- [ ] Import/Export wizard screens — pure static shell (dropdowns, a drag-drop zone, buttons) with zero hooks or handlers, despite the backend `GET /export/:entity` / `POST /import/:entity` endpoints being real and working.

## Screens built beyond this plan's original scope (2026-07-17)
Not part of the phase list above, but real, working admin-web screens added alongside the Phase 1-10 work:
- [x] Roles/Permission list view (`useRoles`, now backed by a real endpoint — see `manish/16`); no "add user"/"edit permissions" action handlers wired yet.
- [x] Audit log viewer (`useAuditLogs`, now backed by a real, fixed endpoint — see `manish/16`).

## Non-Phase-Bound (ongoing)
- [ ] Form-field specs kept current for every new form ([rohit/07](07-form-field-specifications.md)) — several forms (`calls/entry`, `happy-calls/entry`, `leads/[id]`, `service-requests/create`) use raw `<textarea>`/inline Tailwind instead of the documented `AppFormField` wrapper.
- [ ] UI testing checklist run for every screen before marking done ([rohit/15](15-ui-testing-checklist.md)) — no test framework configured in `citycalls-admin-web` (no jest/vitest/testing-library/playwright in `package.json`), zero test files. Both Flutter apps have exactly one widget test each, covering only the login screen.
