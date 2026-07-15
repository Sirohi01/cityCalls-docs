# 04 — Modules and Feature List

Complete feature inventory per module. This is the master checklist against which [22-phase-wise-development-plan.md](22-phase-wise-development-plan.md), [24-final-documentation-review-checklist.md](24-final-documentation-review-checklist.md), and both developers' task manifests are built. Each feature will get full field/API/workflow detail in the relevant downstream doc — this is the index, not the detail.

## M1 — Auth & RBAC
- Login (email/mobile + password), JWT access + refresh token, logout, forced logout (session revoke)
- Password reset (email link), password policy (min length/complexity, configurable)
- Role assignment (single role per user for v1 — see [21-open-decisions-and-clarifications.md](21-open-decisions-and-clarifications.md))
- Data-scope enforcement: own / team / sub-branch / branch / vendor / all-org
- Permission matrix management UI (Super Admin only)
- Session/device tracking, forced logout on password change
- OTP-based login for customer app (mobile OTP)

## M2 — Organization Hierarchy
- Branch CRUD, Sub-Branch CRUD, Team/Department CRUD
- Pin-code / city / state coverage mapping per Branch and Sub-Branch
- Service-category and brand mapping per Branch
- Working hours & holiday calendar per Branch (feeds SLA engine)
- Branch-level overrides: pricing, tax, invoice numbering series, notification templates
- Manager/employee/team/vendor assignment to Branch/Sub-Branch
- Active/inactive toggle with cascade rules (deactivating a branch reassigns or blocks new assignment)

## M3 — Employee & Technician Management
- Employee profile, role, skills, certifications, documents
- Branch/Sub-Branch/Team assignment
- Availability calendar, daily capacity limit
- Daily job list / route view
- Performance metrics (completion time, reopen rate, rating)

## M4 — Vendor & Outsourced Management
- Vendor profile: company details, contacts, GST/PAN/bank details, agreement + expiry
- Service area / pin-code coverage, services offered, brands/product types handled, skills
- Technician roster under vendor
- Commission model (fixed / service-wise), part pricing, payout rules, security deposit
- Vendor performance: acceptance rate, rejection rate, cancellation rate, completion time, reopen rate, customer feedback
- Vendor invoices, payouts, debit/credit notes, penalties, wallet/ledger
- Document-expiry alerts, active/inactive, blacklisting
- Vendor-scoped visibility (a vendor never sees another vendor's jobs or another branch's data)

## M5 — Customer Management
- Individual and business customer types, multiple contacts, multiple addresses
- Saved products/appliances per customer (brand/model/serial/purchase date/warranty)
- Duplicate detection (mobile, alternate mobile, email, address, GSTIN, business name)
- Service/call/lead/invoice/payment/feedback/complaint/reopen history (read, linked — not duplicated)
- Communication preferences & consent (WhatsApp/email/marketing, independently toggleable, audit-logged)
- Tags, segments, notes, uploaded documents
- Blacklist flag (with reason, audit-logged)

## M6 — Dynamic Service Catalog
- Service category / subcategory CRUD
- Per-service: brand/product/model applicability, complaint types, symptoms, possible defects, solution types, required skills, expected duration, base/visiting/inspection/emergency price, tax rate, warranty period, reopen period override, required documents/images, mandatory checklist, custom questions, SLA, assignment rules, cancellation/reschedule policy, notification templates, status-workflow variant, enabled/disabled state

## M7 — Dynamic Configuration / Masters Engine
- Generic master-list management for: categories, brands, models, symptoms, defects, solutions, parts, units, tax rates, priority levels, lead sources, call types, appointment slots, payment methods
- Numbering-series configuration per document type, per branch, per financial year
- Template management (email/WhatsApp/push/PDF) with variable binding
- Reopen/warranty/cancellation/reschedule policy configuration at global → branch → service → product → brand → contract → customer scope, most-specific-wins
- Role/permission/module-availability configuration
- Integration enable/disable toggles (Cloudinary, SMTP, AiSensy, AI, payment gateway, SMS)

