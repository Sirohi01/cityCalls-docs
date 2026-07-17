# Manish 16 — Manifest and Task Checklist

Living checklist of Manish's backend/integration/mobile-functional work, tracked at the module level (per [coordination/09-daily-progress-format.md](../coordination/09-daily-progress-format.md) §3). Update status here as work progresses — this file is the single place to check "what has Manish actually built" without reading git history.

**Status as of 2026-07-17**, corrected against actual `citycalls-api` source/git history/live-server verification — this file had never been updated before (every box was still unchecked despite 10 backend phases being complete), so it could not be trusted at face value. See inline notes for exactly what's verified vs. still outstanding.

## Phase 0
- [x] All shared, `manish/`, `rohit/`, `coordination/` documentation written

## Phase 1 — Foundation
- [x] All five repo scaffolds (`citycalls-docs`, `citycalls-api`, `citycalls-admin-web`, `citycalls-customer-mobile`, `citycalls-vendor-mobile`) — [manish/01](01-project-and-repository-setup.md)
- [x] Backend folder structure in place ([manish/02](02-backend-folder-structure.md))
- [x] Auth + RBAC ([manish/04](04-authentication-and-rbac-plan.md)) — login/refresh/logout/OTP/password-reset, sessions, full permission-cache middleware
- [x] Organization hierarchy models + APIs — Branch/SubBranch/Team
- [x] Masters engine + numbering engine + policy engine
- [x] CI pipeline live — `.github/workflows/ci.yml` present and correct (typecheck → lint → test → build); actual GitHub Actions run history not verified this session (no `gh` CLI available)

## Phase 2 — Customer and Product Management
- [x] Customers + Customer Products — incl. addresses, duplicate detection, consent-with-audit-trail
- [x] Dynamic Service Catalog + Brands/Product Types

## Phase 3 — Call and Lead Management
- [x] Calls (all 6 sub-types)
- [x] Leads (stages/conversion/merge) — generic status-transition engine built here, reused by every later status-driven entity

## Phase 4 — Service Request and Assignment
- [x] Status Engine ([manish/06](06-workflow-engine-plan.md)) — full 37-status transition table
- [x] Assignment Engine — skill/coverage/workload ranking
- [x] Service Request core APIs
- [x] Real-time (Socket.IO) layer live

## Phase 5 — Field Service Execution
- [x] Service Visits APIs
- [x] Offline-sync engine ([manish/09](09-vendor-app-functional-plan.md)) — server-side sync-batch endpoint + idempotency ledger
- [ ] **Vendor/Customer app functional data layers — NOT STARTED.** [manish/08](08-customer-app-functional-plan.md)/[manish/09](09-vendor-app-functional-plan.md) scope this as Manish's responsibility inside the two Flutter repos (repositories, Dio client wiring, the vendor app's SQLite/drift local persistence + sync engine, FCM, location pings). None of it exists — both mobile repos currently contain only the login screen (Rohit/Manish's shared early scaffold), no `lib/data/` or `lib/sync/` layer at all.

## Phase 6 — Financial System
- [x] Estimate/Proforma/Invoice/Payment chain — incl. Credit/Debit notes, server-side GST split, vendor invoice/payout chain

## Phase 7 — Follow-up and Happy Calls
- [x] Happy Calls + Reopen — root-traced reopen ledger, warranty check, auto-escalation

## Phase 8 — Marketing and Communication
- [x] Notification Engine full build-out — trigger→template→channel→delivery-log
- [x] AiSensy + SMTP adapters live — both integration-ready and gated by `isXEnabled()`; neither has been exercised against a real account/SMTP server in this dev environment (no credentials configured here)

## Phase 9 — AI Features
- [x] AI provider adapters + settings — Gemini/OpenAI adapter, settings, usage caps; not exercised against a real API key in this environment

## Phase 10 — Reports and Export
- [x] Report aggregation jobs + endpoints — **note**: implemented as live on-demand aggregation pipelines, not the docs' nightly-refreshed pre-aggregated summary collections (deferred — no Redis/BullMQ available in this dev environment; documented in `citycalls-api/README.md`)
- [x] Import/Export engine — generic registry-driven CSV/XLSX export + CSV-only import (see security note below)

## Phase 11 — Testing and Deployment
- [ ] Security review complete — **partially started as a byproduct of this audit**: found and fixed one real production bug (`GET /audit/logs` 500ing on real data), fixed 3 endpoints that were silently returning fake data instead of live queries, and confirmed a `xlsx` package CVE was already correctly mitigated (import is CSV-only, hand-parsed). Not yet done: a full pass per [17-security-and-audit.md](../17-security-and-audit.md), and closing the `req.scope`/`applyScopeFilter` gap noted below.
- [ ] Production environment live — blocked on a hosting-provider decision ([21-open-decisions-and-clarifications.md](../21-open-decisions-and-clarifications.md) §5)
- [ ] Backup/restore drill complete

## Known gaps found during the 2026-07-17 audit (not previously tracked anywhere)
- [ ] `GET /auth/me` (resolved-permissions-for-current-user endpoint, documented in [manish/10](10-admin-functional-integration-plan.md) §3) — does not exist.
- [ ] `GET /search` (cross-entity global search, [manish/10](10-admin-functional-integration-plan.md) §4) — does not exist.
- [ ] `citycalls-admin-web`'s TanStack Query hook layer ([manish/10](10-admin-functional-integration-plan.md)) — partially exists (~22 hooks), but authored by a third-party contributor rather than delivered ahead of Rohit's screens per the documented process; several hooks (`useForgotPassword`, `useResetPassword`) are themselves still `setTimeout`-mocked, not real.
- [ ] Error tracking (Sentry or equivalent) — not wired into any repo, despite being a documented Phase 1 item ([manish/15](15-deployment-plan.md) §3).
- [ ] Client-side RBAC in admin-web (`usePermission()`) is currently hardcoded to always return `true` — the nav is structurally permission-gated but not functionally gated yet. (Server-side authorization via `requirePermission` middleware is real and enforced — this gap is UI-only.)
- [ ] `src/lib/scopeFilter.ts`'s `applyScopeFilter` has zero call sites across Phases 1-9 — data-scope filtering (e.g. a BRANCH_MANAGER's list queries being provably branch-filtered at the DB level) isn't consistently applied outside Phase 10's own new code (reports, export).

## Non-Phase-Bound (ongoing)
- [ ] Mock JSON kept current for every module ([coordination/05](../coordination/05-mock-data-contract.md)) — **not maintained**: every repo's `mocks/` folder is currently empty.
- [x] `docs/` updated for any implementation-driven contract change ([coordination/08](../coordination/08-change-request-process.md)) — `openapi/citycalls.yaml` (canonical contract spec) written 2026-07-16, covers all 142 real Phase 1-10 endpoints, verified 1:1 against actual route source.
