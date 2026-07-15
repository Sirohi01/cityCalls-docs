# 12 — Frontend Data Contracts

Bridges [11-complete-api-contracts.md](11-complete-api-contracts.md) (what the API returns) to what Rohit actually consumes in `citycalls-admin-web` (TS) and each Flutter app (Dart) independently. Rohit must not invent field names independently — every field used in a screen traces back to this document or the API contracts doc. Since CityCalls is multi-repo with **no shared package** (confirmed decision, [manish/01-project-and-repository-setup.md](manish/01-project-and-repository-setup.md) §2), everything in this document is implemented **independently in each of the three frontend repos** — the pattern is identical, the code is not shared.

## 1. Type Generation Pipeline (per repo)

```
citycalls-docs: canonical OpenAPI spec (source of truth, maintained alongside 11-complete-api-contracts.md)
        │
        ├── synced into citycalls-admin-web  → local TS types + local Zod schemas
        ├── synced into citycalls-customer-mobile → local Dart models (json_serializable)
        └── synced into citycalls-vendor-mobile   → local Dart models (json_serializable, independently generated — not shared with citycalls-customer-mobile)
```

Each frontend repo runs its own `scripts/sync-contracts.sh` (pulls the current OpenAPI spec from `citycalls-docs`) followed by its own `scripts/generate-types.sh` (local codegen from the synced copy) — three independent pipelines producing three independent sets of generated code, all traceable back to the same spec version. Manish authors the OpenAPI spec changes in `citycalls-docs`; each repo's regeneration is a separate, explicit step run by whoever is working in that repo (typically Manish, since he owns the adapter/data layer in every frontend repo per [coordination/03-code-ownership.md](coordination/03-code-ownership.md)). Rohit imports generated types within whichever repo he's working in, never redefines them locally, and never copies generated code between repos by hand.

## 2. Admin-Web Data Layer Convention (`citycalls-admin-web`)

- **`lib/api/`** (local to this repo): thin Axios wrapper + one TanStack Query hook per endpoint, e.g. `useServiceRequests(filters)`, `useServiceRequest(id)`, `useUpdateServiceRequestStatus()`. Manish writes these hooks (per [00-project-overview.md](00-project-overview.md) §6-7 — the adapter seam is Manish's); Rohit only ever imports a typed hook, never calls `axios`/`fetch` directly in a screen component.
- Every list hook returns `{ data: T[], meta: PaginationMeta, isLoading, isError, error }` — a consistent shape so Rohit's table component ([rohit/03-shared-component-inventory.md](rohit/03-shared-component-inventory.md)) can be built once and reused across every module's list screen **within this repo**.
- Every mutation hook (`useCreateX`, `useUpdateX`) follows TanStack Query's mutation pattern and invalidates the relevant list query key on success — this invalidation logic lives in the hook (Manish's file), not duplicated per screen.

## 3. Flutter Data Layer Convention (each mobile repo, independently)

- `lib/data/api_client.dart` (Dio-based), one repository class per module (`ServiceRequestRepository`, `CallRepository`, ...) exposing typed methods matching the same endpoints — this pattern is built **twice**, once in `citycalls-customer-mobile` and once in `citycalls-vendor-mobile`, each consuming only the subset of endpoints that app actually needs. There is no shared Flutter package between them.
- State management: Riverpod (recommended default — confirm in [21-open-decisions-and-clarifications.md](21-open-decisions-and-clarifications.md) if Rohit prefers Bloc instead; either is acceptable, and since the two apps no longer share code, they could technically even differ from each other, though using the same choice in both is still recommended for Rohit's own consistency, not for technical necessity).
- Offline-first repositories (vendor app only, per [08-system-architecture.md](08-system-architecture.md) §5) wrap the same typed methods with a local-first read/queued-write layer — the widget layer is unaware of online/offline state beyond a `syncStatus` indicator. This offline machinery exists only in `citycalls-vendor-mobile` and is not built in `citycalls-customer-mobile` at all.

## 4. Form Field Specification Template

Every form in the admin panel or either mobile app is specified using this exact table shape before Rohit builds it, regardless of which repo it lives in. Full instance-by-instance specs live in [rohit/07-form-field-specifications.md](rohit/07-form-field-specifications.md); this is the template contract.

| Field key | Label | Input type | Data type | Required | Validation | Placeholder | Default | Master source | Visible when | Editable when | Role permission | API field | DB field |
|---|---|---|---|---|---|---|---|---|---|---|---|---|---|
| `serviceId` | Service | Select (searchable) | ObjectId string | Yes | must be an active service | "Select a service" | — | `GET /services?active=true` | always | on create only | `serviceRequests.create` | `serviceId` | `service_requests.serviceId` |

Every field in every form must have a filled row like this before implementation begins on that form — no field is added to a screen ad hoc.

## 5. List/Table Data Shape Convention

List endpoints return a **flattened, display-ready shape** (see [11-complete-api-contracts.md](11-complete-api-contracts.md) §1.3) rather than the full nested document — this means Rohit's table components never need to reach into nested objects or resolve a second lookup just to show a customer's name next to a service request row. If a table needs a field not present in the flattened shape, that's a contract-change request (Manish adds it to the list endpoint's projection in `citycalls-docs` + `citycalls-api`), not something Rohit works around client-side by calling the detail endpoint per row.

## 6. Error Display Mapping

Every form submission handler, in whichever repo, maps the standard error envelope ([10-api-standards.md](10-api-standards.md) §4) `errors[].field` to the corresponding React Hook Form / Flutter form field by matching `field` against the form's field keys (which are the same `fieldKey` values as §4's template) — this is why field-key consistency across DB/API/form-spec is load-bearing, not cosmetic, and why it matters even more now that each repo implements this mapping independently rather than sharing one implementation.

## 7. Enum → UI Label Mapping (duplicated per repo — accepted trade-off)

Enum wire values (`SCREAMING_SNAKE_CASE`, per [coordination/06-naming-conventions.md](coordination/06-naming-conventions.md)) are never shown directly in UI. Since there is no shared `ui-tokens` package, **each frontend repo maintains its own local `{enumValue: displayLabel}` map** (`tokens/` in admin-web, `lib/tokens/` in each mobile app), all built against the same canonical enum list in [coordination/06-naming-conventions.md](coordination/06-naming-conventions.md) §4 and the same visual mapping guidance in [rohit/02-design-system.md](rohit/02-design-system.md) §3. Keeping these three independent copies visually consistent is a manual QA responsibility (cross-repo visual comparison), not something enforced by shared code — see [coordination/03-code-ownership.md](coordination/03-code-ownership.md) §4.

## 8. Real-Time Data Binding

Screens that reflect live state (Service Request detail, Branch dashboard, technician tracking) subscribe to the relevant Socket.IO room (per [08-system-architecture.md](08-system-architecture.md) §4) and merge incoming events into the existing TanStack Query cache / Riverpod state rather than triggering a full refetch. Manish provides a small `useRealtimeServiceRequest(id)` hook in `citycalls-admin-web` and independently-implemented equivalent Flutter providers in each of `citycalls-customer-mobile` and `citycalls-vendor-mobile` — three separate implementations of the same pattern, so Rohit doesn't hand-roll socket handling per screen within any one of them, even though the underlying socket-handling code isn't shared across repos.
