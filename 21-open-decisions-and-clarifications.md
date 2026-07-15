# 21 — Open Decisions and Clarifications

Living document. Every "flagged," "TBD," "default assumed," or "pending confirmation" item across the documentation set is tracked here so nothing silently becomes permanent just because a default was written down somewhere to keep moving. Update this doc the moment a decision is made — move the row to "Resolved" with the decision and date, don't just delete it.

## 1. Resolved (for reference — decided via AskUserQuestion on 2026-07-15)

| Item | Decision | Where it affects |
|---|---|---|
| Mobile app tech stack | Flutter/Dart for both customer and vendor apps | [00-project-overview.md](00-project-overview.md) §5, [08-system-architecture.md](08-system-architecture.md) §9, all `manish/08-09`, `rohit/05-06` |
| Tenancy model | Single company, multi-branch (not multi-tenant SaaS) | [01-business-requirements-document.md](01-business-requirements-document.md) §7, [09-database-architecture.md](09-database-architecture.md) |
| Real-time scope | Required from day one (Socket.IO) | [08-system-architecture.md](08-system-architecture.md) §4, Phase 4 in [22-phase-wise-development-plan.md](22-phase-wise-development-plan.md) |
| Deployment target | Not decided — docs written cloud-agnostically | [20-deployment-and-environments.md](20-deployment-and-environments.md) |
| Repository structure | Multi-repo — five independent Git repositories (`citycalls-docs`, `citycalls-api`, `citycalls-admin-web`, `citycalls-customer-mobile`, `citycalls-vendor-mobile`), **no shared code package of any kind**, both mobile apps fully independent of each other too. Decided 2026-07-15, overriding the originally-drafted monorepo-with-shared-packages design. The OpenAPI spec in `citycalls-docs` replaces the old shared-types package as the cross-repo contract mechanism, synced manually per repo. | [manish/01-project-and-repository-setup.md](manish/01-project-and-repository-setup.md), [00-project-overview.md](00-project-overview.md) §7-8, [08-system-architecture.md](08-system-architecture.md) §9, [12-frontend-data-contracts.md](12-frontend-data-contracts.md), all of `coordination/02,03,04,05,07,08,11` |

## 2. Open — Needs a Decision Before the Relevant Phase Begins

