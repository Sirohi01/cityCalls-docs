# 00 — Project Overview

## 1. What is CityCalls

CityCalls is a production-grade, end-to-end **service management ecosystem** for a multi-branch home/commercial appliance service business (AC repair, refrigerator repair, washing machine repair, plumbing, electrical, cleaning, appliance installation, and any future service category added without code changes).

It covers the complete lifecycle of a service business: a customer calls in or books through the app → the request is qualified, assigned, and scheduled → a technician (employee or vendor) visits, diagnoses, and executes the work → the customer is invoiced and pays → the business follows up, runs a happy call, and tracks reopens and warranty — all while marketing, reporting, and communication (email/WhatsApp/notifications) run alongside it.

CityCalls is **not** a simple booking form. It is an operations platform combining:

- CRM-style call and lead management
- Field-service work-order execution (mobile-first for technicians/vendors)
- Financial document chain (estimate → proforma → invoice → payment)
- Configurable organizational hierarchy and assignment engine
- Marketing automation (WhatsApp, email)
- AI-assisted operations (optional)
- Reporting, dashboards, and audit trail

## 2. Objectives

1. Give the business a single system of record for every customer interaction — call, lead, service request, invoice, and follow-up — instead of spreadsheets.
2. Let non-technical admins configure services, pricing, statuses, rules, and templates **without a code deployment**.
3. Give technicians and vendors a mobile workflow that captures diagnosis, parts, images, and completion evidence in the field, including with unreliable connectivity.
4. Give customers self-service booking, tracking, and payment.
5. Preserve complete history — assignment, status, calls, reopens — for every record, for accountability and reporting.
6. Keep every third-party integration (Cloudinary, SMTP, AiSensy, Gemini/OpenAI, payment gateway, SMS) optional, so their failure or absence never blocks the core business workflow.
7. Let two developers build backend and frontend independently and merge without conflict, by fixing contracts before implementation.

## 3. Who Uses CityCalls

| User | Surface | Core needs |
|---|---|---|
| Super Admin / Admin | Admin web | Full configuration, oversight, reports, override authority |
| Branch / Sub-Branch Manager | Admin web | Manage their branch's requests, team, vendors, performance |
| Call Executive / CSE | Admin web | Log calls, create leads/service requests, follow up |
| Sales / Marketing Executive | Admin web | Leads, campaigns, conversion tracking |
| Finance / Accountant | Admin web | Estimates, invoices, payments, outstanding, payouts |
| Employee / Technician | Vendor/Employee mobile app (Flutter) | Assigned jobs, execution workflow, collections |
| Vendor Owner / Manager / Technician | Vendor/Employee mobile app (Flutter) | Same as employee, scoped to vendor's own jobs/finance |
| Customer (individual/business) | Customer mobile app (Flutter) | Book, track, pay, review history, raise complaints |

## 4. Scope Summary

**In scope for the platform (see [04-modules-and-feature-list.md](04-modules-and-feature-list.md) for full detail):**
Auth & RBAC, organization hierarchy (Branch/Sub-Branch/Team), employee & vendor management, customer management, dynamic service/product/brand catalog, dynamic configuration/masters engine, call management (initial/requirement/pre-service/visit/post-service/happy-call), lead management, service-request workflow engine, field execution, reopen management, estimate/proforma/invoice/payment chain, notification engine (in-app/push/email/WhatsApp/SMS-ready), email & WhatsApp marketing, optional AI features, file management (Cloudinary or fallback), reports & dashboards, audit log & activity timeline, Excel import/export.

**Architecturally reserved but built in a later phase:** AMC / contract management. The data model will reserve the Customer↔Contract↔ServiceRequest relationship now (see [09-database-architecture.md](09-database-architecture.md)) so this doesn't require a breaking migration later.

**Explicitly out of scope for v1** (see [21-open-decisions-and-clarifications.md](21-open-decisions-and-clarifications.md) for anything still being decided):
- Multi-tenant SaaS (platform serves one company with multiple branches, not multiple independent companies)
- Multi-currency / multi-language (assumed India-only, INR, single language unless stated otherwise)
- Payment gateway and SMS provider selection (built behind generic interfaces, provider TBD)

## 5. Technology Stack

| Layer | Technology |
|---|---|
| Admin Panel | Next.js, TypeScript, Tailwind CSS, TanStack Query, Axios, React Hook Form, Zod |
| Backend API | Node.js, Express.js, TypeScript, MongoDB, Mongoose, JWT |
| Customer Mobile App | Flutter (Dart) |
| Vendor/Employee Mobile App | Flutter (Dart) |
| Real-time | Socket.IO (technician location, live status push, live admin dashboard) — required from day one |
| File Storage | Cloudinary (optional, pluggable) with fallback bucket storage |
| Email | SMTP (configurable, optional, multi-account) |
| WhatsApp | AiSensy (optional) |
| AI | Gemini and/or OpenAI (optional, feature-flagged) |
| Contract format between backend and all clients | OpenAPI / JSON Schema (single source of truth — see §7) |

Full rationale in [08-system-architecture.md](08-system-architecture.md).

## 6. Team & Ownership

