# 18 — Error Handling Standards

## 1. HTTP Status Code Usage

| Code | Meaning | When |
|---|---|---|
| `200` | Success | Successful GET/PATCH/action endpoint |
| `201` | Created | Successful POST creating a resource |
| `202` | Accepted | Async job started (large export, background sync) |
| `400` | Bad request | Malformed request (bad query param, invalid sort field) — distinct from validation errors on body |
| `401` | Unauthorized | Missing/invalid/expired token |
| `403` | Forbidden | Authenticated but lacks permission/scope for this action or record |
| `404` | Not found | Record doesn't exist or isn't visible to caller's data scope (see §5 — deliberately indistinguishable from "doesn't exist") |
| `409` | Conflict | Invalid state transition, duplicate unique field, idempotency-key replay with different payload |
| `422` | Unprocessable | Request body fails validation |
| `429` | Too many requests | Rate limit exceeded |
| `500` | Server error | Unhandled exception — always logged, never exposes internal detail to the client |

## 2. Error Code Registry (representative — full list grows per module, registered here before use)

| Code | HTTP | Meaning |
|---|---|---|
| `INVALID_CREDENTIALS` | 401 | Login failed |
| `TOKEN_EXPIRED` | 401 | Access token expired, client should refresh |
| `INVALID_MOBILE` | 422 | Mobile number format invalid |
| `REQUIRED_FIELD_MISSING` | 422 | A required field was absent |
| `INVALID_TRANSITION` | 409 | Status change not allowed from current state (see [07-status-transition-matrix.md](07-status-transition-matrix.md)) |
| `ASSIGNMENT_OUT_OF_SCOPE` | 403 | Caller's data scope doesn't cover the target |
| `DUPLICATE_RECORD` | 409 | Unique constraint violated (e.g. duplicate mobile on create) |
| `PIN_CODE_NOT_SERVICEABLE` | 200 (not an error — see §6) | No branch/vendor coverage for the address |
| `REOPEN_WINDOW_EXPIRED` | 200 (not an error — see §6) | Outside reopen policy window, request still created per [06-complete-workflow-document.md](06-complete-workflow-document.md) Stage 11 |
| `INVALID_SORT_FIELD` | 400 | Unsupported `sort` query value |
| `RATE_LIMITED` | 429 | Too many requests |
| `INTEGRATION_DISABLED` | 200 (informational, not error) | Optional integration off — action still succeeds per [14-integration-architecture.md](14-integration-architecture.md) |
| `FILE_TYPE_NOT_ALLOWED` | 422 | Upload rejected by type allowlist |
| `FILE_TOO_LARGE` | 422 | Upload exceeds category size limit |

New error codes are added to this registry in the same PR that introduces them — never invented ad hoc in a controller without a corresponding row here, since Rohit's error-display logic (§6 of [12-frontend-data-contracts.md](12-frontend-data-contracts.md)) matches on these exact codes for anything needing custom handling beyond generic field-error display.

## 3. Validation Error Detail

Every `422` includes one entry per invalid field in the standard `errors[]` array ([10-api-standards.md](10-api-standards.md) §4) — never a single combined message string for multiple field errors, so the form layer can highlight each field independently.

## 4. Unhandled Exceptions

A global error-handling middleware catches anything not explicitly thrown as a known error type, logs the full stack trace server-side (with request context: user, route, body — minus sensitive fields like passwords), and returns a generic `500` response (`"message": "Something went wrong. Please try again."`) — internal error detail (stack traces, DB error messages) is never returned to the client, to avoid leaking implementation details.

## 5. 404 vs. 403 Disclosure Policy

When a record exists but is outside the caller's data scope, the API returns `404`, not `403` — this avoids confirming a record's existence to a user who shouldn't know about it (e.g. a Branch Manager probing IDs shouldn't learn that a Service Request exists in another branch even via a 403). Genuine "you're logged in as the wrong role for this whole module" cases (not record-specific) do return `403`.

## 6. Business Outcomes That Are Not Errors

Several conditions described elsewhere as "failure cases" in the workflow document are **not** HTTP errors — they're normal business outcomes represented in a `200` success response with an informational flag (see `PIN_CODE_NOT_SERVICEABLE`, `REOPEN_WINDOW_EXPIRED`, `INTEGRATION_DISABLED` above). This distinction matters because Rohit's generic error-toast handling (triggered by `success: false`) must not fire for these — they need dedicated UI treatment (e.g. an inline banner), which is why they're called out explicitly here rather than left for each screen to guess.

## 7. Mobile Offline-Sync Error Handling

Sync-queue errors (per [08-system-architecture.md](08-system-architecture.md) §5) use the same error codes but are surfaced differently: a `409 INVALID_TRANSITION` on a queued action doesn't show a toast (the user isn't actively watching) — it flags that specific queued item in a "sync issues" list within the app for the technician to review and resolve manually, since the underlying cause (someone else already changed the status) needs a human decision, not a silent retry.

## 8. Logging Correlation

Every request is tagged with a `requestId` (returned in a response header, `X-Request-Id`), included in server-side logs, so a user-reported error can be traced to its exact server log entry without exposing internal detail in the client-facing error itself.
