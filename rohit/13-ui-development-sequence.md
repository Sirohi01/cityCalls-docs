# Rohit 13 — UI Development Sequence

Screen build order, tracking Manish's API sequence ([manish/07-api-development-sequence.md](../manish/07-api-development-sequence.md)) with contract-freeze-and-mock happening ahead of each group per [coordination/04-api-handover-process.md](../coordination/04-api-handover-process.md).

1. **Design system + shared components** ([02-design-system.md](02-design-system.md), [03-shared-component-inventory.md](03-shared-component-inventory.md)) — build the primitives before any screen, since every screen depends on them.
2. **Auth screens** (Login, OTP, Password Reset) — admin-web and both mobile apps, unblocks everything behind a login.
3. **Admin shell** (nav, dashboard layout skeleton, permission-gated menu) + **Masters/Config generic screens**.
4. **Organization management** (Branch/Sub-Branch/Team).
5. **Customer + Product management** (admin) + **Customer app onboarding/profile** screens.
6. **Service catalog management** (admin) + **Customer app service browse/booking flow**.
7. **Call management screens** (admin).
8. **Lead management screens** (admin).
9. **Service Request core** (admin list/detail/dispatch) + **Customer app tracking screens** — largest single group, built incrementally per Service Request sub-stage as Manish's status-engine endpoints land.
10. **Vendor app job list + execution flow** — built against mocks well ahead of the offline-sync engine being live, since the UI doesn't need to know sync internals, only the `SyncStatusIndicator` state.
11. **Vendor/Employee management** (admin).
12. **Financial screens** (Estimate/Invoice/Payment, admin + customer app estimate-approval/payment screens).
13. **Happy Call + Reopen screens**.
14. **Notification center + template management**.
15. **Marketing screens** (campaigns, segmentation, consent).
16. **AI settings screen**.
17. **Dashboards + Reports**.
18. **Import/Export screens**.

## Parallelization Notes

Within admin-web, groups 3-8 have minimal cross-dependency and can be built in a different relative order if Manish's API sequence pace suggests it. The two Flutter apps' screens (groups 5-6, 9-10) can be built in parallel with admin-web screens once their respective contracts are frozen — they don't block each other.

## Tracking

Update per-module UI status alongside Manish's manifest, using the same `Not Started / Contract Ready / UI In Progress / Integration Testing / Done` states per [coordination/09-daily-progress-format.md](../coordination/09-daily-progress-format.md) §3, recorded in [16-rohit-task-manifest.md](16-rohit-task-manifest.md).
