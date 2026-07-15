# Manish 16 — Manifest and Task Checklist

Living checklist of Manish's backend/integration/mobile-functional work, tracked at the module level (per [coordination/09-daily-progress-format.md](../coordination/09-daily-progress-format.md) §3). Update status here as work progresses — this file is the single place to check "what has Manish actually built" without reading git history.

## Phase 0
- [x] All shared, `manish/`, `rohit/`, `coordination/` documentation written

## Phase 1 — Foundation
- [ ] All five repo scaffolds (`citycalls-docs`, `citycalls-api`, `citycalls-admin-web`, `citycalls-customer-mobile`, `citycalls-vendor-mobile`) — [manish/01](01-project-and-repository-setup.md)
- [ ] Backend folder structure in place ([manish/02](02-backend-folder-structure.md))
- [ ] Auth + RBAC ([manish/04](04-authentication-and-rbac-plan.md))
- [ ] Organization hierarchy models + APIs
- [ ] Masters engine + numbering engine + policy engine
- [ ] CI pipeline live

## Phase 2 — Customer and Product Management
- [ ] Customers + Customer Products
- [ ] Dynamic Service Catalog + Brands/Product Types

## Phase 3 — Call and Lead Management
- [ ] Calls (all sub-types)
- [ ] Leads (stages/conversion/merge)

## Phase 4 — Service Request and Assignment
- [ ] Status Engine ([manish/06](06-workflow-engine-plan.md))
- [ ] Assignment Engine
- [ ] Service Request core APIs
- [ ] Real-time (Socket.IO) layer live

## Phase 5 — Field Service Execution
- [ ] Service Visits APIs
- [ ] Offline-sync engine ([manish/09](09-vendor-app-functional-plan.md))
- [ ] Vendor/Customer app functional data layers

## Phase 6 — Financial System
- [ ] Estimate/Proforma/Invoice/Payment chain

## Phase 7 — Follow-up and Happy Calls
- [ ] Happy Calls + Reopen

## Phase 8 — Marketing and Communication
- [ ] Notification Engine full build-out
- [ ] AiSensy + SMTP adapters live

## Phase 9 — AI Features
- [ ] AI provider adapters + settings

## Phase 10 — Reports and Export
- [ ] Report aggregation jobs + endpoints
- [ ] Import/Export engine

## Phase 11 — Testing and Deployment
- [ ] Security review complete
- [ ] Production environment live
- [ ] Backup/restore drill complete

## Non-Phase-Bound (ongoing)
- [ ] Mock JSON kept current for every module ([coordination/05](../coordination/05-mock-data-contract.md))
- [ ] `docs/` updated for any implementation-driven contract change ([coordination/08](../coordination/08-change-request-process.md))
