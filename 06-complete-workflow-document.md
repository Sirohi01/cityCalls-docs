# 06 — Complete Workflow Document

This document walks the full Service Request lifecycle stage by stage. For each stage: actor, preconditions, required/optional fields, status before/after, side effects (DB, notifications, audit, timeline), validations, and failure cases. Exact request/response JSON is in [11-complete-api-contracts.md](11-complete-api-contracts.md); exact status names and role-gated transitions are in [07-status-transition-matrix.md](07-status-transition-matrix.md). Call-specific field inventories are in [03-screenshot-and-excel-analysis.md](03-screenshot-and-excel-analysis.md).

## Lifecycle Overview

```
Customer/Staff creates Service Request
        ↓
Acknowledgement (auto)
        ↓
Admin/Branch review & dispatch
        ↓
Branch → Sub-Branch → Team/Employee   OR   direct/bypass assignment to Vendor/Outsourced
        ↓
Assignee accepts or rejects  (reject → Reassignment Required → back to dispatch)
        ↓
Appointment scheduled / rescheduled
        ↓
Technician: start travel → arrive → inspect/diagnose
        ↓
Estimate generated (if required) → customer approval
        ↓
Work starts → parts/labour recorded → before/after images
        ↓
Service marked complete → customer confirmation (OTP/signature)
        ↓
Invoice → payment
        ↓
Post-service follow-up call → Happy Call
        ↓
Closed   OR   Reopened (within policy window)
```

## Stage 1 — Service Request Creation

**Actors**: Customer (via app), Call Executive / Sales Executive (via admin, from a Call or Lead)

**Preconditions**: For app booking — customer authenticated, selected Service is `active`, selected Address's pin code is covered by at least one Branch/Vendor for that Service. For staff creation — an Initial Call (or Requirement Call) has captured enough detail, OR a Lead has been marked ready to convert.

**Required fields**: `customerId` (existing or newly created inline), `serviceId`, `productId` (existing customer product or new), `address` (existing or new), `symptoms[]` or free-text initial complaint, `priority` (defaults to `NORMAL`), `source` (`CUSTOMER_APP`, `CALL`, `LEAD_CONVERSION`, `WALK_IN`).

**Optional fields**: preferred time slot, issue images/video, notes, linked `callId` or `leadId`.

**Status before → after**: *(none)* → `NEW`

**Side effects**:
- `ServiceRequest` document created with generated number (`SR-{branch}-{FY}-{seq}`, branch resolved from address pin-code coverage, or left `null` pending manual branch assignment if no coverage rule matches — never silently drop the request).
- `ActivityLog` entry: `SERVICE_REQUEST_CREATED`.
- Acknowledgement notification to customer (push + email/WhatsApp per consent) — async, does not block the create response.
- If created from a Lead, Lead status moves toward `CONVERTED` per [07-status-transition-matrix.md](07-status-transition-matrix.md); if from a Call, the Call is linked via `relatedServiceRequestId`.

**Validations**: mobile number format, address pin code present, service must be `active`, product-service compatibility if the Service restricts applicable products.

**Failure cases**: pin code not covered by any branch → request still created but flagged `NEEDS_MANUAL_BRANCH_ASSIGNMENT`, surfaced in an Admin queue rather than silently failing; duplicate-request detection (same customer + product + open symptom within a configurable window, default 3 days) surfaces a warning to staff, not a hard block, since a genuinely new issue can coincide.

## Stage 2 — Review & Dispatch

**Actors**: Super Admin, Admin, Branch Manager (for their branch), or the Automated Assignment Engine if enabled for that Service/Branch.

**Preconditions**: Status is `NEW` or `PENDING_REVIEW`.

**Actions**: forward to Branch (if created without one), forward to Sub-Branch, assign to Team/Employee, or assign directly to Vendor/Outsourced Partner — including bypassing intermediate levels if actor is Super Admin/Admin (see [05-user-roles-and-permissions.md](05-user-roles-and-permissions.md) §6).

**Status before → after**: `NEW` → `ASSIGNED_TO_BRANCH` / `ASSIGNED_TO_SUB_BRANCH` / `ASSIGNED_TO_TEAM` / `ASSIGNED_TO_EMPLOYEE` / `ASSIGNED_TO_VENDOR` / `OUTSOURCED`

**Side effects**: `AssignmentHistory` entry (assigner, assignee, assigneeType, timestamp, method: `MANUAL` | `RULE_ENGINE` | `BYPASS`), assignee notified (push to mobile app if Employee/Vendor Technician, in-app + email if a manager role), request appears in assignee's queue in real time via Socket.IO.

**Assignment-rule engine inputs** (when enabled): service-skill match, pin-code/coverage match, current workload, availability calendar, historical performance, SLA remaining. Always produces a **ranked suggestion list**, never a silent auto-commit unless the Service/Branch config explicitly enables full auto-assignment — see [09-database-architecture.md](09-database-architecture.md) policy schema.

