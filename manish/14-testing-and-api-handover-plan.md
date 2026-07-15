# Manish 14 — Testing and API Handover Plan

## 1. Backend Testing Responsibilities

Per [19-testing-strategy.md](../19-testing-strategy.md): Manish owns unit tests (service-layer logic, engines) and integration tests (endpoint behavior against a real test-DB instance) for every module he builds. Test files colocated per [manish/02-backend-folder-structure.md](02-backend-folder-structure.md) §5.

## 2. Handover Execution

Follows [coordination/04-api-handover-process.md](../coordination/04-api-handover-process.md) exactly. Manish's specific responsibilities within that process:
- Steps 1-2 (contract freeze, mock publish) completed **before** starting real implementation of a module, not concurrently — this is what actually unblocks Rohit early rather than being a formality done after the fact.
- Step 4 (real implementation) includes writing the integration tests that prove the contract is met, run in CI before the handover is announced.
- Step 5 (contract test) run locally before flagging a module ready for switch-over.

## 3. Mock Maintenance Responsibility

Manish generates and updates every mock file per [coordination/05-mock-data-contract.md](../coordination/05-mock-data-contract.md) — this is explicitly Manish's task, not something Rohit maintains independently, since drift between "what Rohit assumes the API returns" and "what Manish's mock says" would defeat the purpose of frozen contracts.

## 4. Handover Communication

Each module's handover is announced in the daily progress format ([coordination/09-daily-progress-format.md](../coordination/09-daily-progress-format.md)) with explicit before/after: "Contract Ready" when mocks ship, "Backend In Progress" during implementation, and the switch-over trigger message when live and tested — Rohit doesn't need to poll or guess readiness.

## 5. Regression Protection

Once a module hands over and Rohit switches to live data, Manish's subsequent changes to that module run the full integration test suite for it in CI before merge — a regression that breaks an already-integrated screen is caught before merge, not discovered by Rohit noticing broken UI.
