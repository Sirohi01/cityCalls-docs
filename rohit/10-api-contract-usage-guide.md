# Rohit 10 — API Contract Usage Guide

Practical guide for consuming Manish's contracts — the "how" that complements [12-frontend-data-contracts.md](../12-frontend-data-contracts.md)'s "what."

## 1. Never Do This

- Never call `axios`/`fetch`/`Dio` directly in a screen component — always use the provided hook/repository method.
- Never hand-type a field name that isn't in the generated types — if you need a field that doesn't exist yet, that's a contract-change request to Manish (per [coordination/08-change-request-process.md](../coordination/08-change-request-process.md)), not something to work around by casting to `any`.
- Never reach into a nested field on a list-endpoint response — list shapes are flattened on purpose (§5 of [12-frontend-data-contracts.md](../12-frontend-data-contracts.md)); if a table needs a field not present, request it be added to that endpoint's projection.
- Never hardcode an enum's display label inline — always resolve through this repo's local `tokens/` enum map (§7 of [12-frontend-data-contracts.md](../12-frontend-data-contracts.md)); there is no shared `ui-tokens` package, so each repo has its own copy built to the same spec.

## 2. Standard Screen Pattern (Admin-Web)

```tsx
const { data, meta, isLoading, isError, error } = useServiceRequests(filters);
if (isLoading) return <LoadingState />;
if (isError) return <ErrorState error={error} />;
if (data.length === 0) return <EmptyState variant={hasActiveFilters ? 'no-matches' : 'no-records'} />;
return <DataTable columns={serviceRequestColumns} rows={data} meta={meta} />;
```

## 3. Mutation Pattern

```tsx
const { mutate, isPending } = useUpdateServiceRequestStatus();
mutate({ id, toStatus: 'TECHNICIAN_ARRIVED', geo }, {
  onError: (err) => mapApiErrorToFormOrToast(err), // per §6 of 12-frontend-data-contracts.md
});
```

## 4. Handling the "Not an Error" Cases

Per §6 of [18-error-handling-standards.md](../18-error-handling-standards.md), responses like `PIN_CODE_NOT_SERVICEABLE` or `INTEGRATION_DISABLED` come back as `success: true` with an informational flag — check for these explicitly and render an inline banner, not the generic error-toast path.

## 5. Flutter Equivalent Pattern

```dart
final result = await ref.watch(serviceRequestListProvider(filters).future);
// result.data, result.meta, mirrors the same shape as the TS hook
```

Repository methods return the same envelope shape deserialized into Dart models — error handling mirrors §4 via a sealed `ApiResult` type distinguishing success/validation-error/business-flag/exception.

## 6. When a Contract Doc and Reality Disagree

If the live API doesn't match what [11-complete-api-contracts.md](../11-complete-api-contracts.md) says, that's a bug in the handover (per [coordination/04-api-handover-process.md](../coordination/04-api-handover-process.md) §5) — flag it to Manish rather than adapting the frontend to match the wrong live behavior, which would just hide the bug.
