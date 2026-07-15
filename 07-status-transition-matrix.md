# 07 — Status Transition Matrix

Statuses are stored as data in the `statuses` and `status_transitions` masters (Config/Masters engine, [09-database-architecture.md](09-database-architecture.md)), keyed per `entityType`. The lists below are the **default seed data**, finalized from the prompt's suggested list plus the workflow in [06-complete-workflow-document.md](06-complete-workflow-document.md) — admins may add business-specific statuses later without a code change, but may not remove or rename the ones marked "system" below, since workflow/reporting code branches on them.

## 1. ServiceRequest Status List (final)

| Status | System? | Meaning |
|---|---|---|
| `NEW` | system | Just created, not yet reviewed |
| `NEEDS_MANUAL_BRANCH_ASSIGNMENT` | system | No branch coverage matched automatically |
| `ASSIGNED_TO_BRANCH` | system | Forwarded to a Branch, not yet to Sub-Branch/Team |
| `ASSIGNED_TO_SUB_BRANCH` | system | |
| `ASSIGNED_TO_TEAM` | system | |
| `ASSIGNED_TO_EMPLOYEE` | system | |
| `ASSIGNED_TO_VENDOR` | system | |
| `OUTSOURCED` | system | Assigned to an Outsourced Partner (non-panel vendor) |
| `REASSIGNMENT_REQUIRED` | system | Rejected or unaccepted within SLA |
| `ACCEPTED` | system | Assignee accepted |
| `APPOINTMENT_SCHEDULED` | system | |
| `RESCHEDULED` | system | |
| `CUSTOMER_UNAVAILABLE` | system | |
| `TECHNICIAN_EN_ROUTE` | system | |
| `TECHNICIAN_ARRIVED` | system | |
| `INSPECTION_STARTED` | system | |
| `INSPECTION_COMPLETED` | system | |
| `ESTIMATE_PENDING` | system | |
| `ESTIMATE_SHARED` | system | |
| `AWAITING_CUSTOMER_APPROVAL` | system | |
| `ESTIMATE_APPROVED` | system | |
| `ESTIMATE_REJECTED` | system | |
| `PARTS_PENDING` | system | |
| `WORK_STARTED` | system | |
| `WORK_IN_PROGRESS` | system | |
| `ON_HOLD` | system | |
| `SERVICE_COMPLETED` | system | |
| `CUSTOMER_CONFIRMATION_PENDING` | system | |
| `PAYMENT_PENDING` | system | |
| `PARTIALLY_PAID` | system | |
| `PAID` | system | |
| `FOLLOW_UP_PENDING` | system | |
| `HAPPY_CALL_PENDING` | system | |
| `CLOSED` | system | Terminal (success) |
| `REOPENED` | system | Terminal-of-original, spawns a new linked Service Request |
| `ESCALATED` | system, overlay flag | Can co-exist with any active status — see §4 |
| `CANCELLED` | system | Terminal (customer/business initiated before completion) |

**Note**: `ESCALATED` is modeled as a boolean overlay (`isEscalated: true` + `escalationReason`) rather than a mutually-exclusive status, because a request can be, e.g., both `WORK_IN_PROGRESS` and escalated simultaneously. All other statuses in the table are mutually exclusive (`status` field, single value).

## 2. ServiceRequest Transition Table & Role Gating

