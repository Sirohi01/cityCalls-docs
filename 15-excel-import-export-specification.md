# 15 — Excel Import/Export Specification

Generic engine, applied per-entity. Field-level detail for the two source screenshots is in [03-screenshot-and-excel-analysis.md](03-screenshot-and-excel-analysis.md); this document specifies the reusable mechanism.

## 1. Export Engine

`GET /api/v1/export/{entity}?format=xlsx|csv&{same filters as the entity's list endpoint}`. Reuses the exact filter set already defined for that entity's list endpoint (no separate "export filters") so "export what I'm currently looking at" always matches the on-screen table. Column set defaults to the list endpoint's flattened shape ([12-frontend-data-contracts.md](12-frontend-data-contracts.md) §5); admin can optionally select a column subset via `columns=field1,field2` — the selectable column list per entity is enumerated in that entity's contract, not free-form.

Large exports (row count above a configurable threshold, default 5,000) run as a background job; the endpoint returns `202` with a job ID, and the file is delivered via a notification with a signed download link once ready, rather than holding the HTTP connection open.

## 2. Import Engine

`POST /api/v1/import/{entity}` (multipart file upload). Steps: (1) parse workbook against the entity's template (§3), (2) validate every row with the **same Zod schema** used by that entity's create endpoint, (3) commit valid rows, (4) return the result envelope from [03-screenshot-and-excel-analysis.md](03-screenshot-and-excel-analysis.md) §7. Supports `?dryRun=true` (validate only, no commit) and `?mode=partial|strict` (partial = default, commit valid rows and report failures; strict = all-or-nothing).

Cross-sheet references (e.g. a `service_requests` import referencing a customer) resolve by a documented natural key (mobile number for customers, service `key` for services) rather than requiring the import file to know MongoDB ObjectIds, since legacy data never has them.

## 3. Per-Entity Templates (initial set)

| Entity | Template file | Key columns |
|---|---|---|
| Customers | `customers_import.xlsx` | name, mobile, alternateMobile, email, addressLine1, city, state, pinCode, gstin (business only) |
| Customer Products | `customer_products_import.xlsx` | customerMobile (link key), brand, productType, modelNumber, serialNumber, purchaseDate |
| Calls | `calls_import.xlsx` | callType, direction, callerNumber, customerMobile, callDate, callTime, symptoms, notes |
| Service Requests (historical/closed) | `service_requests_import.xlsx` | customerMobile, service, branch, status, createdAt, closedAt, defectFound, solution, partsUsed |
| Leads | `leads_import.xlsx` | source, customerName, mobile, requirement, stage, ownerEmail |
| Vendors | `vendors_import.xlsx` | companyName, contactMobile, servicesOffered, pinCodesCovered, gst, pan |

Each template's exact column headers match the corresponding create endpoint's field names (per [11-complete-api-contracts.md](11-complete-api-contracts.md)) so validation is identical between manual API calls and bulk import.

## 4. Downloadable Template Files

Every import screen offers a "Download template" action returning a blank `.xlsx` with the correct headers and one example row, generated from the same schema registry as validation (not a hand-maintained static file that can drift from the real schema).

## 5. Error Report Format

See [03-screenshot-and-excel-analysis.md](03-screenshot-and-excel-analysis.md) §7 — same shape reused for every entity's import.

## 6. Excel Export Column Configuration

Admins can define named, reusable "export presets" per entity (`GET/POST /config/export-presets/{entity}`) — a saved column selection + default filters — so recurring exports (e.g. "Monthly Branch Performance") don't require reselecting columns each time. This is separate from ad hoc column selection in §1.

## 7. Legacy Migration

Full mapping and dry-run procedure is documented in [03-screenshot-and-excel-analysis.md](03-screenshot-and-excel-analysis.md) §8; this document supplies the mechanism (import engine with `dryRun=true`, entity order `customers → customer_products → calls → service_requests → happy_calls`) it runs on.
