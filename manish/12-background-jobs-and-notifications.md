# Manish 12 — Background Jobs and Notifications

## 1. Job Queue Technology

BullMQ (Redis-backed) — chosen over a Mongo-backed queue for maturity of retry/backoff/scheduling features, given Redis is already in the stack for Socket.IO adapter scaling if needed. Queues: `notifications`, `pdf-generation`, `scheduled-tasks`, `campaigns`, `ai-requests`, `reports-aggregation`.

## 2. Scheduled Tasks (cron-style, via BullMQ repeatable jobs)

| Job | Schedule | Purpose |
|---|---|---|
| `escalationCheck` | every 15 min | SLA breach detection per §4 of [manish/06-workflow-engine-plan.md](06-workflow-engine-plan.md) |
| `happyCallScheduler` | daily | Creates `HappyCall` tasks for eligible completed/paid requests |
| `followUpReminder` | daily | Surfaces due follow-ups (leads, post-service) |
| `vendorDocumentExpiryCheck` | daily | Triggers `VENDOR_DOCUMENT_EXPIRING` notifications |
| `vendorPerformanceRecalc` | nightly | Recomputes vendor performance metrics per [manish/05-module-wise-backend-plan.md](05-module-wise-backend-plan.md) §Vendors |
| `reportAggregation` | nightly (+ near-real-time for critical dashboards, TBD per load) | Refreshes report summary collections per §Reports of [manish/05-module-wise-backend-plan.md](05-module-wise-backend-plan.md) |
| `numberingSeriesFYRollover` | on financial-year boundary (scheduled check daily near the boundary) | Resets sequence per series configured for FY-reset |

## 3. Notification Job Flow

`notifications.trigger()` enqueues one `notifications` queue job per `{recipient, channel}` (per [13-notification-and-template-system.md](../13-notification-and-template-system.md) §1). Job processor: resolve template, render variables, call the channel adapter (from [14-integration-architecture.md](../14-integration-architecture.md)), update the `notifications` document status. Retry policy per channel configured via BullMQ's backoff options, matching §4 of [13-notification-and-template-system.md](../13-notification-and-template-system.md).

## 4. PDF Generation Jobs

Enqueued on-demand (estimate/invoice share action) rather than scheduled; kept as a queue (not synchronous in the request) because PDF rendering (Puppeteer-based) is slow enough to matter for API response time.

## 5. Job Observability

Bull Board (or equivalent) mounted on an internal-only admin route for developers to inspect queue health/failed jobs during development and staging; failed jobs beyond their retry limit surface in the "failed notifications" report per §4 of [13-notification-and-template-system.md](../13-notification-and-template-system.md) for business visibility, separate from the developer-facing queue dashboard.

## 6. Idempotency in Jobs

Every job payload includes the originating record's ID and an idempotency key where the action could plausibly be double-enqueued (e.g. a retried API request re-triggering the same notification) — job processors check for an existing completed result before re-executing, consistent with §11 of [10-api-standards.md](../10-api-standards.md).