CityCalls is built by two developers working in parallel from shared, frozen documentation.

| | Manish | Rohit |
|---|---|---|
| Owns | Backend, API, DB schemas, auth/RBAC, business logic, workflow engine, dynamic configuration system, admin panel functional wiring, both mobile apps' functional setup, all integrations, security, logging, deployment, final merge | Admin panel UI, customer app UI, vendor/employee app UI, reusable components, forms/tables/dashboards, responsive layouts, frontend validation |
| Stack | Node/Express/TS/Mongo, Flutter functional glue | Next.js/TS/Tailwind, Flutter/Dart UI |
| Builds against | Approved workflow/status/DB docs | Approved API contracts + mock JSON (identical structure to real responses) |

Full working agreement in [coordination/01-team-working-agreement.md](coordination/01-team-working-agreement.md). Code ownership boundaries in [coordination/03-code-ownership.md](coordination/03-code-ownership.md).

## 7. Why Contracts Come Before Code

CityCalls is **five independent Git repositories** (`citycalls-docs`, `citycalls-api`, `citycalls-admin-web`, `citycalls-customer-mobile`, `citycalls-vendor-mobile`), not a monorepo, and there is **no shared code package of any kind** between them — not even between the two Flutter apps. This was an explicit user decision (2026-07-15) prioritizing full independence (each repo buildable, testable, and deployable without touching another) over automatic type-sharing. Full rationale and trade-off in [manish/01-project-and-repository-setup.md](manish/01-project-and-repository-setup.md) §2.

Because nothing is shared code, consistency across all four client/backend repos is maintained entirely through documentation and process, not the compiler:

- **OpenAPI/JSON Schema, canonically stored in `citycalls-docs`**, is the single source of truth for every request/response shape, enum, and validation rule.
- Each of the other four repos pulls its own **local copy** of the spec via a sync script and generates its own local types from it (TS for admin-web, Dart independently for each mobile app, Zod validation for the API) — see [manish/01-project-and-repository-setup.md](manish/01-project-and-repository-setup.md) §3.
- Manish freezes a contract change in `citycalls-docs` first (documented in [11-complete-api-contracts.md](11-complete-api-contracts.md) and [12-frontend-data-contracts.md](12-frontend-data-contracts.md)) and ships mock JSON before the real endpoint exists.
- Rohit builds against the mock in each consuming repo; when Manish ships the real endpoint, the response shape is identical, so no UI rework is needed.
- Neither developer changes a contract without updating `citycalls-docs` first and notifying the other — see [coordination/08-change-request-process.md](coordination/08-change-request-process.md). Propagation to each consuming repo is a deliberate, tracked, asynchronous step, not automatic.

## 8. How This Documentation Set Is Organized

`docs/` is the entire content of the `citycalls-docs` repository:

```
docs/
├── 00-24 ......... shared documents — architecture, workflows, contracts, standards (this folder)
├── openapi/ ....... the canonical OpenAPI spec, synced into every other repo (see §7)
├── manish/ ........ backend/integration/mobile-functional implementation plans
├── rohit/ .......... UI implementation plans, screen inventories, component specs
└── coordination/ ... process docs governing how the two developers work together across all five repos
```

Read order for shared docs and dependency rationale are in [22-phase-wise-development-plan.md](22-phase-wise-development-plan.md) and were agreed with the user before writing began. In short: business requirements → naming conventions (locked early since everything downstream depends on it) → roles → workflow → status matrix → architecture → database → API standards → API contracts → frontend contracts → everything else.

No document is considered final until the user has explicitly reviewed and approved it. No application code is written until every Phase 0 document (numbers 00–24, plus `coordination/`, `manish/`, `rohit/`) is complete and approved.

## 9. Core Glossary (expanded fully in [coordination/06-naming-conventions.md](coordination/06-naming-conventions.md))

| Term | Meaning |
|---|---|
| **Call** | A single phone-call event (incoming/outgoing) logged against a customer, may or may not result in a Lead or Service Request |
| **Lead** | A pre-sale opportunity being qualified, not yet a confirmed service booking |
| **Service Request** | A confirmed job — the core work-order entity tracked through the status/workflow engine |
| **Service Visit** | One field visit against a Service Request (a request may need multiple visits) |
| **Happy Call** | A post-completion follow-up call verifying customer satisfaction, distinct entity linked to the Service Request |
| **Reopen** | A new linked cycle on a previously closed Service Request, governed by a configurable eligibility window |
| **Branch / Sub-Branch** | Organizational units below the company; a Service Request is scoped to a Branch/Sub-Branch or a Vendor |
| **Vendor / Outsourced Partner** | An external party that can receive assignments in parallel to the internal hierarchy |
| **Master** | An admin-configurable reference list (service categories, symptoms, defects, etc.) — see the Config/Masters engine in [09-database-architecture.md](09-database-architecture.md) |

## 10. Status of This Document

Approved pending user review. This is document 1 of the Phase 0 sequence — see [22-phase-wise-development-plan.md](22-phase-wise-development-plan.md) (not yet written) for the full plan, and the assistant's first-output response in conversation history for the agreed document order.
