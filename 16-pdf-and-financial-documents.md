# 16 — PDF and Financial Documents

## 1. Document Types & Conversion Chain

`Estimate → Proforma Invoice → Service Request (execution happens) → Invoice → Payment Receipt`, with `Credit Note`/`Debit Note` issued against an existing Invoice, and `Vendor Invoice`/`Vendor Payout Statement` as a parallel chain for vendor settlement. Status sets per document type are in [07-status-transition-matrix.md](07-status-transition-matrix.md) §5.

A **conversion** (e.g. `POST /estimates/{id}/convert`) creates a new document referencing the source (`convertedFromId`, `convertedFromType`) and carries line items forward — it never mutates the source document's own items, since the source remains the historical record of what was originally quoted.

## 2. Numbering

Per [coordination/06-naming-conventions.md](coordination/06-naming-conventions.md) §3 and the Numbering Engine in [09-database-architecture.md](09-database-architecture.md) §4 — branch + financial-year scoped series, admin-configurable prefix/reset behavior.

## 3. Document Structure (shared across Estimate/Proforma/Invoice)

- Header: document number, date, branch details (name, address, GSTIN), customer billing details, service address (if different from billing), place of supply.
- Line items: description, linked `partId`/service line, qty, unit price, tax rate, line total.
- Totals: subtotal, discount, CGST/SGST (intra-state) or IGST (inter-state, determined by comparing branch state to customer's place-of-supply state), round-off, grand total.
- Footer: terms and conditions (from a configurable template), notes, authorized signatory.

GST split logic (CGST+SGST vs IGST) is computed server-side from branch and customer state codes — never client-entered, to avoid tax-calculation drift between admin-web and mobile.

## 4. Revision vs. Cancellation

- **Before any payment is recorded**: an issued Invoice can be **cancelled** (status `CANCELLED`, reason required, original preserved for audit) and a fresh one issued.
- **After a payment is recorded against it**: the Invoice cannot be cancelled outright — corrections happen via a **Credit Note** (reducing) or **Debit Note** (adding), never by editing the original Invoice's line items in place. This mirrors standard accounting practice and keeps the financial audit trail intact.
- Estimates/Proforma Invoices (pre-execution, no payment yet) support a simpler **revision** flow: editing creates a new version (`revisionOf: originalId`, `version: 2`), original retained, both visible in history.

## 5. PDF Generation

Server-side PDF rendering (e.g. Puppeteer/HTML-to-PDF or a templating library — decided in [manish/](manish/) implementation planning, not user-facing) from an HTML template per document type, populated with the document's data. Generated PDFs are stored via the File Management module ([14-integration-architecture.md](14-integration-architecture.md)) and referenced by `pdfUrl` on the document; regenerating a PDF for an already-issued document reproduces identical content (deterministic from stored data), never silently changing historical numbers.

## 6. Delivery

`POST /{documentType}/{id}/share` with `channels: ["EMAIL","WHATSAPP"]` triggers PDF (re)generation if not cached, then dispatch via the Notification Engine ([13-notification-and-template-system.md](13-notification-and-template-system.md)) — subject to the same enabled/disabled and consent rules as any other notification.

## 7. Payment Application

A `payment_receipts` entry references its `invoiceId` and `amount`; an Invoice's `status` (`PARTIALLY_PAID`/`PAID`) is derived from the sum of its linked receipts, not manually set — this prevents an Invoice showing `PAID` while its receipt total doesn't actually cover the invoice amount. Overpayment (rare, e.g. rounding) is tracked as a customer credit balance rather than silently discarded.

## 8. Vendor Invoices & Payouts

Structurally parallel to the customer-facing chain but scoped to `vendorId` instead of `customerId`, generated either per completed Service Request (piece-rate vendors) or per settlement period (retainer vendors) per the vendor's configured `commissionModel` ([09-database-architecture.md](09-database-architecture.md) `vendors` schema). Penalties and security-deposit adjustments post as debit/credit entries against the vendor's ledger, not as edits to a past payout statement.

## 9. Printing

Every generated PDF is print-ready (A4, standard margins); the admin UI's "Print" action opens the same `pdfUrl` in a new tab rather than maintaining a separate print-CSS rendering path — one source of truth for the document's visual output.
