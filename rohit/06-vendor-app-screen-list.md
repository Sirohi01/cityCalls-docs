# Rohit 06 — Vendor/Employee App Screen List

Full screen inventory for the `citycalls-vendor-mobile` repo (Flutter, fully independent of `citycalls-customer-mobile` — no shared code between the two mobile apps), functional wiring per [manish/09-vendor-app-functional-plan.md](../manish/09-vendor-app-functional-plan.md). Same binary serves both Employee and Vendor Technician roles (§6 of that doc).

| Flow | Screens |
|---|---|
| Onboarding | Splash, Login, (Vendor-only) Company/Technician selector if a Vendor Owner logs in managing multiple technicians |
| Job Management | My Jobs (today's route, ordered by scheduled slot), Job List (all assigned, filterable by status/date), Job Detail |
| Execution | Accept/Reject screen, Start Travel action, Arrival Check-in (with geolocation capture), Inspection & Diagnosis form, Estimate creation/review (if applicable), Customer Approval capture screen, Work Progress log, Parts Entry, Before/After Image Capture, Completion screen (remarks + OTP/signature capture), Payment Collection screen |
| Sync | Sync Status screen, Sync Issues / Conflict Resolution screen (per §3 of [manish/09](../manish/09-vendor-app-functional-plan.md)) |
| Escalation/Reassignment | Raise Escalation form, Request Reassignment form |
| History | Completed Jobs History, Job Detail (past, read-only) |
| Vendor-Owner-Only | Technician roster management, Vendor performance dashboard, Vendor invoice/payout view |
| Profile | Profile view, Availability toggle, Notification Center |

## Offline-First UI Requirements (cross-cutting, applies to every Execution screen)

Every action button in the Execution flow works fully offline — tapping "Start Travel" with no connectivity succeeds locally immediately (per §1-2 of [manish/09-vendor-app-functional-plan.md](../manish/09-vendor-app-functional-plan.md)) and shows a `SyncStatusIndicator` (pending/synced) rather than a spinner blocking the UI. This is the single most important non-negotiable UI requirement for this app — see [12-responsive-design-guidelines.md](12-responsive-design-guidelines.md) for how offline states are visually distinguished from loading/error states.

## Screen-to-Workflow-Stage Mapping

Job Management + Execution screens implement Stages 3, 5-8 of [06-complete-workflow-document.md](../06-complete-workflow-document.md); Sync screens are the UI surface for [08-system-architecture.md](../08-system-architecture.md) §5's conflict-resolution requirement.