| From | To | Allowed actor(s) | Trigger |
|---|---|---|---|
| — | `NEW` | Customer, Call Executive, Sales Executive | Create |
| `NEW` | `NEEDS_MANUAL_BRANCH_ASSIGNMENT` | system (auto) | No coverage match |
| `NEW` / `NEEDS_MANUAL_BRANCH_ASSIGNMENT` | `ASSIGNED_TO_BRANCH` | Super Admin, Admin | Dispatch |
| `ASSIGNED_TO_BRANCH` | `ASSIGNED_TO_SUB_BRANCH` | Branch Manager, Admin, Super Admin | Forward |
| `ASSIGNED_TO_SUB_BRANCH` | `ASSIGNED_TO_TEAM` / `ASSIGNED_TO_EMPLOYEE` | Sub-Branch Admin, Branch Manager, Admin, Super Admin | Assign |
| any `ASSIGNED_TO_*` / `NEW` (bypass) | `ASSIGNED_TO_VENDOR` / `OUTSOURCED` | Admin, Super Admin (bypass); Branch Manager (within branch) | Outsource |
| `ASSIGNED_TO_*` / `OUTSOURCED` | `ACCEPTED` | the assignee | Accept |
| `ASSIGNED_TO_*` / `OUTSOURCED` | `REASSIGNMENT_REQUIRED` | the assignee (reject), or system (SLA timeout) | Reject / timeout |
| `REASSIGNMENT_REQUIRED` | any `ASSIGNED_TO_*` / `OUTSOURCED` | same actors as original dispatch | Re-dispatch |
| `ACCEPTED` | `APPOINTMENT_SCHEDULED` | assignee, Call Executive | Schedule |
| `APPOINTMENT_SCHEDULED` | `RESCHEDULED` | assignee, Call Executive, Customer (within reschedule policy) | Reschedule |
| `APPOINTMENT_SCHEDULED` / `RESCHEDULED` | `TECHNICIAN_EN_ROUTE` | Technician/Employee | Start travel |
| `TECHNICIAN_EN_ROUTE` | `TECHNICIAN_ARRIVED` | Technician/Employee | Check-in |
| `TECHNICIAN_ARRIVED` | `CUSTOMER_UNAVAILABLE` | Technician/Employee | Customer not present |
| `CUSTOMER_UNAVAILABLE` | `APPOINTMENT_SCHEDULED` / `RESCHEDULED` | Call Executive, Technician | Re-attempt |
| `TECHNICIAN_ARRIVED` | `INSPECTION_STARTED` | Technician/Employee | Begin diagnosis |
| `INSPECTION_STARTED` | `INSPECTION_COMPLETED` | Technician/Employee | Diagnosis recorded |
| `INSPECTION_COMPLETED` | `ESTIMATE_PENDING` | system (if estimate required by Service config) | |
| `INSPECTION_COMPLETED` | `WORK_STARTED` | Technician/Employee | If estimate not required |
| `ESTIMATE_PENDING` | `ESTIMATE_SHARED` | Technician, Branch/Finance | Estimate sent |
| `ESTIMATE_SHARED` | `AWAITING_CUSTOMER_APPROVAL` | system (auto on send) | |
| `AWAITING_CUSTOMER_APPROVAL` | `ESTIMATE_APPROVED` / `ESTIMATE_REJECTED` | Customer | Approve/reject |
| `ESTIMATE_APPROVED` | `WORK_STARTED` | Technician/Employee | |
| `ESTIMATE_REJECTED` | `CLOSED` / `CANCELLED` | Branch Manager, Admin | If customer declines all work |
| `WORK_STARTED` | `WORK_IN_PROGRESS` | Technician/Employee | |
| `WORK_IN_PROGRESS` | `PARTS_PENDING` / `ON_HOLD` | Technician/Employee | Blocked |
| `PARTS_PENDING` / `ON_HOLD` | `WORK_IN_PROGRESS` | Technician/Employee | Resume |
| `WORK_IN_PROGRESS` | `SERVICE_COMPLETED` | Technician/Employee | Work finished |
| `SERVICE_COMPLETED` | `CUSTOMER_CONFIRMATION_PENDING` | system (auto) | |
| `CUSTOMER_CONFIRMATION_PENDING` | `PAYMENT_PENDING` | Customer (confirm) or system (auto-confirm timeout, configurable) | |
| `PAYMENT_PENDING` | `PARTIALLY_PAID` / `PAID` | Technician, Finance/Accountant | Payment recorded |
| `PARTIALLY_PAID` | `PAID` | Finance/Accountant | Balance settled |
| `PAID` | `FOLLOW_UP_PENDING` | system (auto, per Service follow-up policy) | |
| `FOLLOW_UP_PENDING` | `HAPPY_CALL_PENDING` | Customer Support / system schedule | |
| `HAPPY_CALL_PENDING` | `CLOSED` | Happy Call Executive | Happy call complete |
| `CLOSED` | `REOPENED` (spawns new SR) | Customer, Call Executive (within reopen policy window) | Reopen |
| any pre-`PAID` status | `CANCELLED` | Customer (policy-gated), Branch Manager, Admin, Super Admin | Cancel |
| any active status | `+ isEscalated=true` | Branch Manager+, or system (SLA breach) | Escalate |

