# Rohit 14 — Mock Data and API Adapter Plan

Rohit's side of [coordination/05-mock-data-contract.md](../coordination/05-mock-data-contract.md) and [coordination/04-api-handover-process.md](../coordination/04-api-handover-process.md).

## 1. Consuming Mocks

Every hook/repository from Manish (§2-3 of [12-frontend-data-contracts.md](../12-frontend-data-contracts.md)) works identically whether backed by a mock file or a live endpoint — Rohit's screen code never branches on mock-vs-live; that's internal to the hook implementation.

## 2. Per-Module Adapter Flag (implemented independently per repo)

Within `citycalls-admin-web`: `lib/config/mockConfig.ts`. Within each mobile repo independently: an equivalent Riverpod/Bloc-level config file, local to that app. Each holds a per-module flag (`serviceRequests: 'mock' | 'live'`, etc.) — set once per module when Manish announces handover (per §6 of [coordination/04-api-handover-process.md](../coordination/04-api-handover-process.md)), not a single global switch, so partially-migrated states are representable during active development. Since there's no shared config package, this flag mechanism is built three separate times (once per frontend repo) — same pattern, independent files.

## 3. Building a Screen Before the Mock Exists

If Rohit needs to start a screen before Manish has published its mock, Rohit does **not** invent a plausible-looking mock independently — that reintroduces exactly the drift risk the contract-first process exists to prevent. Instead: flag it in the daily progress format ([coordination/09-daily-progress-format.md](../coordination/09-daily-progress-format.md)) as blocked, or (if genuinely urgent) pair briefly with Manish to get the contract shape agreed and stubbed, even if the real backend logic isn't built yet.

## 4. Verifying Mock Fidelity

Before switching a module to live, Rohit does a manual pass comparing a few real API responses against the mock the screen was built with — catches any drift Manish's automated contract test might miss (e.g. a field that's technically present but empty in test data, masking a shape mismatch).

## 5. Test Fixtures Post-Handover

Once live, the mock files remain useful as fixtures for component tests (§1 "Component/Widget" row of [19-testing-strategy.md](../19-testing-strategy.md)) — Rohit's widget/component tests import the same mock JSON rather than hand-rolling separate test fixtures, keeping one source of "what does this data look like" throughout the codebase.