**Failure cases**: no eligible assignee found by the rule engine → falls back to a manual-assignment-required flag, never blocks the request from existing.

## Stage 3 — Accept / Reject

**Actor**: the assignee (Employee, Team Lead, Vendor Owner/Manager, or Vendor Technician depending on how granular the assignment was).

**Status before → after**: `ASSIGNED_TO_*` / `OUTSOURCED` → `ACCEPTED` (or `REJECTED` → `REASSIGNMENT_REQUIRED`)

**Required fields on reject**: `rejectionReason` (from a master list, plus free text).

**Side effects**: on accept — appointment-scheduling step unlocked, original assigner notified; on reject — request returns to the dispatcher's queue with the rejection reason visible, `AssignmentHistory` updated, original assigner notified, SLA clock does not reset (elapsed time carries over) so repeated rejection is visible in reporting.

**Failure cases**: no response within the configured acceptance SLA (default configurable, e.g. 30 minutes) → auto-escalation notification to the next level up, request remains assignable/reassignable by Admin/Branch regardless.

## Stage 4 — Appointment Scheduling

**Actor**: assignee (Employee/Vendor) or Call Executive on a Pre-Service Call.

**Required fields**: `scheduledDate`, `scheduledSlot` (from configured appointment slots for that Branch), confirmation method (call/app).

