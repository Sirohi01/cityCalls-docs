# Rohit 03 — Shared Component Inventory

Built once per platform (admin-web React components, Flutter widgets), reused across every module's screens — the payoff of consistent list/detail/form shapes defined in [11-complete-api-contracts.md](../11-complete-api-contracts.md) and [12-frontend-data-contracts.md](../12-frontend-data-contracts.md).

## Admin-Web (React)

| Component | Purpose | Consumes |
|---|---|---|
| `DataTable` | Generic paginated/sortable/filterable table | Any list hook's `{data, meta}` shape ([12-frontend-data-contracts.md](../12-frontend-data-contracts.md) §5) |
| `FilterBar` | Search + filter controls above a `DataTable` | Per-entity filterable field list ([08-table-and-filter-specifications.md](08-table-and-filter-specifications.md)) |
| `StatusBadge` | Renders any enum value with its mapped label/color | this repo's local `tokens/` enum map (§7 of [12-frontend-data-contracts.md](../12-frontend-data-contracts.md)) |
| `FormField` (wrapper around RHF + Zod) | One consistent field renderer for select/text/date/etc. per the field-spec template | [07-form-field-specifications.md](07-form-field-specifications.md) rows |
| `DetailPageLayout` | Header + tabs/sections shell for record detail pages | Used by every entity's detail screen |
| `Timeline` | Chronological activity feed | `GET /{entityType}/{id}/timeline` |
| `AssignmentPanel` | Assignee picker with ranked suggestions + override | Assignment engine response shape |
| `FileUploadZone` | Signed-upload dropzone | `useFileUpload` hook |
| `EmptyState`, `ErrorState`, `LoadingState` | Per [11-loading-error-empty-states.md](11-loading-error-empty-states.md) | — |
| `PdfViewerModal` | Preview generated financial/estimate PDFs | `pdfUrl` field |
| `MasterListManager` | Generic CRUD screen for any master type | `/masters/{masterType}` |
| `PermissionGate` | Conditionally renders children based on `usePermission()` | [manish/10](../manish/10-admin-functional-integration-plan.md) §3 |
| `RealtimeIndicator` | Live-update badge/dot for sockets-backed screens | `useRealtimeServiceRequest` etc. |

## Flutter (both mobile apps)

| Widget | Purpose |
|---|---|
| `StatusChip` | Same enum→label/color mapping as web's `StatusBadge` |
| `AppFormField` | Consistent field widget matching the same field-spec template |
| `SyncStatusIndicator` | Shows pending/synced/conflict state (vendor app) |
| `EmptyStateWidget`, `ErrorStateWidget`, `LoadingStateWidget` | Mirrors web equivalents |
| `JobCard` | Summary card for a Service Request in a list (vendor app) |
| `TrackingMapView` | Live technician location map (customer app) |
| `ImageCaptureWidget` | Camera/gallery capture with offline-queue awareness (vendor app) |
| `SignatureCaptureWidget` | Completion-proof signature capture |
| `OtpInputWidget` | OTP entry for login and completion confirmation |

## Governance

A new screen needing a UI pattern not in this inventory adds it here first (with the platform it belongs to) before building a one-off — the inventory is the thing that keeps three separate client codebases feeling like one product.
