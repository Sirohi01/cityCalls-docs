# 10 — API Standards

Binding for every endpoint Manish builds and every call Rohit's adapters make. No endpoint ships that deviates from this document without updating it first (see [coordination/08-change-request-process.md](coordination/08-change-request-process.md)).

## 1. Base URL & Versioning

`https://{host}/api/v1/...` — version is in the URL path. A breaking change to any existing endpoint requires a `v2` path for that resource, not an in-place change; additive, backward-compatible changes (new optional field) may ship within `v1`.

## 2. Authentication

`Authorization: Bearer {jwt}` header on every authenticated request. Access token short-lived (default 15 min), refresh token longer-lived (default 7 days), refresh via `POST /api/v1/auth/refresh`. Customer app additionally supports OTP-based login (`POST /api/v1/auth/otp/request`, `POST /api/v1/auth/otp/verify`).

## 3. Standard Success Envelope

```json
{
  "success": true,
  "message": "Service request fetched successfully",
  "data": {},
  "meta": {
    "page": 1,
    "limit": 20,
    "total": 100,
    "totalPages": 5
  },
  "errors": null
}
```

`meta` is present only on list/paginated endpoints; `null` on single-record endpoints. `message` is always a human-readable, present-tense confirmation, never omitted.

## 4. Standard Error Envelope

```json
{
  "success": false,
  "message": "Validation failed",
  "data": null,
  "errors": [
    {
      "field": "customer.mobile",
      "code": "INVALID_MOBILE",
      "message": "Enter a valid mobile number"
    }
  ]
}
```

Full HTTP status code / error code catalog in [18-error-handling-standards.md](18-error-handling-standards.md). `errors` is always an array (even for a single error) or `null`; never a bare object.

## 5. Pagination

Query params: `page` (1-indexed, default 1), `limit` (default 20, max 100). Response `meta.total`/`meta.totalPages` always computed server-side (not trusted from client). Cursor-based pagination is not used in v1 — offset-based is sufficient at expected data volumes and simpler for Rohit's table components to implement (page-number UI).

## 6. Sorting

Query param `sort`, format `field:asc|desc`, e.g. `sort=createdAt:desc`. Multiple sort keys: comma-separated, `sort=priority:desc,createdAt:asc`. Unsupported sort fields return a `400` with `INVALID_SORT_FIELD`, not a silent ignore.

## 7. Filtering

Query params match field names directly for simple equality (`?status=NEW&branchId=...`), with these operators for ranges/sets via suffix: `field_gte`, `field_lte`, `field_in` (comma-separated). Free-text search uses a dedicated `q` param, scoped per-endpoint to the fields documented in that endpoint's contract (e.g. customer name/mobile/email). Every list endpoint documents its exact filterable field set in [11-complete-api-contracts.md](11-complete-api-contracts.md) — Rohit must not assume a filter exists that isn't documented there.

## 8. Field Typing & Nullability Rules

- Dates/times are ISO 8601 strings in UTC (`2026-07-15T09:30:00.000Z`); display-timezone conversion happens client-side.
- IDs are MongoDB ObjectId strings (24-char hex).
- A field that can be absent-or-null is documented as `nullable: true` in the contract; the API always includes the key with `null` rather than omitting it, so clients can rely on key presence.
- Enum fields only ever contain values from the registry in [coordination/06-naming-conventions.md](coordination/06-naming-conventions.md) §4 — any new value requires updating that doc first.
- Money amounts are numbers in INR with up to 2 decimal places, never formatted strings (e.g. `1250.50`, not `"₹1,250.50"`) — formatting is a client concern.

## 9. File Upload Rules

Uploads use `multipart/form-data` for direct-to-API upload endpoints, or a **signed-upload flow** for Cloudinary-bound files (API issues a short-lived signed upload URL/params, client uploads directly to Cloudinary, then confirms the asset back to the API) — preferred for images/video to avoid proxying large files through the API server. See [14-integration-architecture.md](14-integration-architecture.md) for the signed-upload contract and fallback-storage behavior when Cloudinary is disabled. File-type and size limits are enforced server-side regardless of which flow is used, per file category (defined in [09-database-architecture.md](09-database-architecture.md) `services.requiredImages` and equivalent per-entity config).

## 10. Validation Rules

Every request body is validated server-side with a Zod schema (mirrored locally within `citycalls-admin-web` for its own client-side pre-validation, and mirrored in the OpenAPI spec for each mobile repo's independent codegen — no shared validation-schemas package exists) — client-side validation is a UX convenience, never a substitute for server enforcement. Validation failures return `422` with the error envelope in §4, one entry per invalid field.

## 11. Idempotency

State-changing endpoints that a flaky mobile connection might retry (status changes, payment recording, file-upload confirmation) accept an optional `Idempotency-Key` header; the API deduplicates by this key within a rolling window (default 24h) and returns the original response rather than performing the action twice. Mandatory for the offline-sync endpoints used by the vendor app (see [08-system-architecture.md](08-system-architecture.md) §5).

## 12. Rate Limiting

Public/customer-facing and OTP endpoints are rate-limited per IP and per identifier (mobile number) — exact limits in [17-security-and-audit.md](17-security-and-audit.md). Rate-limited responses return `429` with a `Retry-After` header.

## 13. Endpoint Documentation Template

Every endpoint in [11-complete-api-contracts.md](11-complete-api-contracts.md) is documented with: Endpoint, Method, Purpose, Auth required (Y/N), Required permission (`module.action.scope`), Path params, Query params, Request body (with example), Success response (with example), Error responses (with codes), Pagination/sorting/filtering support, Validation rules, File-upload rules if applicable.
