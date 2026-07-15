# Coordination 10 — Integration Checklist

Run per module, at the "integration testing" stage of [04-api-handover-process.md](04-api-handover-process.md), before marking a module `Done` in [22-phase-wise-development-plan.md](../22-phase-wise-development-plan.md).

## Checklist

- [ ] Every endpoint in the module's contract ([11-complete-api-contracts.md](../11-complete-api-contracts.md)) is implemented and live (not still mocked), or explicitly deferred with a documented reason.
- [ ] Contract-schema test passes against live responses (§1 "Contract" row of [19-testing-strategy.md](../19-testing-strategy.md)).
- [ ] Every screen consuming this module's data has been switched from mock to live adapter and manually exercised.
- [ ] Every status/permission rule specific to this module verified against a real user of each relevant role (not just the developer's own admin account).
- [ ] Loading/error/empty states verified against real network conditions (slow connection, actual server error, actual empty-result query) — not just simulated in mock mode.
- [ ] Notifications triggered by this module's actions actually arrive (or correctly show `SKIPPED_INTEGRATION_DISABLED` if the channel is off) per [13-notification-and-template-system.md](../13-notification-and-template-system.md).
- [ ] Audit log entries are actually being written for this module's sensitive actions — spot-checked in the database, not just assumed from code review.
- [ ] Real-time events (if applicable to this module) fire correctly and are received by a second connected client.
- [ ] Offline sync (if applicable — field-execution module) tested with actual airplane-mode toggling, not just simulated offline state.
- [ ] Export/import (if applicable) tested with a real file matching [15-excel-import-export-specification.md](../15-excel-import-export-specification.md) templates.
- [ ] No console errors/warnings in admin-web; no uncaught exceptions in Flutter debug console during the full manual walkthrough.
- [ ] [23-definition-of-done.md](../23-definition-of-done.md) §1 fully checked for every feature in the module.

## Sign-off

Both developers confirm the checklist together (not one developer checking on the other's behalf) before the module is marked `Done`.
