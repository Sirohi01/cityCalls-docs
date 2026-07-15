# Coordination 08 — Change Request Process

Governs any change to a frozen contract: API shape, field name, enum value, DB schema field, status/permission rule. Applies whether the change originates from Manish (backend reality diverged), Rohit (UI need reveals a gap), or the user (new/changed business requirement).

## 1. When This Process Applies

- Adding, removing, or renaming a field on an existing API request/response.
- Adding, removing, or renaming an enum value (statuses, roles, etc. from [coordination/06-naming-conventions.md](06-naming-conventions.md) §4).
- Changing a status-transition rule in [07-status-transition-matrix.md](../07-status-transition-matrix.md).
- Changing a permission/data-scope rule in [05-user-roles-and-permissions.md](../05-user-roles-and-permissions.md).
- Any change that would make an already-built mock or UI component structurally incorrect.

**Does not apply** to internal implementation details invisible to the contract (e.g. refactoring a backend service function's internals, restyling a component) — those follow normal code-ownership rules with no cross-review required.

## 2. Steps (multi-repo)

1. **Propose**: the requester documents the change directly in the relevant `citycalls-docs` file (not a side conversation) — what's changing, why, and what it breaks. This is a PR against the `citycalls-docs` repo.
2. **Impact check**: the other developer checks what currently depends on the old contract across **every repo that consumes it** (existing UI screens in up to three frontend repos, existing mocks, existing tests, existing backend implementation) and flags the full blast radius — this check is wider than in a monorepo, since nothing automatically shows "who imports this type."
3. **Agree**: both developers (and the user, if it's a business-rule change, not just a technical one) agree on the new shape.
4. **Update in order**: `citycalls-docs` (doc + OpenAPI spec) merged first → `citycalls-api` implementation → each affected frontend repo's local sync + regeneration + implementation, in whatever order [manish/07-api-development-sequence.md](../manish/07-api-development-sequence.md)/[rohit/13-ui-development-sequence.md](../rohit/13-ui-development-sequence.md) calls for. The doc is updated *first*, never after the fact as documentation of what already shipped.
5. **Regenerate**: each consuming repo independently re-runs its `scripts/sync-contracts.sh` and regenerates its local TS/Dart types per [12-frontend-data-contracts.md](../12-frontend-data-contracts.md) §1 — this is not one step, it's one step *per repo*, and a repo that skips it is now silently out of sync (tracked per §7 of [02-git-and-branch-strategy.md](../coordination/02-git-and-branch-strategy.md)).
6. **Verify**: each repo's own contract test (per [19-testing-strategy.md](../19-testing-strategy.md)) passes against its updated local mock and, once implemented, the live endpoint.

## 2a. Tracking Multi-Repo Propagation

Because a single contract change now touches up to four downstream repos independently, log the change in the daily progress format ([09-daily-progress-format.md](09-daily-progress-format.md)) with a per-repo status (`citycalls-api: done`, `citycalls-admin-web: pending`, `citycalls-customer-mobile: n/a`, `citycalls-vendor-mobile: done`) until every affected repo has caught up — a contract change is not "complete" until every repo that consumes it has synced, even though each repo's own PR merges independently.

## 3. Versioning Decision

Per [10-api-standards.md](../10-api-standards.md) §1: additive/backward-compatible changes ship in place; breaking changes to an endpoint already consumed by shipped mobile app versions require a `v2` path rather than breaking existing clients — this matters more for mobile than web since app-store review cycles mean old mobile clients can be in the field for weeks after a backend change.

## 4. Emergency Changes

A production bug requiring an urgent contract fix still follows §2 (impact check) at minimum, compressed to same-day, but is never skipped — an urgent unilateral contract change is exactly the scenario this process exists to prevent, not an exception to it.

## 5. Record-Keeping

Every change request that affects an already-approved doc is a normal PR against that doc with a clear commit message describing the change — git history on `docs/` is the change log, no separate change-request ticket system is introduced for a two-person team.
