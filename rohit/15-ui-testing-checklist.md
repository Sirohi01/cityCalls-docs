# Rohit 15 — UI Testing Checklist

Run per screen before marking it done (feeds into [23-definition-of-done.md](../23-definition-of-done.md) §1).

## Checklist (every screen)

- [ ] Matches its form-field/table spec exactly ([07-form-field-specifications.md](07-form-field-specifications.md) / [08-table-and-filter-specifications.md](08-table-and-filter-specifications.md)) — no invented field, no missing required field.
- [ ] Loading, error, and both empty-state variants all verified visually (§ of [11-loading-error-empty-states.md](11-loading-error-empty-states.md)), not just the happy path.
- [ ] Form validation: every required field blocks submit when empty; every validation rule triggers its correct inline error; server-side validation errors map to the correct field (§6 of [12-frontend-data-contracts.md](../12-frontend-data-contracts.md)).
- [ ] Permission gating verified: logged in as a role that shouldn't see/edit this screen or action, confirm it's actually hidden/disabled, not just visually deemphasized.
- [ ] Responsive at the breakpoints specified in [12-responsive-design-guidelines.md](12-responsive-design-guidelines.md).
- [ ] No console errors/warnings (admin-web) or uncaught exceptions (Flutter debug console) during a full interaction pass.
- [ ] Enum values render via the shared label/color map (§7 of [12-frontend-data-contracts.md](../12-frontend-data-contracts.md)) — no raw `SCREAMING_SNAKE_CASE` string visible anywhere in the UI.
- [ ] Real-time updates (if applicable) verified with a second session/tab showing the live update.
- [ ] Component/widget tests written for non-trivial logic (conditional rendering, form validation branches) per [19-testing-strategy.md](../19-testing-strategy.md).
- [ ] Verified against both mock and (once available) live data, per [14-mock-data-and-api-adapter-plan.md](14-mock-data-and-api-adapter-plan.md) §4.

## Vendor App Additional Checklist

- [ ] Every execution-flow action tested in actual airplane mode, confirming local queue + `SyncStatusIndicator` behave correctly.
- [ ] Sync-conflict scenario manually triggered (two devices/sessions racing a status change) and the conflict UI verified.

## Customer App Additional Checklist

- [ ] Booking flow tested against an uncovered pin code, confirming the `PIN_CODE_NOT_SERVICEABLE` inline banner (not an error toast) per [11-loading-error-empty-states.md](11-loading-error-empty-states.md) §4.
- [ ] Push notification receipt tested on a physical device (simulators don't reliably deliver push).

## Cross-Browser/Device Note

Admin-web: verified on the two most recent versions of Chrome and one other major browser (Firefox or Edge) at minimum. Mobile apps: verified on at least one physical Android and one physical iOS device before a module is marked done, not simulator-only.
