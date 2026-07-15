# 01 — Business Requirements Document (BRD)

## 1. Business Context

CityCalls' operator runs (or manages, via branches and vendors) a multi-category home/commercial appliance service business. Revenue flows from: (a) per-visit service charges, (b) parts markup, (c) AMC/contract fees (later phase), (d) vendor-fulfilled jobs where the business takes a margin. The business currently runs on manual call logging and spreadsheets (see [03-screenshot-and-excel-analysis.md](03-screenshot-and-excel-analysis.md)); CityCalls replaces that with a system of record plus customer/vendor self-service apps.

## 2. Business Goals

| Goal | Success looks like |
|---|---|
| Eliminate spreadsheet-based call/service tracking | Every call, lead, and service request is created and tracked in-system, with no parallel Excel process |
| Reduce missed follow-ups | Follow-up calls, happy calls, and reopen windows are system-driven with reminders, not memory-driven |
| Improve first-time-fix and SLA adherence | SLA timers and escalation are automatic and reported on |
| Increase repeat/AMC revenue | Customer and service history is retained and queryable to drive upsell and renewal |
| Reduce assignment friction | Dispatch (branch/sub-branch/team/vendor) is rule-assisted, with manual override always available |
| Give management real-time visibility | Dashboards reflect live status, not end-of-day reconciliation |
| Support scaling to new service lines | New services/categories are added via configuration, not a code release |

## 3. Business Rules (binding constraints for design)

1. **No service type, status, or numbering scheme is hardcoded.** All are managed via the Config/Masters engine ([09-database-architecture.md](09-database-architecture.md)).
2. **A Service Request always belongs to exactly one Branch** (and optionally a Sub-Branch), OR is assigned directly to a Vendor/Outsourced Partner — never both simultaneously as owner, though a Branch can still outsource a request to a Vendor while remaining the owning branch for reporting.
3. **Assignment can bypass hierarchy.** Super Admin/Admin can assign directly to any Branch, Sub-Branch, Team, Employee, or Vendor regardless of normal routing.
4. **Every status change, assignment, and reassignment is logged immutably** — no in-place overwrite of history (see [17-security-and-audit.md](17-security-and-audit.md)).
5. **Reopen eligibility is time-boxed and configurable** at global, service, product, brand, contract, customer, and branch level, with the most specific applicable rule winning. Default 90 days.
6. **Every financial document (Estimate, Proforma, Invoice, Credit/Debit Note) is numbered per a configurable series** scoped by branch and financial year, and once issued, cannot be silently edited — only revised (new version) or cancelled.
7. **Integration failure never blocks core workflow.** A Service Request can be created, assigned, executed, and closed even if WhatsApp, email, Cloudinary, or AI is down or disabled.
8. **Role- and data-scope-based visibility is mandatory.** A Branch user never sees another Branch's customer contact details or financials unless explicitly permitted; a Vendor sees only their assigned jobs.
9. **Customer consent governs marketing communication.** WhatsApp/email marketing sends require an explicit, auditable consent state per channel per customer.

## 4. Stakeholders

| Stakeholder | Interest |
|---|---|
| Business Owner / Super Admin | Overall visibility, revenue, compliance |
| Branch/Sub-Branch Managers | Local operations, team performance, SLA |
| Call Executives / CSEs | Fast, low-friction call and lead entry |
| Technicians/Employees | Clear job instructions, simple field data capture |
| Vendors | Fair job visibility, timely payout, low-friction acceptance flow |
| Customers | Easy booking, transparency, trust in pricing and technician identity |
| Finance/Accounts | Accurate, auditable financial documents and reconciliation |
| Marketing | Segmentable, consent-respecting campaign tools |
| Developers (Manish, Rohit) | Unambiguous, stable contracts to build against |

## 5. High-Level Functional Requirements

Numbered FR groups map 1:1 to the modules in [04-modules-and-feature-list.md](04-modules-and-feature-list.md); this BRD states *why*, that document states *what* in feature-list form, and [06-complete-workflow-document.md](06-complete-workflow-document.md) states *how* step by step.

- **FR-1 Identity & Access** — the system must authenticate all users, enforce role- and data-scoped permissions, and support the full role list in [05-user-roles-and-permissions.md](05-user-roles-and-permissions.md).
- **FR-2 Organization** — the system must model Branch → Sub-Branch → Team → Employee, and Vendor/Outsourced Partner as a parallel structure, all admin-manageable.
- **FR-3 Customer & Product** — the system must retain customer identity, multiple addresses/contacts, and owned products/appliances, reusable across calls and requests.
- **FR-4 Dynamic Catalog** — the system must let admins define services, categories, pricing, SLA, and rules without deployment.
- **FR-5 Call Management** — the system must capture the full call lifecycle (initial, requirement, pre-service, visit, post-service, happy call) as discrete, linked, timestamped events.
- **FR-6 Lead Management** — the system must track pre-sale opportunities through configurable stages to conversion or loss.
- **FR-7 Service Request Workflow** — the system must drive a Service Request through a configurable status machine from creation to closure/reopen, with role-gated transitions.
- **FR-8 Field Execution** — the mobile field app must capture travel, arrival, diagnosis, parts, images, completion, and confirmation, with offline resilience.
- **FR-9 Financial Documents** — the system must generate Estimates, Proforma Invoices, Invoices, and Payment records with correct numbering, tax, and PDF/email/WhatsApp delivery.
- **FR-10 Notifications** — the system must notify the right party on the right channel for every significant event, without failing the triggering action if the channel fails.
- **FR-11 Marketing** — the system must support consent-respecting WhatsApp and email campaigns, independently toggle-able.
- **FR-12 AI Assistance** — the system must optionally use AI for summarization/classification/suggestion, never for irreversible actions without human approval.
- **FR-13 Reporting** — the system must provide role-scoped dashboards and exportable reports across all operational and financial data.
- **FR-14 Audit** — the system must log every sensitive action and expose a chronological timeline per record.

## 6. Non-Functional Requirements

| Category | Requirement |
|---|---|
| Availability | Core booking/assignment/execution flow must not depend on any optional integration being up |
| Performance | List/table endpoints must paginate; dashboards must not run unbounded aggregations on request path (pre-aggregate where needed) |
| Security | JWT auth, role + data-scope authorization on every endpoint, encrypted secrets, audit logging on sensitive actions — see [17-security-and-audit.md](17-security-and-audit.md) |
| Data integrity | Status transitions, numbering, and history writes must be atomic/transactional at the document level |
| Offline resilience | Vendor/employee field app must capture and queue actions offline, syncing when connectivity returns |
| Real-time | Technician location, live status changes, and admin dashboard counters must update via Socket.IO, not polling, from day one |
| Localization | India-only, INR, single language for v1 (see [21-open-decisions-and-clarifications.md](21-open-decisions-and-clarifications.md)) |
| Extensibility | New service categories, statuses, roles, and document types must be addable via configuration/masters, not schema migration, wherever the requirement says "dynamic" |

## 7. Assumptions

- Single company, multiple branches (not multi-tenant SaaS).
- India-centric business (GST-aware financial documents, INR).
- Technicians/vendors primarily operate via the Flutter mobile app, not the admin web.
- Existing operational data lives in Excel today and will be migrated per [15-excel-import-export-specification.md](15-excel-import-export-specification.md).

## 8. Out of Scope for v1

See [00-project-overview.md](00-project-overview.md) §4 and [21-open-decisions-and-clarifications.md](21-open-decisions-and-clarifications.md) for the authoritative, living list (multi-tenant SaaS, multi-currency/language, payment gateway/SMS provider selection, AMC module activation).
