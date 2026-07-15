# 02 — Product Requirement Document (PRD)

This document translates the business requirements in [01-business-requirements-document.md](01-business-requirements-document.md) into product-level user stories, acceptance criteria, and release prioritization. Full field-level and API-level detail lives in later docs ([11-complete-api-contracts.md](11-complete-api-contracts.md), [rohit/07-form-field-specifications.md](rohit/07-form-field-specifications.md)); this document is the bridge between "why" and "exactly how."

## 1. Release Prioritization

| Priority | Meaning |
|---|---|
| **P0 — MVP** | Platform is not usable as a service business system without this |
| **P1 — Fast follow** | Needed for full operational parity with the current (Excel-based) process, ships shortly after P0 |
| **P2 — Growth** | Marketing, AI, advanced reporting — value-add, not blocking |
| **P3 — Later phase** | AMC/contracts, advanced analytics — architecturally reserved, not built now |

Mapped in detail per module in [22-phase-wise-development-plan.md](22-phase-wise-development-plan.md); this section gives the product-level cut.

| Module | Priority |
|---|---|
| Auth & RBAC, Organization hierarchy | P0 |
| Customer & Product management | P0 |
| Dynamic Service Catalog & Masters engine | P0 |
| Call Management (all sub-types) | P0 |
| Service Request Workflow Engine | P0 |
| Field Execution (technician/vendor app) | P0 |
| Estimate/Invoice/Payment core | P0 |
| Notification engine (in-app, email, push) | P0 |
| Lead Management | P1 |
| Vendor Management (full lifecycle: payouts, ledger, docs) | P1 |
| Reopen Management | P1 |
| Excel Import/Export | P1 |
| WhatsApp (AiSensy) integration | P1 |
| Reports & Dashboards (core set) | P1 |
| Audit Log & Timeline | P0 (cross-cutting, cannot be deferred without losing traceability from day one) |
| Email marketing campaigns | P2 |
| WhatsApp marketing campaigns | P2 |
| AI features (Gemini/OpenAI) | P2 |
| Advanced reports/analytics | P2 |
| AMC / Contract management | P3 |

## 2. User Stories & Acceptance Criteria (representative set)

Full story set per module lives in each module's section of [04-modules-and-feature-list.md](04-modules-and-feature-list.md). This section illustrates the acceptance-criteria bar every story must meet — concrete, testable, no vague verbs.

### 2.1 Customer books a service (Customer App)

**As a** customer, **I want to** book a service for a specific appliance at a specific address and time, **so that** I don't need to call in.

Acceptance criteria:
- Customer selects a Service (from active, published services only) → selects or adds a Product (brand/model/serial optional) → selects an existing or new Address → optionally uploads issue images/video → selects a preferred time slot from availability the backend actually has open for that pin code/branch.
- On submit, a Service Request is created in status `New`, a Call record of type `Customer App Booking` is NOT required (booking is not a call) but an activity-timeline entry is created, and an acknowledgement notification (push + optionally email/WhatsApp per consent) is sent within the request/response cycle's async job, not blocking the booking response.
- If no branch/vendor covers the customer's pin code for the selected service, the customer sees a clear "not serviceable in your area" state before submission, not after.
- Full API/status detail in [06-complete-workflow-document.md](06-complete-workflow-document.md) §"Customer books service."

### 2.2 Admin dispatches a Service Request

**As a** Branch Manager, **I want to** assign a new Service Request to a Team or Employee (or forward to Sub-Branch, or outsource to a Vendor), **so that** work gets scheduled.

Acceptance criteria:
- Manager sees only requests scoped to their Branch (data-level permission), can view suggested assignees ranked by the assignment-rule engine (skill/coverage/availability/workload), and can override to any valid assignee.
- On assignment, status moves per the transition matrix ([07-status-transition-matrix.md](07-status-transition-matrix.md)), the assignee is notified, an assignment-history entry is written (assigner, assignee, timestamp, reason if reassignment), and the request appears in the assignee's queue (admin list or mobile job list) in real time via Socket.IO.
- Rejecting an assignment requires a reason, reverts status to `Reassignment Required`, and notifies the original assigner.

### 2.3 Technician executes a job (Vendor/Employee App)

**As a** technician, **I want to** record diagnosis, parts, and completion evidence for my assigned job, even with poor connectivity, **so that** the job closes correctly and I get credit for the work.

Acceptance criteria:
- Actions (accept, start travel, arrive, inspect, diagnose, add parts, upload images, complete) are captured locally first and queued for sync; the UI never blocks on network availability for these actions (offline-first — see [rohit/06-vendor-app-screen-list.md](rohit/06-vendor-app-screen-list.md) and [manish/09-vendor-app-functional-plan.md](manish/09-vendor-app-functional-plan.md)).
- Completion requires at minimum: diagnosis notes, at least one after-service image (unless the service category is configured not to require images), and either an OTP/signature/customer-confirmation per the service's configured completion-proof requirement.
- On successful sync, the Service Request status advances per the transition matrix and the customer is notified.

### 2.4 Admin configures a new service (Dynamic Catalog)

**As an** Admin, **I want to** add "Water Purifier Service" as a new service without asking a developer, **so that** the business can expand offerings independently.

Acceptance criteria:
- Admin creates the service with category, applicable brands/products, pricing (base/visiting/inspection/emergency), tax rate, warranty period, reopen period override, required checklist/images/documents, SLA, and assignment rules — entirely through admin UI, with no code or deployment involved.
- The new service is immediately bookable (customer app) and assignable once marked active, and does not require any change to the Service Request schema (services are data, not code).

## 3. MVP Definition

MVP is reached when: a customer can book a service (app or call-entry by staff) → it is assigned through the hierarchy or to a vendor → a technician executes it end-to-end on the mobile app including offline capture → an invoice/payment is recorded → the record and its full history are visible and reportable to admin. This corresponds to all P0 rows in §1.

## 4. Non-Goals for This PRD

This document does not specify pixel-level UI (see `rohit/` docs), does not specify database schemas (see [09-database-architecture.md](09-database-architecture.md)), and does not specify exact API payloads (see [11-complete-api-contracts.md](11-complete-api-contracts.md)).
