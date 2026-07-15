# Coordination 04 — API Handover Process

Describes exactly how an endpoint moves from "documented" to "Rohit is using the real thing," per module, following the sequence in [00-project-overview.md](../00-project-overview.md) §7.

## 1. Steps

1. **Contract freeze**: Manish writes/updates the endpoint's full entry in [11-complete-api-contracts.md](../11-complete-api-contracts.md) (following the template in [10-api-standards.md](../10-api-standards.md) §13) and the canonical OpenAPI spec — both committed to the `citycalls-docs` repo.
2. **Mock publish**: Manish (or a generation script from the OpenAPI spec) produces the mock JSON file **locally within each consuming frontend repo** (`citycalls-admin-web/mocks/{endpoint}.json`, and independently in `citycalls-customer-mobile/mocks/` and `citycalls-vendor-mobile/mocks/` as applicable) — matching the frozen contract exactly. Since there's no shared mocks package, this is a small copy/generation step run per repo, not one file three repos import.
3. **Rohit builds**: screens/components in each frontend repo built against that repo's local mock adapter (`USE_MOCK=true`, per [05-mock-data-contract.md](05-mock-data-contract.md)), no waiting on real implementation.
4. **Manish implements**: real endpoint in `citycalls-api` built to produce byte-for-byte structurally identical responses to the mock (values differ, shape doesn't).
5. **Contract test**: each frontend repo's CI runs JSON-schema validation (per [19-testing-strategy.md](../19-testing-strategy.md) §1 "Contract" row) against both its local mock and a live call to the new endpoint — must pass before that repo's handover is considered complete. Because repos are independent, this test runs and passes **per repo**, not once centrally.
6. **Switch-over**: Rohit flips the adapter flag for that module from mock to live in each frontend repo independently (`USE_MOCK=false` scoped per module, not a single global flag — see [rohit/14-mock-data-and-api-adapter-plan.md](../rohit/14-mock-data-and-api-adapter-plan.md)) and verifies the UI behaves identically against real data.
7. **Sign-off**: both developers confirm the module's screens work end-to-end against the live API in each affected frontend repo; module marked complete in [22-phase-wise-development-plan.md](../22-phase-wise-development-plan.md) tracking.

## 2. When Reality Diverges from the Mock

If, during implementation, Manish discovers the frozen contract can't be satisfied as written (e.g. a field needs to be paginated separately, or an enum needs an additional value), that's a **contract change**, not a silent deviation — goes through [08-change-request-process.md](08-change-request-process.md) before the real endpoint ships, so the mock, the doc, and the implementation stay in lockstep rather than the doc becoming stale.

## 3. Partial Modules and Partial Repos

A module's endpoints don't all have to hand over simultaneously — list/detail endpoints can go live while a bulk-action endpoint is still mocked, as long as each individual endpoint's switch-over follows §1 fully. The mock-vs-live flag is per-endpoint-group granularity in the adapter plan, not all-or-nothing per module.

Additionally, because `citycalls-admin-web`, `citycalls-customer-mobile`, and `citycalls-vendor-mobile` are independent repos, one may switch a module to live while another is still on mock — e.g. admin-web's Service Request dispatch screen goes live before the vendor app's job-acceptance screen does. This is expected and tracked per-repo in [09-daily-progress-format.md](09-daily-progress-format.md) §3, not treated as an inconsistency to force into lockstep.

## 4. Rollback

If a live endpoint ships with a bug that breaks the UI, Rohit can flip the flag back to mock for that endpoint group without a deploy, buying Manish time to fix forward — this is the practical reason the mock/live switch is a runtime flag, not something removed once real APIs exist.
