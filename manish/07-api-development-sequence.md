# Manish 07 — API Development Sequence

Backend build order within and across phases, expanding [22-phase-wise-development-plan.md](../22-phase-wise-development-plan.md) into an endpoint-level sequence. Each numbered group is contract-frozen + mocked (per [coordination/04-api-handover-process.md](../coordination/04-api-handover-process.md)) before Rohit needs it, ideally 1 group ahead of Rohit's active work.

1. Auth (`/auth/*`) — unblocks every other authenticated screen.
2. Organization (`/branches`, `/sub-branches`, `/teams`) + Masters (`/masters/{type}`) + Config (`/config/*`) — unblocks every dropdown/reference-data need across the whole app.
3. Users/Employees/Vendors — unblocks assignment UI later, but also needed early for admin user-management screens.
4. Customers + Customer Products — unblocks booking flow.
5. Catalog (Services/Brands/Product Types) — unblocks booking flow and service management screens.
6. Calls — unblocks call-center screens.
7. Leads — unblocks lead pipeline screens.
8. Service Requests core (create, list, detail, status, assign) — the central module, largest single group.
9. Service Visits (field execution) — unblocks mobile app screens.
10. Estimates → Proforma → Invoices → Payments (finance chain, built in conversion order since each depends on the previous existing).
11. Happy Calls + Reopen.
12. Notifications (in-app list/read) + Notification Templates admin.
13. Marketing (Campaigns, Consent).
14. AI settings + endpoints.
15. Files (signed upload/confirm).
16. Reports (built last per module since they aggregate across everything above).
17. Import/Export (built per-entity as that entity's core CRUD stabilizes, not all at once at the end).

Real-time (Socket.IO) events are added incrementally alongside group 8 onward (wherever a status change first needs to be live) rather than as a separate late phase, since retrofitting real-time into an already-built polling UI is more work than building it in from the start per [08-system-architecture.md](../08-system-architecture.md) §4.

## Parallel-Safe Grouping

Groups 2-5 have no interdependency on 6-11 and could be built in any order relative to each other if useful for pacing against Rohit's UI sequence ([rohit/13-ui-development-sequence.md](../rohit/13-ui-development-sequence.md)); groups 6 onward have hard sequential dependencies as listed (a Call/Lead can reference a Customer/Service, a Service Request references Calls/Leads, etc.).
