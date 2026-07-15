# Manish 08 — Customer App Functional Plan

Functional/data-layer scope for the `citycalls-customer-mobile` repo (Flutter) — screens themselves are Rohit's, per [rohit/05-customer-app-screen-list.md](../rohit/05-customer-app-screen-list.md). This repo shares no code with `citycalls-vendor-mobile`.

## 1. Data Layer

`lib/data/` repositories per module consumed by the customer app: `AuthRepository` (OTP login), `CustomerRepository` (profile/addresses), `ServiceRepository` (browse catalog), `ServiceRequestRepository` (booking/tracking/history), `EstimateInvoiceRepository`, `NotificationRepository`. Each wraps the Dio-based API client per [12-frontend-data-contracts.md](../12-frontend-data-contracts.md) §3.

## 2. Booking Flow (functional wiring for Stage 1 of [06-complete-workflow-document.md](../06-complete-workflow-document.md))

Service selection → coverage check (`GET /services/{id}/coverage?pinCode=`) before allowing progression → product selection/creation → address selection/creation → optional image upload (signed Cloudinary flow per [14-integration-architecture.md](../14-integration-architecture.md)) → slot selection (`GET /appointment-slots?branchId=&date=`) → submit (`POST /service-requests`).

## 3. Real-Time Tracking

Customer app subscribes to `service-request:{id}` Socket.IO room while a request is active (per [08-system-architecture.md](../08-system-architecture.md) §4), showing live status + technician location (once `TECHNICIAN_EN_ROUTE`) without polling.

## 4. Push Notifications

FCM token registered on login, deregistered on logout, per [13-notification-and-template-system.md](../13-notification-and-template-system.md) §6.

## 5. Offline Behavior

Customer app is **not** offline-first (unlike the vendor app) — network-dependent screens show a clear offline state (per [rohit/11-loading-error-empty-states.md](../rohit/11-loading-error-empty-states.md)) rather than queuing actions, since booking/payment actions genuinely require connectivity and shouldn't silently queue.

## 6. Payments

Payment recording UI calls `POST /invoices/{id}/payments` for manual methods (UPI reference, etc.); gateway-based in-app payment is stubbed behind the `PaymentGateway` interface (§6 of [14-integration-architecture.md](../14-integration-architecture.md)) until a provider is selected — the screen exists and is functional for manual methods from day one.

## 7. Consent Capture

Registration/profile screens capture WhatsApp/email marketing consent explicitly (not pre-checked) per [17-security-and-audit.md](../17-security-and-audit.md) §8, writing to `customers.consent` with an audit trail.
