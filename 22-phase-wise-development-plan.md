# 22 — Phase-Wise Development Plan

Each phase follows the parallel-development sequence from [00-project-overview.md](00-project-overview.md) §7: Manish ships contracts + mocks → Rohit builds UI against mocks → Manish implements real APIs matching the mocks exactly → Rohit swaps the adapter → joint integration testing → phase sign-off → next phase. No phase begins until the previous phase's [23-definition-of-done.md](23-definition-of-done.md) criteria are met.

## Phase 0 — Documentation and Architecture
**Scope**: everything under `docs/` (this phase). **Deliverable**: all shared, `manish/`, `rohit/`, and `coordination/` documents complete. **Acceptance**: user has reviewed and approved the document set (or explicitly authorized proceeding, as happened for this batch — see [coordination/09-daily-progress-format.md](coordination/09-daily-progress-format.md) for how progress is tracked from here).

## Phase 1 — Foundation
**Scope**: repo setup, environments, auth, users, roles/permissions, organization hierarchy (branch/sub-branch/team), dynamic config/masters engine skeleton.
- **Manish**: `citycalls-api` repo scaffold (Express/TS/Mongo connection), auth module (login/refresh/OTP), `users`/`branches`/`sub_branches`/`teams` models + CRUD APIs, masters-engine generic collection + CRUD API, numbering engine, policy-resolution engine, seed script.
- **Rohit**: design system tokens ([rohit/02-design-system.md](rohit/02-design-system.md)), shared component library skeleton, admin-web auth screens (login, forgot password), org-hierarchy management screens, masters management generic screen.
- **Shared**: OpenAPI spec bootstrap, mock JSON for all Phase 1 endpoints, CI pipeline (lint/type-check/test).
- **Dependencies**: none (first phase).
- **APIs**: `/auth/*`, `/users`, `/branches`, `/sub-branches`, `/teams`, `/masters/{type}`, `/config/*`.
- **Screens**: Login, Dashboard shell, Branch/Sub-Branch/Team management, Masters management, User management.
- **Test cases**: auth flows (login/refresh/OTP/reset), role-permission enforcement on a sample protected route, org-hierarchy CRUD, masters CRUD.
- **Definition of done**: see [23-definition-of-done.md](23-definition-of-done.md) §Phase 1.
- **Merge order**: auth → org hierarchy → masters engine (masters engine unblocks Phase 2's catalog work).

## Phase 2 — Customer and Product Management
**Scope**: customers, addresses, products/appliances, brands/models, service categories, dynamic service catalog.
- **Manish**: `customers`, `customer_products` models + APIs, duplicate-detection logic, `services`/`brands`/`product_types` catalog APIs.
- **Rohit**: Customer list/detail/create/edit screens, address management, product management, service catalog management screens.
- **Dependencies**: Phase 1 (masters engine for brand/product-type/service-category lists, org hierarchy for branch-scoping).
- **APIs**: `/customers`, `/customers/{id}/addresses`, `/customers/{id}/products`, `/customers/duplicates`, `/services`, `/brands`, `/product-types`.
- **Screens**: [rohit/04-admin-page-list.md](rohit/04-admin-page-list.md) Customer and Catalog sections.
- **Acceptance criteria**: duplicate detection flags on customer create per configured match rules; a new service is immediately bookable once activated, with no code change.

## Phase 3 — Call and Lead Management
**Scope**: initial/requirement/pre-service/visit/post-service/happy-calls, leads (all stages/sources/conversion).
- **Manish**: `calls` model (with `details` sub-schema per call type) + APIs, `leads` model + APIs including convert/merge/bulk-assign.
- **Rohit**: Call entry screens (per sub-type), call timeline view, Lead pipeline/kanban and list views, lead detail with logs/tasks.
- **Dependencies**: Phase 2 (customer/product linkage).
- **APIs**: `/calls`, `/leads`, `/leads/{id}/convert`, `/leads/merge`, `/leads/bulk-assign`.
- **Acceptance criteria**: a Lead converts to a Service Request carrying forward customer/product/requirement data without re-entry; call duplicate/repeat-issue detection surfaces to the call executive.

## Phase 4 — Service Request and Assignment
**Scope**: booking, the full status/workflow engine, assignment (hierarchy + bypass + vendor), appointment scheduling, SLA, escalation.
- **Manish**: `service_requests` model + status-transition engine (enforcing [07-status-transition-matrix.md](07-status-transition-matrix.md)), assignment engine (rule-assisted + manual + bypass), `assignment_history`, SLA/escalation background jobs, Socket.IO real-time layer live from this phase.
- **Rohit**: Service Request list/detail/create screens, dispatch/assignment UI (ranked suggestions + override), real-time status badges, escalation views.
- **Dependencies**: Phase 1–3 (org, catalog, customer, call/lead linkage).
- **Acceptance criteria**: every transition in [07-status-transition-matrix.md](07-status-transition-matrix.md) enforced server-side with the exact role gating specified; real-time status updates visible on admin dashboard within the socket round-trip, no polling.

## Phase 5 — Field Service Execution
**Scope**: technician/vendor mobile app functional core — travel, inspection, diagnosis, parts, images, completion, offline sync.
- **Manish**: `service_visits` model + APIs, offline-sync endpoint with idempotency handling, image upload flow (signed Cloudinary or fallback), Flutter app functional/data-layer setup for both mobile apps.
- **Rohit**: Vendor/Employee app screens (job list, job detail, execution flow screens) per [rohit/06-vendor-app-screen-list.md](rohit/06-vendor-app-screen-list.md), offline-state UI indicators.
- **Dependencies**: Phase 4 (Service Request + status engine).
- **Acceptance criteria**: full offline capture-then-sync journey (§5 of [08-system-architecture.md](08-system-architecture.md)) verified with airplane-mode testing; a conflicting queued status change surfaces a resolvable conflict, not data loss or a forced overwrite.

## Phase 6 — Financial System
**Scope**: estimates, proforma invoices, invoices, payments, receipts, credit/debit notes, vendor payouts.
- **Manish**: financial document models + numbering + PDF generation + conversion-chain APIs, payment recording APIs, vendor-payout accrual logic.
- **Rohit**: Estimate/Invoice creation and detail screens, payment recording UI, customer-facing estimate approval (customer app), invoice/receipt PDF viewing.
- **Dependencies**: Phase 4 (Service Request linkage), Phase 2 (customer billing details).
- **Acceptance criteria**: GST split (CGST/SGST vs IGST) computed correctly per branch/customer state; an invoice cannot be edited in place post-payment (only credit/debit note correction, per [16-pdf-and-financial-documents.md](16-pdf-and-financial-documents.md) §4).

## Phase 7 — Follow-up and Happy Calls
**Scope**: post-service follow-up, happy calls, reopen flow, warranty tracking.
- **Manish**: `happy_calls`, `reopen_records` models + APIs, reopen-eligibility calculation against the policy engine, scheduled happy-call task generator.
- **Rohit**: Happy Call entry/list screens, Reopen request flow (admin + customer app), warranty status display.
- **Dependencies**: Phase 4–6 (a completed, paid Service Request is the trigger).
- **Acceptance criteria**: reopen eligibility correctly resolves the most-specific applicable policy per [09-database-architecture.md](09-database-architecture.md) §5, verified with overlapping test policies.

## Phase 8 — Marketing and Communication
**Scope**: AiSensy WhatsApp, SMTP email, campaigns, templates, segments, consent, delivery logs.
- **Manish**: notification-engine full build-out (trigger catalog from [13-notification-and-template-system.md](13-notification-and-template-system.md)), AiSensy/SMTP adapters, campaign scheduling jobs.
- **Rohit**: Notification template management, campaign builder, audience segmentation UI, consent management screens, delivery-log/analytics views.
- **Dependencies**: Phase 1 (customers exist to message), can build in parallel with Phases 4–7 once customer data exists, since notifications are triggered by events across all prior phases.
- **Acceptance criteria**: disabling AiSensy or SMTP mid-operation does not block any core workflow action, verified per Test Case #5 in [19-testing-strategy.md](19-testing-strategy.md) §3.

## Phase 9 — AI Features
**Scope**: Gemini/OpenAI settings, summarization, classification, suggestions.
- **Manish**: AI provider adapters, `ai_requests` logging, per-feature enable flags, usage/cost caps.
- **Rohit**: AI settings screen, AI-suggestion UI affordances (always presented as suggestion/override, never auto-applied) embedded into relevant existing screens (call entry, lead scoring).
- **Dependencies**: Phase 3 (calls/leads exist to summarize/classify).
- **Acceptance criteria**: every AI feature has a fully functional non-AI fallback path; usage cap enforcement verified.

## Phase 10 — Reports and Export
**Scope**: dashboards, full report catalog, Excel export/import, PDF/print.
- **Manish**: report-query APIs (pre-aggregated where needed per [08-system-architecture.md](08-system-architecture.md) performance NFR), generic export/import engine per [15-excel-import-export-specification.md](15-excel-import-export-specification.md).
- **Rohit**: Dashboard screens per role ([rohit/09-dashboard-and-chart-specifications.md](rohit/09-dashboard-and-chart-specifications.md)), report list/filter/export UI.
- **Dependencies**: all prior phases (reports aggregate across every module).
- **Acceptance criteria**: every table supports search/filter/sort/pagination/export per [10-api-standards.md](10-api-standards.md); large exports run as background jobs without blocking the UI.

## Phase 11 — Testing and Deployment
**Scope**: security review, performance testing, UAT, production deployment, monitoring, backup verification.
- **Shared**: full E2E suite run (§3 of [19-testing-strategy.md](19-testing-strategy.md)), security review against [17-security-and-audit.md](17-security-and-audit.md), load testing on list/report endpoints, production environment provisioning per [20-deployment-and-environments.md](20-deployment-and-environments.md), backup-restore drill.
- **Acceptance criteria**: [24-final-documentation-review-checklist.md](24-final-documentation-review-checklist.md) and every phase's DoD confirmed complete; UAT sign-off from the user.

## Later Phase — AMC / Contract Management (P3, not scheduled)
Activated only after Phase 11, using the reserved `contracts` schema stub from [09-database-architecture.md](09-database-architecture.md) — scoped and planned as its own mini-phase-set when prioritized.
