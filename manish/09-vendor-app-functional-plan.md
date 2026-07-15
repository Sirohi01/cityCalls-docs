# Manish 09 — Vendor/Employee App Functional Plan

Functional/data-layer scope for the `citycalls-vendor-mobile` repo (Flutter), the most architecturally involved client given offline-first requirements (§5 of [08-system-architecture.md](../08-system-architecture.md)). This repo shares no code with `citycalls-customer-mobile`, even though both are Flutter.

## 1. Local Persistence

SQLite (via `drift`) mirroring: assigned Service Requests (subset of fields needed for execution), their Service Visits, a local action queue (`pending_actions` table: `actionType, payload, clientTimestamp, idempotencyKey, syncStatus`).

## 2. Sync Engine

`lib/sync/syncEngine.dart` — on connectivity restore (or periodic timer while online), pushes queued `pending_actions` to `POST /service-requests/{id}/sync-batch` in client-recorded order, per action; server processes each independently (per §"Field Execution" of [manish/05-module-wise-backend-plan.md](05-module-wise-backend-plan.md)), returns per-action success/conflict; engine marks each local action `SYNCED` or `CONFLICT` accordingly, never silently drops a conflict.

## 3. Conflict Surfacing

A `CONFLICT` action appears in a "Sync Issues" screen (Rohit builds the UI per [rohit/06-vendor-app-screen-list.md](../rohit/06-vendor-app-screen-list.md)) showing the server's actual current state vs. the technician's queued attempt, with options to discard or retry against current state — never auto-resolved.

## 4. Image/Video Handling

Captured images queue locally (file path stored, not yet uploaded) with the same `pending_actions` mechanism; upload happens opportunistically via the signed-upload flow, and the associated Service Visit record links the image once its upload confirms — the workflow action (e.g. "mark inspection complete") does not block on the image finishing upload.

## 5. Location Pings

While status is `TECHNICIAN_EN_ROUTE`, a background location service sends periodic pings (§4 of [08-system-architecture.md](../08-system-architecture.md), default 60s) via a lightweight endpoint — pings queue locally too if briefly offline, but old stale pings are dropped rather than sent late (only the most recent unsent ping matters for live tracking).

## 6. Role-Based Screen Access Within the App

The same app binary serves Employee and Vendor Technician roles; `req.user.role` and `vendorId`/`branchId` scoping (per [05-user-roles-and-permissions.md](../05-user-roles-and-permissions.md)) determine which jobs appear in the "My Jobs" list — no separate app build per role.

## 7. Daily Job List

`GET /service-requests?assigneeId={self}&status_in=ACCEPTED,APPOINTMENT_SCHEDULED,...&scheduledDate={today}` ordered by `scheduledSlot` — the functional basis for the "daily route" screen flagged as a missing-feature consideration in project scoping.

## 8. Completion Proof Capture

OTP flow: `POST /service-requests/{id}/completion-otp/request` (sends OTP to customer) → `POST /service-requests/{id}/completion-otp/verify`. Signature: captured as an image upload via the same file flow. Both queue/sync exactly like other actions in §1-3.
