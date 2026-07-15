# 03 — Screenshot and Excel Analysis

**Status note**: no image files were actually attached in the conversation that produced this document — only a detailed text description of two Excel screenshots' contents. This document is built from that text description. If the actual screenshots are provided later, this document should be revisited to verify exact column names/order and to catch any field the text description omitted, before the Excel import template ([15-excel-import-export-specification.md](15-excel-import-export-specification.md)) is finalized against real sample data.

## 1. What the Two Screenshots Represent

**Screenshot 1 — narrow customer/call record**: reads as one case's full resolution trail — customer identity, address, appliance identity (brand/product/product type/age), symptoms, actual defect found, solution type, part details, actual solution, remarks, call-closed-by/date, happy-call-by/date/status/remarks — flattened into a single row per case.

**Screenshot 2 — wide call-management sheet**: an operational export, one row per call/visit event, repeating customer + product + service + part + solution + closing + happy-call columns per row rather than normalizing shared data (the same customer's name/address repeats on every row for every call they've ever had).

## 2. Why This Cannot Become One MongoDB Collection

A flat sheet works for humans scanning rows in Excel but breaks a database in three ways: (1) customer/product data is duplicated on every row, so a customer's address correction requires updating every historical row instead of one record; (2) history (happy call outcome, solution used) is stored as columns that get overwritten on the "current" case rather than preserved as an append-only trail; (3) reporting questions like "how many AC repairs did Branch X close this month" require string-matching flat text columns instead of querying structured, indexed fields.

## 3. Normalized Field-to-Module Mapping

| Excel column group (from description) | Normalized entity | Field(s) | Data type | Required? |
|---|---|---|---|---|
| Customer name, second name, contact number, alternate number | `customers` | `name`, `contacts[]` | string | name required, alt optional |
| State, city, area, address 1/2, landmark, pin code, country | `customers.addresses[]` | as listed | string | address line 1 + pin code required |
| Brand, product, product type, product age, serial number, model number | `customer_products` | `brandId`, `productTypeId`, `modelNumber`, `serialNumber`, `purchaseDate`/derived age | string/ObjectId/date | brand + product type required |
| Call ID, call date, call time, direction, source, call type, priority, created by, assigned person, notes, attachments, recording | `calls` | as named per [09-database-architecture.md](09-database-architecture.md) `calls` schema | mixed | call date + direction required |
| Initial complaint, requirement, symptoms | `calls.details.initial` / `service_requests.symptoms[]` | text/master refs | string/array | at least one symptom or free text required |
| Actual defect found, type of solution, actual solution, service remarks | `service_visits.inspection` / `service_visits.workNotes` | as named | string/masterRef | required at inspection-completed stage |
| Part name, part quantity, part price | `service_visits.parts[]` | `partId`/`name`, `qty`, `unitPrice` | masterRef/number | required if parts used |
| Before/after images | `service_visits.beforeImages[]/afterImages[]` | file refs | array of ObjectId | per Service's `requiredImages` config |
| Call closed by, call closed date | `service_requests` (derived from status-history) / `activity_logs` | actor + timestamp of `CLOSED` transition | ref/date | system-derived, not manually entered |
| Happy call by, happy call date, happy call status, happy call remarks | `happy_calls` | `performedBy`, `callDate`, `status`, `remarks` | ref/date/enum/string | required once happy call performed |
| Payment status | `service_requests.status` (payment sub-states) / `payment_receipts` | enum/document | — | required at payment stage |

## 4. Duplicate-Field Identification

Fields that appear once per customer in the flat sheet but repeat across every one of that customer's rows (and must be normalized into `customers`/`customer_products`, never re-entered per call): customer name, contact numbers, full address, brand, product type, model/serial number. Fields that are genuinely one-per-event and correctly belong on `calls`/`service_visits`/`happy_calls`: call date/time, defect found, solution, parts used, remarks, closing info, happy-call outcome.

## 5. Searchable / Filterable / Exportable Field Classification

| Classification | Fields |
|---|---|
| Searchable (free-text `q` param) | Customer name, mobile, service request number, call number, serial number |
| Filterable | Branch, status, priority, service category, brand, product type, date range, assignee, happy-call status |
| Exportable (every list screen, per [10 §5](10-api-standards.md)) | All fields in the relevant list endpoint's flattened response shape ([12-frontend-data-contracts.md](12-frontend-data-contracts.md) §5) |
| History-only (not on the main Service Request document) | Happy call outcome/remarks (on `happy_calls`), assignment trail (on `assignment_history`), status change trail (on `activity_logs`) — see [09-database-architecture.md](09-database-architecture.md) §7 for why |

## 6. Excel Import Template (legacy migration + ongoing bulk entry)

One workbook per entity, not one giant sheet: `customers_import.xlsx`, `service_requests_import.xlsx` (references customer by mobile number or a generated temp ID for cross-sheet linking), `calls_import.xlsx`. Column headers match the API field names in [11-complete-api-contracts.md](11-complete-api-contracts.md) exactly (not the original Excel's human-readable headers) so the same validation schema used by the API validates import rows. Full column-by-column template finalized once real sample files are available (see status note above).

## 7. Invalid-Row Error Report Format

Import returns, per [10-api-standards.md](10-api-standards.md) envelope:
```json
{
  "success": true,
  "message": "Import processed: 142 succeeded, 8 failed",
  "data": {
    "successCount": 142,
    "failureCount": 8,
    "failures": [
      { "row": 17, "field": "contacts[0].mobile", "code": "INVALID_MOBILE", "message": "Enter a valid mobile number", "rawValue": "98765" }
    ]
  },
  "meta": null,
  "errors": null
}
```
Import is **partial-success by default** (valid rows commit, invalid rows are reported) unless the admin explicitly selects an all-or-nothing mode — matching real-world legacy data quality, where a strict all-or-nothing import on an 8000-row legacy sheet would be unusable.

## 8. Legacy Excel → Database Migration Mapping

Migration runs the same import engine (§6/§7) against the historical sheets, in this entity order to satisfy foreign keys: `customers` → `customer_products` → `calls` (initial rows create the base call record) → `service_requests` (backfilled from the same rows, `status: CLOSED` for historical completed cases, with `createdAt`/`closedAt` backdated to the original dates rather than defaulted to migration time) → `happy_calls`. A dry-run mode (`?dryRun=true`) reports the full error set without committing anything, so the legacy sheet's real data-quality issues surface before a production migration is attempted.

Full detail to be finalized once actual sample Excel files are provided.
