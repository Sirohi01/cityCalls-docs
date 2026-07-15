# 19 — Testing Strategy

## 1. Test Layers

| Layer | Owner | Scope | Tooling |
|---|---|---|---|
| Unit | Manish (backend), Rohit (frontend logic) | Pure functions: policy resolution, numbering, status-transition validation, form validation schemas | Jest (TS), `flutter_test` (Dart) |
| Integration | Manish | API endpoint behavior against a real (test) MongoDB instance — request in, DB state + response out | Jest + Supertest, `mongodb-memory-server` or a disposable test DB |
| Contract | Both | Mock JSON (Rohit's fixtures) structurally matches real API responses | Shared JSON schema validation run in CI against both mock files and live endpoint responses |
| Component/Widget | Rohit | Individual UI components render correctly across states (loading/error/empty/populated) | React Testing Library (admin-web), `flutter_test` widget tests |
| End-to-end | Shared, added per phase | Critical user journeys (see §3) | Playwright (admin-web), `integration_test` (Flutter) |
| Manual/UAT | User (Manish as product owner) | Full workflow walkthroughs before phase sign-off | Per [23-definition-of-done.md](23-definition-of-done.md) |

## 2. What Must Be Unit/Integration Tested (non-negotiable)

- Every status-transition rule in [07-status-transition-matrix.md](07-status-transition-matrix.md) — both the allowed and the rejected paths (a test asserting `INVALID_TRANSITION` fires for each disallowed pair, not just happy-path coverage).
- Policy resolution order in [09-database-architecture.md](09-database-architecture.md) §5 (customer > contract > brand > product > service > branch > global) — tested with overlapping policies at multiple scopes to confirm most-specific-wins.
- Numbering engine concurrency (two simultaneous requests never receive the same sequence number) — tested with concurrent `getNextNumber` calls.
- Data-scope query filtering (§2 of [17-security-and-audit.md](17-security-and-audit.md)) — a test per role confirming it cannot see/act on out-of-scope records, not just that in-scope access works.
- Integration-disabled degradation paths (§ of [14-integration-architecture.md](14-integration-architecture.md)) — every optional-integration code path tested with the integration both enabled and disabled, confirming the core action still succeeds when disabled.

## 3. Critical End-to-End Journeys (minimum set, expand per phase)

1. Customer books a service → staff assigns → technician completes (offline-then-synced) → invoice generated → payment recorded → happy call → closed.
2. Lead created → qualified → converted to Service Request → same downstream flow as #1.
3. Closed Service Request reopened within policy window → new linked request created → routed back to original assignee.
4. Assignment rejected → reassignment → accepted → continues normally.
5. WhatsApp/email disabled mid-flow → booking/assignment/completion still succeed, notifications show `SKIPPED_INTEGRATION_DISABLED`.

## 4. Test Data

Seed scripts (`scripts/seed-test-data.ts`) create a representative dataset (branches, services, a few customers/products/vendors) reused across integration and E2E tests — not hand-crafted per test file, to avoid drift between test fixtures and real schema.

## 5. CI Gates

Every PR runs: type-check (TS strict mode + Dart analyze), lint, unit tests, integration tests (backend PRs), contract-schema validation (any PR touching API contracts or mocks). E2E runs on merge to `develop`/`main` (not every PR, to keep PR feedback fast) — see [coordination/02-git-and-branch-strategy.md](coordination/02-git-and-branch-strategy.md).

## 6. Coverage Expectations

No hard percentage target enforced in CI for v1 (avoids incentivizing low-value tests written purely to hit a number) — instead, §2's non-negotiable list is the actual bar, reviewed in PR review per [coordination/07-merge-and-conflict-prevention.md](coordination/07-merge-and-conflict-prevention.md).