## M8 — Call Management
- Initial Call Entry (see [03-screenshot-and-excel-analysis.md](03-screenshot-and-excel-analysis.md) for full field inventory)
- Requirement Call (confirms requirement, product, warranty, appointment preference)
- Pre-Service Call (confirms appointment/technician/address/charges, reschedule, unreachable handling)
- Service Visit Call/Update (visit/travel status, defect, solution, parts, images, remarks, payment status)
- Post-Service Follow-up Call (satisfaction, completion confirmation, repeat-issue flag, feedback)
- Happy Call (assigned-to, performed-by, outcome, satisfaction, reopen/escalation flags, next follow-up)
- Call recording attachment (where available), call notes, call outcome codes

## M9 — Lead Management
- Lead creation (all sources: website, app, call, WhatsApp, email, social, referral, vendor, branch, campaign, manual, Excel import, API)
- Lead stages (configurable, see [07-status-transition-matrix.md](07-status-transition-matrix.md))
- Lead assignment/ownership, scoring, priority
- Call/email/WhatsApp logs, notes, attachments, tasks, meetings linked to lead
- Duplicate detection & merge, bulk assign/update, import/export
- Conversion: Lead → Customer, Lead → Estimate, Lead → Service Request
- Lost-lead reason tracking

## M10 — Service Request & Workflow Engine
- Full status-transition engine (see [07-status-transition-matrix.md](07-status-transition-matrix.md))
- Assignment engine: hierarchy-based and rule-assisted (skill/coverage/availability/workload/SLA/distance), plus manual override and privileged bypass
- Reassignment with reason, rejection with reason, full history retained
- Appointment scheduling & rescheduling with policy enforcement
- Escalation triggers (SLA breach, repeated rejection, customer complaint)
- Cancellation with policy enforcement

## M11 — Field Execution (Vendor/Employee App)
- Accept/reject assignment, start travel, arrival/check-in
- Inspection & diagnosis capture (defect, symptoms, solution type)
- Parts (name, quantity, price), labour charge, estimated cost
- Before/after images, customer approval capture, work progress notes
- Completion: final remarks, OTP/signature/confirmation, payment collection
- Follow-up requirement flag, escalation raise, reassignment request
- Offline-first capture and background sync

## M12 — Reopen Management
- Eligibility calculation against the applicable (most-specific) policy
- Reopen reason, linkage to original Service Request, new-cycle creation
- Warranty and charge applicability at reopen
- Reassignment logic (default: same assignee first, configurable), original-assignee notification
- Reopen count, reopen SLA, full reopen history per request

## M13 — Estimate / Proforma / Invoice / Payment
- Document types: Estimate/Quotation, Proforma Invoice, Tax Invoice, Payment Receipt, Credit Note, Debit Note, Vendor Invoice, Vendor Payout Statement
- Dynamic numbering (financial year + branch series), items/parts/labour/charges/discounts/taxes (CGST/SGST/IGST)/round-off, terms, PDF, email/WhatsApp delivery, revision history, cancellation
- Conversion chain: Lead → Estimate → Proforma → Service Request → Invoice → Payment Receipt
- Payment methods (cash, UPI, card, bank transfer, gateway [pluggable], cheque, credit), partial/advance/balance, refunds, ledgers (customer/vendor), outstanding & settlement reports

## M14 — Notification Engine
- Trigger → template → channel → recipient-rule → delivery-log pipeline
- Channels: in-app, push, email, WhatsApp, SMS-ready
- Per-notification: enable/disable, retry policy, delivery/read/failure state
- Full trigger catalog in [13-notification-and-template-system.md](13-notification-and-template-system.md)