**Status before → after**: `ACCEPTED` → `APPOINTMENT_SCHEDULED` (or `RESCHEDULED` if changing an existing appointment, with `rescheduleReason` and count tracked against the Service's reschedule policy).

**Side effects**: customer notified of the appointment; if rescheduled, both customer and assignee notified; a Pre-Service Call record may be logged with outcome `CONFIRMED` / `RESCHEDULED` / `UNREACHABLE`.

**Failure cases**: customer unreachable for confirmation → Pre-Service Call outcome `CUSTOMER_UNAVAILABLE`, a follow-up task auto-created, status moves to `CUSTOMER_UNAVAILABLE` (does not silently stay `APPOINTMENT_SCHEDULED` with a stale date).

## Stage 5 — Travel, Arrival, Inspection

**Actor**: Employee/Vendor Technician via mobile app (offline-capable — see [manish/09-vendor-app-functional-plan.md](manish/09-vendor-app-functional-plan.md)).

**Sequence & status**: `APPOINTMENT_SCHEDULED` → (start travel) `TECHNICIAN_EN_ROUTE` → (arrival/check-in, geolocation captured) `TECHNICIAN_ARRIVED` → (begin diagnosis) `INSPECTION_STARTED` → (diagnosis recorded) `INSPECTION_COMPLETED`

**Required fields at Inspection Completed**: `actualDefectFound`, `symptoms confirmed`, `solutionType` (from master), whether parts are required, before-service images (count per Service's `requiredImages` config).

**Side effects**: each sub-step is a `ServiceVisit` update, timeline-visible to admin and (status-only, not raw notes) to the customer in real time; a `ServiceVisit` record is created at "start travel" and closed at completion of that visit (a Service Request may span multiple `ServiceVisit` records if work spans multiple days).

**Failure cases**: customer not present at arrival → status `CUSTOMER_UNAVAILABLE`, technician logs a Visit Call/Update with outcome `CUSTOMER_UNAVAILABLE`, reschedule flow re-enters at Stage 4.

## Stage 6 — Estimate & Customer Approval

**Actor**: Technician (drafts) or Branch/Finance (reviews/adjusts) generates the Estimate; Customer approves.

**Trigger**: Service's config flags whether an estimate is mandatory before paid work beyond the visiting/inspection charge, or optional/skippable for low-value standard jobs.

**Status before → after**: `INSPECTION_COMPLETED` → `ESTIMATE_PENDING` → `ESTIMATE_SHARED` → `AWAITING_CUSTOMER_APPROVAL` → `ESTIMATE_APPROVED` / `ESTIMATE_REJECTED`

**Required fields**: parts (name/qty/price from master + technician-entered), labour charge, taxes, total; delivery channel (email/WhatsApp/in-app) per customer consent.

**Side effects**: Estimate document created/numbered (see [16-pdf-and-financial-documents.md](16-pdf-and-financial-documents.md)), PDF generated, sent via enabled channels; customer approval/rejection captured with timestamp and (for app) an explicit tap-to-approve action, not implicit.

**Failure cases**: Estimate rejected → status `ESTIMATE_REJECTED`, work does not proceed past visiting/inspection charge unless customer separately authorizes partial work; no response within a configurable follow-up window → a follow-up task is created, request does not auto-cancel.

## Stage 7 — Work Execution

**Actor**: Technician.

**Status before → after**: `ESTIMATE_APPROVED` (or directly from `INSPECTION_COMPLETED` if no estimate required) → `WORK_STARTED` → `WORK_IN_PROGRESS` → (optionally) `ON_HOLD` / `PARTS_PENDING` → `SERVICE_COMPLETED`

**Required fields during work**: work-progress notes, part usage confirmation (qty actually used may differ from estimate — variance is recorded, not silently overwritten), after-service images (per Service's required-image config), final remarks.

**Side effects**: real-time status pushed to customer app; if parts become newly required mid-work, an Estimate revision may be triggered per Service config.

**Failure cases**: work cannot finish in one visit → status `ON_HOLD` or `PARTS_PENDING` with a `nextVisitDate`, a new `ServiceVisit` is scheduled without creating a new Service Request.

## Stage 8 — Completion & Customer Confirmation

**Actor**: Technician captures; Customer confirms.

**Required fields**: completion proof per the Service's configured requirement — OTP sent to customer's registered mobile, or digital signature, or in-app confirmation tap.

**Status before → after**: `SERVICE_COMPLETED` → `CUSTOMER_CONFIRMATION_PENDING` → (confirmed) `PAYMENT_PENDING` / `PARTIALLY_PAID` / `PAID` (see Stage 9)

**Failure cases**: customer does not confirm within a configurable window → auto-confirmed with a system note (configurable — some businesses require it to stay pending indefinitely instead; see [21-open-decisions-and-clarifications.md](21-open-decisions-and-clarifications.md)) so the case doesn't stall the pipeline indefinitely by default.

## Stage 9 — Invoice & Payment

**Actor**: Technician (collection at site) or Finance/Accountant (office processing).

**Required fields**: payment method, amount, reference (for UPI/card/bank transfer/cheque), proof (image/receipt for cash where applicable).

**Status before → after**: `PAYMENT_PENDING` → `PARTIALLY_PAID` (if less than total) → `PAID`

**Side effects**: Invoice generated (converted from the approved Estimate/Proforma, or generated fresh if no estimate stage occurred), Payment Receipt generated on each payment event, customer ledger updated, vendor payout accrual updated if a Vendor performed the work.

**Failure cases**: payment gateway failure (once integrated) → status remains `PAYMENT_PENDING`, retry available, does not block the Service Request from being marked `SERVICE_COMPLETED` administratively (payment collection can lag completion for credit customers).

## Stage 10 — Post-Service Follow-up & Happy Call

**Actor**: Customer Support / Happy Call Executive (or automated call-scheduling task).

**Status before → after**: `PAID` (or `SERVICE_COMPLETED` for credit accounts) → `FOLLOW_UP_PENDING` → `HAPPY_CALL_PENDING` → `CLOSED`

**Required fields (Happy Call)**: outcome, customer satisfaction rating, remarks, reopen requested (Y/N), escalation required (Y/N), next follow-up date if needed.

**Side effects**: a `HappyCall` record is created and linked to the `ServiceRequest`; if `reopenRequested = true`, Stage 11 begins; if `escalationRequired = true`, an escalation is raised per [07-status-transition-matrix.md](07-status-transition-matrix.md) escalation rules.

**Failure cases**: customer unreachable for happy call → outcome `UNREACHABLE`, a retry task is scheduled per policy (default: 2 retries over 5 days before giving up and closing anyway with a flag).

## Stage 11 — Reopen

**Actor**: Customer (via app/call) or staff, within the eligibility window.

**Eligibility calculation**: `today <= serviceRequest.completedAt + reopenPolicy.windowDays`, where `reopenPolicy` resolves via the most-specific applicable rule (customer > contract > brand > product > service > branch > global), default 90 days globally — see [09-database-architecture.md](09-database-architecture.md) policy resolution.

**Required fields**: `reopenReason`, link to original `serviceRequestId`.

**Side effects**: a new `ServiceRequest` is created with `isReopen: true`, `originalServiceRequestId` set, and (default policy) routed back to the original assignee first before falling back to normal dispatch; `ReopenRecord` created capturing reopen count and reason; original assignee notified; warranty/charge applicability evaluated per policy (in-warranty reopens typically waive the visiting charge — configurable per Service).

**Failure cases**: outside eligibility window → reopen request is still logged (as a new independent Service Request, not linked) but flagged so staff/finance know standard charges apply, rather than silently rejecting the customer.

## Cross-Cutting Rules (apply to every stage)

- Every status change writes an `ActivityLog` entry and a timeline entry — no status field is ever overwritten without a paired history write (see [17-security-and-audit.md](17-security-and-audit.md)).
- Every notification trigger listed above degrades gracefully per [14-integration-architecture.md](14-integration-architecture.md) — a failed WhatsApp/email send never rolls back or blocks the underlying state change.
- Every "required field" above is enforced server-side regardless of what the client sends — see [18-error-handling-standards.md](18-error-handling-standards.md) for the validation-error contract.
