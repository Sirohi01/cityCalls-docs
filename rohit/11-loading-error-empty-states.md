# Rohit 11 — Loading, Error, and Empty States

Every data-driven screen/component implements all three states — no screen ships with only the happy path, per [23-definition-of-done.md](../23-definition-of-done.md) §1.

## 1. Loading States

- **List/table**: skeleton rows matching the table's column structure (not a generic spinner) — gives the user a sense of layout before data arrives.
- **Detail page**: skeleton matching the page's section layout.
- **Form submission**: submit button shows a spinner + disables (prevents double-submit, complements the idempotency-key mechanism in [10-api-standards.md](../10-api-standards.md) §11 as a UX-level safeguard).
- **Dashboard tiles/charts**: skeleton shimmer per tile, independently loading (a slow chart doesn't block fast stat tiles from appearing).

## 2. Error States

- **List/table load failure**: `ErrorState` component with the error message (from the standard envelope, §4 of [10-api-standards.md](../10-api-standards.md)) and a retry action.
- **Form validation errors**: inline, per-field, per §6 of [12-frontend-data-contracts.md](../12-frontend-data-contracts.md) — never a single toast summarizing multiple field errors.
- **Mutation failure (non-validation, e.g. 409/500)**: toast/banner with the error message and, where applicable (e.g. `INVALID_TRANSITION`), the recovery context from the error payload (§1.4 of [11-complete-api-contracts.md](../11-complete-api-contracts.md) — shows `allowedTransitions` so the user isn't stuck guessing).
- **Network/offline (admin-web)**: a global banner when the browser goes offline, distinct from a specific request's error state.

## 3. Empty States

Two distinct empty-state messages per list, not one generic "No data":
- **No records exist yet** (e.g. a brand-new branch with zero service requests) — friendly, often with a primary CTA ("Create your first Service Request").
- **No records match your current filters** — different messaging ("Try adjusting your filters"), with a "Clear filters" action.

## 4. Business-Flag States (not errors — per §6 of [18-error-handling-standards.md](../18-error-handling-standards.md))

`PIN_CODE_NOT_SERVICEABLE`, `REOPEN_WINDOW_EXPIRED`, `INTEGRATION_DISABLED`-derived states render as **inline informational banners**, styled distinctly from both error and empty states (they're neither a failure nor an absence of data — they're a business condition the user needs to understand and act on).

## 5. Offline States (Vendor App specific)

Per [06-vendor-app-screen-list.md](06-vendor-app-screen-list.md) — not an error state. A `SyncStatusIndicator` (pending/synced/conflict) replaces the error/loading treatment for actions in the offline-first execution flow; see [12-responsive-design-guidelines.md](12-responsive-design-guidelines.md) for exact visual treatment.

## 6. Component-Level Enforcement

`DataTable`, `AppFormField`, and equivalents in [03-shared-component-inventory.md](03-shared-component-inventory.md) accept `isLoading`/`isError`/`isEmpty` props and render the correct state internally — individual screens pass state through, they don't reimplement these three states per screen.