## M15 — WhatsApp Marketing (AiSensy)
- Integration settings, API key security, template sync/categories
- Transactional vs. marketing message separation, consent management
- Campaign creation, audience segmentation/filters, scheduling, personalization variables
- Delivery/read/failed status, retry logic, opt-out management, analytics
- Full graceful-degradation-when-disabled behavior in [14-integration-architecture.md](14-integration-architecture.md)

## M16 — Email Marketing & SMTP
- SMTP configuration (host/port/encryption/credentials/sender), test email, multiple accounts, branch-level override
- Transactional and marketing templates, campaigns, segmentation, scheduling
- Open/click tracking, bounce handling, unsubscribe, retry, logs, attachments

## M17 — AI Features (Gemini/OpenAI)
- Provider/model selection, API key storage, usage/token/cost limits, role permissions, logging
- Features: call-note summarization, complaint classification, service-category suggestion, lead scoring/summary, content generation (email/WhatsApp), reply suggestions, technician remark cleanup, sentiment analysis, duplicate-detection assist, dashboard/report insights, KB chatbot
- Enable/disable globally and per-feature; AI never performs irreversible actions without human approval

## M18 — File Management (Cloudinary + fallback)
- Upload types: issue images, product images, before/after service images, part images, vendor/employee documents, invoice attachments, recordings, videos, signatures, profile images
- File type/size restriction, folder naming convention, signed/secure upload, deletion, replacement, metadata, audit history
- Fallback storage strategy when Cloudinary is disabled (see [14-integration-architecture.md](14-integration-architecture.md))

## M19 — Reports & Dashboards
- Role-scoped dashboards (Super Admin, Admin, Branch, Sub-Branch, Employee, Vendor, Sales, Marketing, Finance, Support)
- Full report catalog: request volume/pipeline/SLA, branch/sub-branch/employee/vendor performance, category/brand/defect/solution/part analysis, repeat complaints, CSAT, happy-call results, lead funnel/source/conversion, campaign performance, estimate/invoice/payment/outstanding/payout reports, consent report
- Every table: search, filter, sort, pagination, column visibility, saved filters, date range, Excel/CSV export, print/PDF where applicable

## M20 — Audit Log & Activity Timeline
- Generic `activity_log` capturing user, role, action, module, record, old/new value, timestamp, IP, device, source app, reason
- Chronological timeline per Lead, Call, Customer, Service Request, Invoice, Vendor

## M21 — Excel Import/Export
- Generic import engine: template per entity, row-wise validation, error report, partial-success handling
- Generic export engine: column selection, filters applied, format (xlsx/csv)
- Legacy Excel-to-database migration mapping (see [15-excel-import-export-specification.md](15-excel-import-export-specification.md))

## M22 — Customer Mobile App (Functional)
- Register/login (OTP), profile & addresses, service browsing & booking, image/video upload, status tracking, technician info (permission-gated), reschedule/cancel (policy-gated), estimates/invoices/payments, service history, complaints, reopen request, ratings/feedback, support contact, push/email/WhatsApp updates

## M23 — Vendor/Employee Mobile App (Functional)
- Role-based login, assigned-job list, accept/reject, customer/location detail (permission-gated), travel/arrival, inspection/diagnosis/parts/pricing capture, before/after images, customer approval, work progress, completion (remarks, OTP/signature), payment collection, follow-up flag, escalation, reassignment request, daily task/history view

## M24 — Admin Panel (Functional Shell)
- Role-driven navigation, module availability per role/config, global search, notification center, settings area for all M7 configuration

## M25 — AMC / Contract Management *(P3 — architecturally reserved only)*
- Contract types, free/paid visit limits, product/service coverage, validity, included/excluded parts, renewal, contract invoice, usage history/balance, auto reminders

---

Full backend build sequencing of these modules is in [manish/07-api-development-sequence.md](manish/07-api-development-sequence.md); UI build sequencing in [rohit/13-ui-development-sequence.md](rohit/13-ui-development-sequence.md).