| Item | Default assumed in docs | Needs deciding before | Affects |
|---|---|---|---|
| Hosting/compute provider | None — cloud-agnostic | Phase 4 (staging environment needed for integration testing) | [20-deployment-and-environments.md](20-deployment-and-environments.md) §7 |
| Payment gateway provider | None wired — manual payment methods only | Before online in-app payment is needed (likely Phase 6+) | [14-integration-architecture.md](14-integration-architecture.md) §6, [manish/08](manish/08-customer-app-functional-plan.md) §6 |
| SMS provider | None wired — interface-ready only | Whenever SMS notifications become a priority (not currently on critical path) | [14-integration-architecture.md](14-integration-architecture.md) §7 |
| Technician location-tracking granularity | Event-based pings + periodic ping while en route (default 60s), not continuous GPS trail | Before Phase 4 real-time work begins, due to battery/privacy/consent implications | [08-system-architecture.md](08-system-architecture.md) §4, [manish/09](manish/09-vendor-app-functional-plan.md) §5 |
| Parts/inventory stock tracking model | Not decided — current schema records parts as line items on a Service Visit with no stock/reorder concept | Before Phase 6 if the business wants actual inventory tracking (stock levels, reorder alerts) rather than just record-keeping | [09-database-architecture.md](09-database-architecture.md) `service_visits.parts`, materially changes the schema if stock tracking is wanted |
| Offline-first for vendor app | Assumed necessary and designed in from the start (§5 of [08-system-architecture.md](08-system-architecture.md)) — proposed by the assistant based on likely field conditions, not explicitly requested by the user | Confirm this is actually wanted before Phase 5 — it adds real implementation complexity (local DB, sync engine, conflict UI) | [manish/09](manish/09-vendor-app-functional-plan.md), [rohit/06](rohit/06-vendor-app-screen-list.md) |
| AiSensy transactional vs. marketing template categorization | Assumed distinguishable per WhatsApp Business API policy, exact rules TBD | Before Phase 8 (WhatsApp integration) | [14-integration-architecture.md](14-integration-architecture.md) §4 |
| Device binding / stricter session security for mobile apps | Not enforced in v1 (login from new device works without extra verification) | Revisit if the business wants stricter control (e.g. vendor account-sharing prevention) | [17-security-and-audit.md](17-security-and-audit.md) §9 |
| Flutter state management: Riverpod vs. Bloc | Riverpod recommended as default | Rohit's call, confirm before Phase 1 mobile scaffolding | [12-frontend-data-contracts.md](12-frontend-data-contracts.md) §3 |
| Financial year format/boundary | Assumed India-standard Apr–Mar (`FY2526` style code) | Confirm if the business's FY differs | [coordination/06-naming-conventions.md](coordination/06-naming-conventions.md) §3 |
| Customer confirmation auto-timeout behavior | Default: auto-confirms after a configurable window if customer doesn't respond | Confirm whether some businesses should instead leave it pending indefinitely (configurable either way, but default behavior needs sign-off) | [06-complete-workflow-document.md](06-complete-workflow-document.md) Stage 8 |
| Multi-role-per-user | Not supported in v1 (one role per `User`) | Revisit if an actual need for a person holding two roles simultaneously arises | [05-user-roles-and-permissions.md](05-user-roles-and-permissions.md) §1 |
| Multi-currency / multi-language | Assumed India-only, INR, single language | Confirm if any planned expansion changes this | [00-project-overview.md](00-project-overview.md) §4 |
| Team working hours/availability expectations | Not specified (informal two-person team) | Add if the team scales or timezone differences emerge | [coordination/01-team-working-agreement.md](coordination/01-team-working-agreement.md) §7 |

## 3. Missing/Under-Specified Functionality Flagged During Analysis (not in original requirements, assistant-proposed additions)

| Item | Status | Where addressed |
|---|---|---|
| Working-hours/holiday-aware SLA calculation | Adopted into architecture | [manish/06-workflow-engine-plan.md](manish/06-workflow-engine-plan.md) §5 |
| Duplicate Service Request detection (not just duplicate customer/lead) | Adopted | [06-complete-workflow-document.md](06-complete-workflow-document.md) Stage 1 |
| Technician daily route/schedule view | Adopted | [manish/09](manish/09-vendor-app-functional-plan.md) §7, [rohit/06](rohit/06-vendor-app-screen-list.md) |
| Parts/inventory stock tracking | Flagged, not decided — see §2 above | — |
| Consent/DND compliance audit trail | Adopted as a compliance control, not just UX | [17-security-and-audit.md](17-security-and-audit.md) §8, §10 |
| Offline mode for vendor app | Adopted, but flagged for confirmation — see §2 above | — |
| Rate limiting on booking/OTP endpoints | Adopted | [17-security-and-audit.md](17-security-and-audit.md) §4 |

## 4. Data/Input Gaps

| Item | Status |
|---|---|
| The two Excel screenshots referenced in the original requirements | **Never actually attached** in the conversation — [03-screenshot-and-excel-analysis.md](03-screenshot-and-excel-analysis.md) was written from the user's detailed text description only. Should be revisited against real sample files (exact column names/order, any field the text description omitted) before the Excel import templates in [15-excel-import-export-specification.md](15-excel-import-export-specification.md) are finalized or a legacy migration is attempted. |

## 5. Process Note

Per the project's memory record: the original requirement was one-document-at-a-time with explicit approval between each; on 2026-07-15 the user authorized producing the full Phase 0 set in one continuous pass ("karo bhai sabhi bnata chalo"). This document set has **not** yet had a per-file line-by-line user review — that review is still expected before Phase 1 implementation begins, per [24-final-documentation-review-checklist.md](24-final-documentation-review-checklist.md).