Statuses not listed as reachable from a given "From" are **not permitted** — enforced server-side on every status-changing endpoint (see [11-complete-api-contracts.md](11-complete-api-contracts.md) `PATCH /service-requests/{id}/status`).

## 3. Lead Stage Transition Table

| From | To | Allowed actor(s) |
|---|---|---|
| — | `NEW` | Any role that can create a lead |
| `NEW` | `CONTACT_ATTEMPTED` | Owner |
| `CONTACT_ATTEMPTED` | `CONNECTED` / `NOT_INTERESTED` / `INVALID` | Owner |
| `CONNECTED` | `REQUIREMENT_COLLECTED` | Owner |
| `REQUIREMENT_COLLECTED` | `QUALIFIED` / `LOST` | Owner |
| `QUALIFIED` | `ESTIMATE_REQUIRED` / `CONVERTED` (direct if no estimate needed) | Owner |
| `ESTIMATE_REQUIRED` | `ESTIMATE_SHARED` | Owner |
| `ESTIMATE_SHARED` | `NEGOTIATION` / `CONVERTED` / `LOST` | Owner |
| `NEGOTIATION` | `CONVERTED` / `LOST` / `FOLLOW_UP` | Owner |
| `FOLLOW_UP` | any earlier active stage | Owner |
| any | `DUPLICATE` | Owner, Sales Manager (merge detection) |
| `CONVERTED` | *(terminal — a Service Request or Customer now exists)* | — |
| `LOST` / `NOT_INTERESTED` / `INVALID` | *(terminal, reopenable to `FOLLOW_UP` manually)* | Sales Manager, Admin |

## 4. Escalation Rules

| Trigger | Escalates to |
|---|---|
| Acceptance SLA breached (no accept/reject within configured window) | next level up (Sub-Branch → Branch → Admin) |
| Overall Service Request SLA breached (per Service config) | Branch Manager, then Admin if unresolved |
| Repeated rejection (configurable threshold, default 2) | Branch Manager |
| Customer marks dissatisfied on Happy Call | Customer Support Executive → Branch Manager if unresolved in configured window |
| Reopen count exceeds configurable threshold on same product | Branch Manager (signals a recurring/unresolved defect) |

## 5. Estimate / Proforma Invoice / Invoice Status Sets

| Entity | Statuses |
|---|---|
| Estimate | `DRAFT` → `SHARED` → `APPROVED` / `REJECTED` → `EXPIRED` (if not actioned within validity window) → `CONVERTED` (to Proforma or directly to Invoice) |
| Proforma Invoice | `DRAFT` → `SHARED` → `ACCEPTED` → `CONVERTED` (to Invoice); or `CANCELLED` |
| Invoice | `DRAFT` → `ISSUED` → `PARTIALLY_PAID` → `PAID`; or `CANCELLED` (only before any payment recorded — see [16-pdf-and-financial-documents.md](16-pdf-and-financial-documents.md) for revision-vs-cancel rules) |

## 6. Vendor Assignment Sub-Statuses (within `ASSIGNED_TO_VENDOR` / `OUTSOURCED`)

Vendor-side acceptance mirrors §2 (`ACCEPTED`/`REASSIGNMENT_REQUIRED`) but additionally tracks `vendorAcknowledgedAt` separate from `technicianAssignedAt`, since a Vendor Owner accepting the job and a Vendor Technician being assigned within that vendor's team are two distinct events, both logged in `AssignmentHistory`.
