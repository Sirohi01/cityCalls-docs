# Manish 06 — Workflow Engine Plan

## 1. Status Engine (`lib/statusEngine.ts`)

Loads `status_transitions` (seeded per [07-status-transition-matrix.md](../07-status-transition-matrix.md)) into memory on boot, refreshed on config change. Exposes:
- `assertValidTransition(entityType, fromStatus, toStatus, actorRole)` — throws `INVALID_TRANSITION` if not in the allowed set.
- `getAllowedTransitions(entityType, currentStatus, actorRole)` — used to populate the `allowedTransitions[]` array in the `409` error response (§1.4 of [11-complete-api-contracts.md](../11-complete-api-contracts.md)) and to drive which status-change buttons the UI shows.

## 2. Assignment Rule Engine (`lib/assignmentEngine.ts`)

Given a Service Request, produces a ranked list of candidate assignees by scoring: skill match (binary gate), coverage match (binary gate), current workload (inverse — fewer active jobs scores higher), availability (binary gate against calendar), historical performance (completion rate/reopen rate, weighted), SLA remaining (higher urgency weighted higher). Weights are configurable (`config.assignmentRuleWeights`), not hardcoded, so the business can tune the engine's behavior without a deploy. Full auto-assignment (engine commits without human review) is a per-Service/Branch config flag, off by default.

## 3. Policy Resolver (`lib/policyResolver.ts`)

Implements the most-specific-wins lookup from §5 of [09-database-architecture.md](../09-database-architecture.md) — `resolvePolicy(policyType, context: {customerId, contractId?, brandId, productId, serviceId, branchId})` checks each scope in order, returns the first match's `rules`, falls back to the seeded `GLOBAL` policy.

## 4. Escalation Engine (`jobs/escalationCheck.job.ts`)

Scheduled job (runs every N minutes, configurable) scans for: acceptance-SLA breaches, overall SLA breaches (respecting working-hours/holiday-aware timers — see §5), repeated-rejection threshold crossings. Sets `isEscalated: true` + `escalationReason`, triggers the `SLA_BREACHED` notification per [13-notification-and-template-system.md](../13-notification-and-template-system.md).

## 5. Working-Hours-Aware SLA Timers

SLA "minutes remaining" calculation excludes time outside the applicable Branch's `workingHours`/`holidays` (per [09-database-architecture.md](../09-database-architecture.md) `branches` schema) — implemented as a small business-calendar utility (`lib/businessCalendar.ts`) rather than naive wall-clock subtraction, addressing the gap flagged in the project's initial missing-functionality review.

## 6. Idempotent Status Writes

Every status-changing call passes through a single `updateStatus(serviceRequestId, toStatus, actorId, meta)` function that: calls `statusEngine.assertValidTransition`, writes the new status, writes the `activity_logs` entry, triggers relevant notifications, and emits the Socket.IO event — all in one place, so no code path can change a status without triggering its required side effects (this is the concrete mechanism behind the "every status change writes history and triggers notifications" rule in [06-complete-workflow-document.md](../06-complete-workflow-document.md)).
