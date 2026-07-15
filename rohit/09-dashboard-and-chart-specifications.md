# Rohit 09 — Dashboard and Chart Specifications

Per-role dashboards, sourced from the report catalog in [04-modules-and-feature-list.md](../04-modules-and-feature-list.md) M19 and the report endpoints in [11-complete-api-contracts.md](../11-complete-api-contracts.md) §2 (`GET /reports/{reportKey}`).

## Super Admin / Admin Dashboard
- Stat tiles: Total Active Service Requests, New Today, SLA Breaches (live), Pending Assignments, Revenue This Month.
- Charts: Service Request volume trend (line, 30-day), Status distribution (donut), Branch performance comparison (bar), Lead funnel (funnel chart).
- Live feed: recent escalations, recent high-priority requests (real-time via Socket.IO per §8 of [12-frontend-data-contracts.md](../12-frontend-data-contracts.md)).

## Branch Manager Dashboard
Same shape as Admin, filtered to `BRANCH` data scope — same components, different query params, not a separate design.

## Employee/Technician Dashboard (mobile, within Vendor App)
- Today's job count, next job card, completion rate this week (simple stat row, not a full chart dashboard — mobile screen real estate is limited).

## Vendor Owner Dashboard
- Active jobs, acceptance rate, this month's payout total, document-expiry alerts.

## Sales/Marketing Dashboard
- Lead funnel, conversion rate trend, campaign performance summary, source performance bar chart.

## Finance Dashboard
- Outstanding amount, this month's collections, invoice aging breakdown, vendor payout summary.

## Chart Implementation Notes

- Follow the `dataviz` design guidance (color-by-series, accessible palette, consistent light-theme styling matching [02-design-system.md](02-design-system.md)) when building any chart — read that skill's guidance before implementing the first chart component, then reuse the resulting chart primitives across every dashboard rather than restyling per chart.
- Every chart has a loading/empty/error state like any other data-driven component (§ of [11-loading-error-empty-states.md](11-loading-error-empty-states.md)).
- Report data endpoints return pre-aggregated shapes (per §Reports of [manish/05-module-wise-backend-plan.md](../manish/05-module-wise-backend-plan.md)) — charts render directly from the response, no client-side aggregation of raw records.
