# 13 — Notification and Template System

## 1. Pipeline

```
Trigger event (domain module emits)
        ↓
Notification Engine resolves: template + recipient(s) + enabled channels
        ↓
One Notification job enqueued per {recipient, channel}
        ↓
Background Job Runner dispatches via channel adapter (in-app / push / email / WhatsApp / SMS-ready)
        ↓
Delivery/read/failure status recorded, retried per policy
```

A domain module (e.g. `service-requests`) never calls an email/WhatsApp SDK directly — it calls `notifications.trigger(triggerKey, context)` and the engine handles the rest, including whether that channel is even enabled. This is what makes "failure of WhatsApp/email must not crash the main business workflow" structurally true rather than something each module has to remember to guard.

## 2. Trigger Catalog

| Trigger key | Fires when | Default recipients | Default channels |
|---|---|---|---|
| `SERVICE_REQUEST_CREATED` | New SR created | Customer (ack), Branch dispatch queue (in-app) | IN_APP, PUSH, EMAIL, WHATSAPP |
| `SERVICE_REQUEST_ASSIGNED` | Assignment made | Assignee | IN_APP, PUSH |
| `ASSIGNMENT_ACCEPTED` / `ASSIGNMENT_REJECTED` | Assignee responds | Assigner | IN_APP, PUSH |
| `APPOINTMENT_SCHEDULED` / `APPOINTMENT_RESCHEDULED` | Scheduling event | Customer, assignee | PUSH, EMAIL, WHATSAPP |
| `TECHNICIAN_EN_ROUTE` | Travel started | Customer | PUSH, WHATSAPP |
| `TECHNICIAN_ARRIVED` | Check-in | Customer | PUSH |
| `ESTIMATE_SHARED` | Estimate sent | Customer | EMAIL, WHATSAPP |
| `ESTIMATE_APPROVED` / `ESTIMATE_REJECTED` | Customer responds | Assignee, Branch | IN_APP, PUSH |
| `PARTS_REQUIRED` | Diagnosis flags parts | Branch/Vendor procurement contact | IN_APP |
| `WORK_STARTED` / `SERVICE_COMPLETED` | Status change | Customer | PUSH |
| `INVOICE_GENERATED` | Invoice issued | Customer | EMAIL, WHATSAPP |
| `PAYMENT_PENDING` / `PAYMENT_RECEIVED` | Payment event | Customer, Finance | PUSH, EMAIL |
| `FOLLOW_UP_DUE` | Follow-up call scheduled | Assigned executive | IN_APP |
| `HAPPY_CALL_DUE` | Happy call scheduled | Happy Call Executive | IN_APP |
| `COMPLAINT_REOPENED` | Reopen created | Original assignee, Branch Manager | IN_APP, PUSH |
| `SLA_BREACHED` | SLA timer expires | Branch Manager, Admin | IN_APP, EMAIL |
| `VENDOR_DOCUMENT_EXPIRING` | Scheduled check, N days before expiry | Vendor Owner, Admin | EMAIL, IN_APP |
| `LEAD_FOLLOW_UP_DUE` | Lead follow-up date reached | Lead owner | IN_APP, PUSH |

Every row's recipients and enabled channels are **admin-editable** via `GET/PATCH /config/notification-rules` (masters engine) — the table above is seed data, not a hardcoded switch statement.

## 3. Template Model

`notification_templates` (per [09-database-architecture.md](09-database-architecture.md)): `triggerKey, channel, subjectTemplate? (email only), bodyTemplate, variables[]`. Templates use `{{variableName}}` interpolation, e.g. `"Hi {{customerName}}, your technician is on the way for {{serviceName}} (SR {{serviceRequestNumber}})."`. Variable catalog per trigger is documented alongside each row in §2 as the module is implemented.

## 4. Delivery, Retry, and Failure Handling

- Each `{recipient, channel}` pair is one `notifications` document with `status: PENDING → SENT → DELIVERED/READ` or `→ FAILED`.
- Retry policy: exponential backoff, default 3 attempts, configurable per channel (WhatsApp/email more retry-tolerant than push).
- A `FAILED` notification is logged with `failureReason` and surfaced in an admin "failed notifications" report — it never re-raises into the triggering business operation.
- Read receipts: `IN_APP` marked read on view; `PUSH` via platform delivery callback where available; `EMAIL`/`WHATSAPP` via provider webhook (open/read tracking, see [14-integration-architecture.md](14-integration-architecture.md)).

## 5. In-App Notification Center

`GET /api/v1/notifications` (paginated, per-user), `PATCH /api/v1/notifications/{id}/read`, unread count surfaced via a Socket.IO event (`notification.new`) so the bell icon updates in real time without polling.

## 6. Push Notifications

Firebase Cloud Messaging (default, per [00-project-overview.md](00-project-overview.md) open-items) for both Flutter apps. Device token registered on login (`POST /api/v1/users/{id}/devices`), deregistered on logout. Admin-web does not receive native push — its "real-time" equivalent is the Socket.IO in-app notification center plus a browser tab badge, not a separate push channel, for v1.

## 7. Template Governance

Templates are versioned (`revisionOf` pattern, same as financial documents) so a template edit doesn't retroactively change the wording of already-sent notifications visible in history/audit logs — the sent notification stores its fully-rendered content, not a live reference to the template.
