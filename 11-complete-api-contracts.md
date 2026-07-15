# 11 — Complete API Contracts

This document is the contract surface between Manish and Rohit. All shapes follow [10-api-standards.md](10-api-standards.md). Every endpoint listed in §2 (the full catalog) gets fully worked request/response JSON **before Rohit starts the screen that consumes it** — Manish fills in the JSON using the template in [10-api-standards.md](10-api-standards.md) §13 as each module is built, per the sequence in [manish/07-api-development-sequence.md](manish/07-api-development-sequence.md). §1 below front-loads full worked examples for the highest-risk-of-misalignment flows so both developers can start immediately without waiting for the full catalog to be fleshed out.

## 1. Fully Worked Examples (core critical path)

### 1.1 `POST /api/v1/auth/login`
Auth: none. Body: `{ "identifier": "9876543210", "password": "..." }` (identifier = email or mobile).
Success `200`:
```json
{
  "success": true,
  "message": "Login successful",
  "data": {
    "accessToken": "eyJ...",
    "refreshToken": "eyJ...",
    "user": { "id": "665f...", "name": "Ramesh Kumar", "role": "BRANCH_MANAGER", "branchId": "665a..." }
  },
  "meta": null,
  "errors": null
}
```
Error `401`: `errors: [{ "field": "identifier", "code": "INVALID_CREDENTIALS", "message": "Incorrect email/mobile or password" }]`

### 1.2 `POST /api/v1/service-requests`
Auth required. Permission: `serviceRequests.create.own` (customer) or `.branch` (staff).
Body:
```json
{
  "customerId": "665f1...",
  "customerProductId": "665f2...",
  "addressId": "665f1...addr0",
  "serviceId": "665f3...",
  "symptoms": ["665f4...sym1", "665f4...sym2"],
  "priority": "NORMAL",
  "source": "CUSTOMER_APP",
  "preferredSlot": { "date": "2026-07-18", "slotId": "665f5...slot2" },
  "notes": "AC not cooling since yesterday",
  "images": ["665f6...file1"]
}
```
Success `201`:
```json
{
  "success": true,
  "message": "Service request created successfully",
  "data": {
    "id": "665f7...",
    "number": "SR-DEL01-2526-000482",
    "status": "NEW",
    "customerId": "665f1...",
    "serviceId": "665f3...",
    "branchId": "665a...",
    "priority": "NORMAL",
    "createdAt": "2026-07-15T09:30:00.000Z"
  },
  "meta": null,
  "errors": null
}
```
Error `422` (uncovered pin code is NOT an error — see §1.3): standard validation envelope for missing/invalid fields.

