# Manish 05 — Module-Wise Backend Plan

Implementation notes per module, beyond what [09-database-architecture.md](../09-database-architecture.md) and [11-complete-api-contracts.md](../11-complete-api-contracts.md) already specify. Build order follows [22-phase-wise-development-plan.md](../22-phase-wise-development-plan.md).

## Organization (`branches`, `sub_branches`, `teams`)
Coverage matching (pin code → branch) is a simple array-contains lookup for v1, not geospatial — a pin code list per branch, checked with `$in`. If pin-code overlap between branches ever needs priority rules (two branches covering the same pin code), resolve by explicit priority field, not first-match-wins silently.

## Vendors
`vendor.performance` fields (acceptance/rejection/completion rate) are **computed, not stored as manually-updated counters** — a scheduled job recalculates them nightly from `assignment_history`/`service_requests` aggregation, avoiding drift between the stored number and reality.

## Customers
Duplicate detection (`GET /customers/duplicates`) runs a fuzzy-match query (mobile exact match is authoritative; name/address similarity is a secondary suggestion, not auto-merge) — merging is always a manual admin action, never automatic, since a false-positive merge is far more damaging than a missed duplicate.

## Catalog / Services
`Service.active` toggling is checked at booking time, not cached — a service deactivated mid-day immediately stops being bookable without a deploy or cache-bust.

## Calls
The `details` sub-document (per [09-database-architecture.md](../09-database-architecture.md) `calls` schema) validation is dispatched by `callType` in `calls.validation.ts` — one Zod discriminated union, not a generic `any` field, so call-type-specific required fields are still enforced.

## Leads
`leads/{id}/convert` is a transaction (Mongo session) spanning the Lead update and the new ServiceRequest/Customer creation — either both succeed or neither does, avoiding a half-converted lead.

## Service Requests / Status Engine
`lib/statusEngine.ts` is the single function every status-changing endpoint calls (`assertValidTransition(entityType, from, to, actorRole)`) — throws `INVALID_TRANSITION` (per [18-error-handling-standards.md](../18-error-handling-standards.md)) if not permitted. This is the literal enforcement point for [07-status-transition-matrix.md](../07-status-transition-matrix.md); no controller ever writes a `status` field directly without going through it.

## Field Execution
Offline-sync endpoint accepts a batch of queued actions with client timestamps and idempotency keys; processes them **in client-recorded order** per Service Request, running each through the same `statusEngine` check — a stale/invalid queued transition is rejected individually (per action), not the whole batch, per [18-error-handling-standards.md](../18-error-handling-standards.md) §7.

## Finance
Numbering + GST-split logic lives in `finance/lib/` shared by all document types (Estimate/Proforma/Invoice) rather than duplicated per model, since the calculation rules are identical across them (§3 of [16-pdf-and-financial-documents.md](../16-pdf-and-financial-documents.md)).

## Notifications
`notifications.trigger(triggerKey, context)` is the only entry point other modules use — internally resolves recipients/template/channels from the `notification_templates`/rule config and enqueues jobs (§1 of [13-notification-and-template-system.md](../13-notification-and-template-system.md)).

## Reports
Heavy aggregations (branch performance, defect analysis) run against **pre-aggregated summary collections** refreshed by scheduled jobs, not live `$group` pipelines on the raw operational collections at request time — protects list/detail endpoint performance from being impacted by report queries, per the NFR in [01-business-requirements-document.md](../01-business-requirements-document.md) §6.
