# Coordination 05 — Mock Data Contract

## 1. Principle

Mock JSON and real API responses **must be structurally identical** — same keys, same types, same nesting, same enum value spelling. Only the actual data values differ. This is what lets Rohit build a whole screen against a mock and have it work unchanged once the real endpoint ships, per [00-project-overview.md](../00-project-overview.md) §7.

## 2. Location & Format (per repo — no shared mocks package)

Since there is no shared package, **each frontend repo maintains its own local mock directory**, independently: `citycalls-admin-web/mocks/{resource}/{operation}.json`, `citycalls-customer-mobile/mocks/{resource}/{operation}.json`, `citycalls-vendor-mobile/mocks/{resource}/{operation}.json` — e.g. `mocks/service-requests/list.json`, `mocks/service-requests/create-response.json`, `mocks/service-requests/detail-{status}.json` (multiple detail fixtures per meaningfully different state, e.g. one for `NEW`, one for `WORK_IN_PROGRESS`, one for `CLOSED`, so Rohit can build/test status-dependent UI without a live backend). A frontend app only needs mocks for the endpoints *it* consumes — the vendor app's mock set and the customer app's mock set will differ even for the same underlying resource (e.g. the vendor app needs `service-requests/assign-response.json`, the customer app doesn't).

Each file is the **exact envelope** from [10-api-standards.md](../10-api-standards.md) §3/§4 — including `meta` and `errors: null` — not just the `data` payload, so mock consumption code is identical to real-response consumption code.

## 3. Error-State Mocks

For every endpoint a repo consumes, also provide at least one error-shape mock (`{operation}-error-validation.json`, `{operation}-error-notfound.json`) so Rohit can build error-state UI ([rohit/11-loading-error-empty-states.md](../rohit/11-loading-error-empty-states.md)) without needing a live backend to deliberately trigger failures.

## 4. Adapter Switch Mechanism

Within each frontend repo, its local API client layer exposes the same typed hook regardless of mock/live mode; an environment/config flag local to that repo (per-module granularity, see [04-api-handover-process.md](04-api-handover-process.md) §3) determines whether the hook's underlying fetch hits the local mock file or the real endpoint. Screens/components never contain mock-vs-live branching logic themselves. This flag and its wiring are independently implemented in each of the three frontend repos — there's no shared adapter code to reuse across them, so Manish sets up this same small pattern three times (once per frontend repo).

## 5. Who Maintains Mocks

Manish creates and updates mock files as the source of truth for a not-yet-built endpoint, in whichever frontend repo(s) consume that endpoint (§1-2 of [04-api-handover-process.md](04-api-handover-process.md)). Once an endpoint is live, its mock file is retained in that repo (not deleted) for use in tests and Storybook-style component development — it becomes a **test fixture** rather than being discarded. Keeping the *same* mock consistent across repos that both consume the same endpoint (e.g. both mobile apps consuming `POST /service-requests/{id}/status`) is a manual discipline — copy the same JSON into each repo's `mocks/` directory rather than letting them silently diverge.

## 6. Keeping Mocks Honest

Each repo's own CI runs contract-schema validation (per [19-testing-strategy.md](../19-testing-strategy.md) §1) against every mock file in that repo on every PR that touches `mocks/**` or that repo's synced copy of the OpenAPI spec, catching drift immediately rather than at integration time. Because there's no single shared mocks directory, this check runs independently per repo — a drift caught in `citycalls-admin-web` doesn't automatically catch the same drift in `citycalls-customer-mobile`, so each repo's CI is equally load-bearing here.