### 1.3 `GET /api/v1/service-requests` (list)
Auth required. Permission-scoped automatically (server filters by caller's data scope).
Query: `?status=ASSIGNED_TO_BRANCH&branchId=665a...&page=1&limit=20&sort=createdAt:desc&q=SR-DEL01`
Success `200`: `data` = array of list-shape objects (`id, number, status, customerName, serviceName, branchName, priority, scheduledDate, assigneeName, createdAt` — deliberately denormalized/flattened for table rendering, not the full nested document), `meta` populated per [10-api-standards.md](10-api-standards.md) §5.

### 1.4 `PATCH /api/v1/service-requests/{id}/status`
Auth required. Permission checked against the role-gating table in [07-status-transition-matrix.md](07-status-transition-matrix.md) for the specific `from → to` pair.
Body: `{ "toStatus": "TECHNICIAN_ARRIVED", "geo": { "lat": 28.61, "lng": 77.23 }, "idempotencyKey": "..." }`
Success `200`: updated `{ id, status, updatedAt }`. Error `409 INVALID_TRANSITION` if `toStatus` isn't reachable from the record's current status for the caller's role — body includes `currentStatus` and `allowedTransitions[]` so the client can recover gracefully instead of guessing.

### 1.5 `POST /api/v1/service-requests/{id}/assign`
Permission: `serviceRequests.assign.{scope}`.
Body: `{ "assigneeType": "EMPLOYEE", "assigneeId": "665f8...", "method": "MANUAL", "reason": null }`
Success `200`: updated SR + new `assignment_history` entry id. Error `403 ASSIGNMENT_OUT_OF_SCOPE` if caller's data scope doesn't cover the target branch/assignee — this is the concrete enforcement of the bypass rules in [05-user-roles-and-permissions.md](05-user-roles-and-permissions.md) §6.

### 1.6 `POST /api/v1/calls`
Body varies by `callType`; shared envelope + `details` sub-object per [09-database-architecture.md](09-database-architecture.md) `calls.details`. Example for `callType: "INITIAL"`:
```json
{
  "callType": "INITIAL",
  "direction": "INCOMING",
  "callerNumber": "9876543210",
  "customerId": null,
  "customerName": "Suresh Mehta",
  "brandId": "665f9...",
  "productTypeId": "665fa...",
  "symptoms": ["665f4...sym1"],
  "initialComplaint": "Fridge not cooling",
  "priority": "NORMAL",
  "createdBy": "665fb..."
}
```
Success `201`: `{ id, number: "CL-DEL01-2526-001204", callType, createdAt }`.

### 1.7 `POST /api/v1/leads/{id}/convert`
Body: `{ "convertTo": "SERVICE_REQUEST", "serviceId": "665f3...", ... same shape as 1.2 minus customer fields already on the lead }`
Success `200`: `{ leadId, stage: "CONVERTED", serviceRequestId: "665f7..." }`.

### 1.8 `POST /api/v1/estimates`
Body: `{ "serviceRequestId": "665f7...", "items": [{ "description": "Compressor replacement", "partId": "665fc...", "qty": 1, "unitPrice": 4500, "taxRateId": "665fd..." }], "labourCharge": 500 }`
Success `201`: `{ id, number: "EST-DEL01-2526-000091", status: "DRAFT", total: 5900, pdfUrl: null }`. A follow-up `POST /api/v1/estimates/{id}/share` (channels: `["EMAIL","WHATSAPP"]`) triggers PDF generation + send and moves status to `SHARED`.

### 1.9 `PATCH /api/v1/estimates/{id}/approve` (customer)
Body: `{}`. Success `200`: `{ id, status: "APPROVED", approvedAt }`. Triggers the linked Service Request's status transition per [07-status-transition-matrix.md](07-status-transition-matrix.md).

### 1.10 `POST /api/v1/notifications` — internal only, not client-callable; documented here because Rohit needs `GET /api/v1/notifications` (list, for the in-app notification center) and `PATCH /api/v1/notifications/{id}/read`, both standard list/update shapes per §10 conventions.

## 2. Full Endpoint Catalog (method, path, purpose, permission — JSON detail filled per §0 process)

| Module | Endpoint | Purpose | Permission |
|---|---|---|---|
| Auth | `POST /auth/login`, `POST /auth/refresh`, `POST /auth/logout`, `POST /auth/otp/request`, `POST /auth/otp/verify`, `POST /auth/password/reset-request`, `POST /auth/password/reset` | Session lifecycle | public / self |
| Users | `GET/POST /users`, `GET/PATCH/DELETE /users/{id}` | Staff/vendor user management | `users.*.{scope}` |
| Organization | `GET/POST /branches`, `GET/PATCH /branches/{id}`, same for `/sub-branches`, `/teams` | Org hierarchy CRUD | `organization.*.{scope}` |
| Employees | `GET/POST /employees`, `GET/PATCH /employees/{id}`, `GET /employees/{id}/availability` | Employee management | `employees.*.{scope}` |
| Vendors | `GET/POST /vendors`, `GET/PATCH /vendors/{id}`, `GET/POST /vendors/{id}/technicians`, `GET /vendors/{id}/performance`, `GET/POST /vendors/{id}/documents` | Vendor lifecycle | `vendors.*.{scope}` |
| Customers | `GET/POST /customers`, `GET/PATCH /customers/{id}`, `POST /customers/{id}/addresses`, `POST /customers/{id}/products`, `GET /customers/{id}/history`, `GET /customers/duplicates` | Customer management | `customers.*.{scope}` |
| Catalog | `GET/POST /services`, `GET/PATCH /services/{id}`, `GET/POST /brands`, `/product-types`, `/models` | Dynamic service catalog | `catalog.*.{scope}` |
| Masters | `GET/POST /masters/{masterType}`, `GET/PATCH/DELETE /masters/{masterType}/{id}` | Generic masters CRUD (§3 of [09](09-database-architecture.md)) | `config.manageSettings.all` |
| Config | `GET/PATCH /config/policies`, `GET/PATCH /config/numbering-series`, `GET/PATCH /config/role-permissions` | Dynamic configuration | `config.manageSettings.all` |
| Calls | `GET/POST /calls`, `GET/PATCH /calls/{id}`, `GET /calls/{id}/timeline` | Call management (§1.6) | `calls.*.{scope}` |
| Leads | `GET/POST /leads`, `GET/PATCH /leads/{id}`, `POST /leads/{id}/convert`, `POST /leads/bulk-assign`, `POST /leads/merge` | Lead management | `leads.*.{scope}` |
| Service Requests | `GET/POST /service-requests`, `GET/PATCH /service-requests/{id}`, `PATCH /{id}/status`, `POST /{id}/assign`, `POST /{id}/reassign`, `POST /{id}/cancel`, `POST /{id}/reopen`, `GET /{id}/assignment-history` | Core workflow (§1.2-1.5) | `serviceRequests.*.{scope}` |
| Service Visits | `POST /service-requests/{id}/visits`, `PATCH /visits/{id}`, `POST /visits/{id}/images` | Field execution capture | `fieldExecution.*.own` |
| Happy Calls | `GET/POST /happy-calls`, `PATCH /happy-calls/{id}` | Post-service follow-up | `calls.*.{scope}` |
| Reopen | `GET /service-requests/{id}/reopen-eligibility`, `POST /service-requests/{id}/reopen` | Reopen flow | `serviceRequests.*.{scope}` |
| Finance | `GET/POST /estimates`, `/proforma-invoices`, `/invoices`, `/payment-receipts`, `/credit-notes`, `/debit-notes`, conversion endpoints (`POST /estimates/{id}/convert`), `POST /invoices/{id}/payments` | Financial document chain (§1.8-1.9) | `finance.*.{scope}` |
| Vendor Finance | `GET/POST /vendor-invoices`, `/vendor-payouts` | Vendor settlement | `vendors.viewFinancial.{scope}` |
| Notifications | `GET /notifications`, `PATCH /notifications/{id}/read`, `GET/PATCH /notification-templates` | In-app + template mgmt | self / `config.manageSettings.all` |
| Marketing | `GET/POST /campaigns`, `GET /campaigns/{id}/stats`, `GET/PATCH /consent` | WhatsApp/email campaigns | `marketing.*.all` |
| AI | `POST /ai/summarize-call`, `POST /ai/classify-complaint`, `GET/PATCH /ai/settings` | Optional AI features | `ai.*.all` (feature-gated) |
| Files | `POST /files/signed-upload`, `POST /files/confirm`, `DELETE /files/{id}` | Cloudinary/fallback uploads | self/module-scoped |
| Reports | `GET /reports/{reportKey}` (generic, filters via query) | Reporting | `reports.view.{scope}` |
| Export | `GET /export/{entity}` (returns file), `POST /import/{entity}` (returns row-wise result) | Excel import/export | `*.export/import.{scope}` |
| Audit | `GET /audit-logs`, `GET /{entityType}/{entityId}/timeline` | Audit/timeline | `audit.viewAuditLog.{scope}` |

Each row above expands into the full template ([10-api-standards.md](10-api-standards.md) §13) in this file as Manish implements that module, keeping this document the living single source of truth rather than letting contracts drift into code comments.

## 3. Mock Data Policy

Every endpoint in §1 has a corresponding static mock JSON file under each consuming frontend repo's own local `mocks/` directory (no shared mocks package — see [coordination/03-code-ownership.md](coordination/03-code-ownership.md) §4) matching the exact success-response shape shown above, so Rohit can build against mocks immediately in whichever repo he's working in. See [coordination/05-mock-data-contract.md](coordination/05-mock-data-contract.md) and [rohit/14-mock-data-and-api-adapter-plan.md](rohit/14-mock-data-and-api-adapter-plan.md) for the switch-over mechanism from mock to live API.
