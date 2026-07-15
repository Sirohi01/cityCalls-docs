# Rohit 05 — Customer App Screen List

Full screen inventory for the `citycalls-customer-mobile` repo (Flutter, fully independent of `citycalls-vendor-mobile`), functional wiring per [manish/08-customer-app-functional-plan.md](../manish/08-customer-app-functional-plan.md).

| Flow | Screens |
|---|---|
| Onboarding | Splash, Login (mobile + OTP), OTP Verify, Registration/Profile Setup |
| Home | Home/Dashboard (quick-book, active requests, promotions), Service Category Browse, Service Detail |
| Booking | Product Select/Add, Address Select/Add, Issue Description + Image Upload, Slot Selection, Booking Review & Confirm, Booking Success |
| Tracking | Service Request List (active + history tabs), Service Request Detail (status timeline, live map once en route), Technician Info (permission-gated) |
| Reschedule/Cancel | Reschedule screen, Cancel confirmation (policy-aware messaging) |
| Estimates & Payments | Estimate Review & Approve/Reject, Invoice View, Payment screen (manual methods; gateway stubbed), Payment History |
| History | Service History list, Service Detail (past), Reopen Request screen |
| Feedback | Rating & Feedback screen (post-completion), Happy Call response (if applicable via app) |
| Support | Contact Support / Raise Complaint, FAQ/Help |
| Profile | Profile edit, Address book, Saved Products, Notification Preferences / Consent management, Notification Center |

## Screen-to-Workflow-Stage Mapping

Booking screens implement Stage 1 of [06-complete-workflow-document.md](../06-complete-workflow-document.md); Tracking screens implement Stages 2-8 (read-only + limited actions like reschedule/cancel/approve-estimate); Estimates & Payments implements Stages 6 and 9; History/Feedback implements Stages 10-11.

## Tab Structure (bottom navigation, typical)

Home, My Services (active + history), Notifications, Profile — kept to 4 tabs for scan-ability; exact structure finalized during visual design, this document fixes the screen inventory, not the navigation chrome.
