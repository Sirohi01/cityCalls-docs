# Manish 10 — Admin Functional Integration Plan

Manish's responsibility for admin-web is the adapter/hook layer, not screens (Rohit owns those) — this document covers what that seam looks like functionally.

## 1. Hook Layer Delivery

For every module in [manish/07-api-development-sequence.md](07-api-development-sequence.md), a matching set of TanStack Query hooks ships in `citycalls-admin-web`'s own `lib/hooks/` (local to that repo, no shared package) before Rohit needs it, per [12-frontend-data-contracts.md](../12-frontend-data-contracts.md) §2 — list hook, detail hook, create/update/delete mutation hooks, and any module-specific action hooks (e.g. `useAssignServiceRequest()`, `useConvertLead()`).

## 2. Real-Time Hook

`useRealtimeServiceRequest(id)` and `useRealtimeBranchDashboard(branchId)` (or equivalent) wrap Socket.IO subscription + TanStack Query cache merging per [12-frontend-data-contracts.md](../12-frontend-data-contracts.md) §8 — delivered alongside the Service Request module (Phase 4).

## 3. Navigation/Permission Gating

`usePermission(module, action)` hook reads the logged-in user's role-permission set (fetched once at login, cached client-side) so Rohit's nav/menu components and action buttons can conditionally render without duplicating the permission matrix in frontend code — the matrix itself lives server-side ([05-user-roles-and-permissions.md](../05-user-roles-and-permissions.md)) and is exposed via `GET /auth/me` (includes resolved permissions for the current user) rather than the frontend re-deriving it from role name alone.

## 4. Global Search

`useGlobalSearch(query)` hits a lightweight cross-entity search endpoint (`GET /search?q=`) — searches Customers, Service Requests, Leads, Calls by their key identifying fields, returns a small ranked result set for the admin shell's search bar.

## 5. File Upload Hook

`useFileUpload(category, entityType, entityId)` wraps the signed-upload flow (§9 of [10-api-standards.md](../10-api-standards.md)) so any form needing an image/document upload (issue images, vendor documents, etc.) uses one consistent hook rather than each screen implementing the signed-URL dance independently.

## 6. Settings/Config Screens

The generic Masters management screen and Config screens (§7-8 of [00-project-overview.md](../00-project-overview.md) context) are functionally backed by the generic `/masters/{masterType}` and `/config/*` endpoints — Manish ensures the hook layer here is genuinely generic (one `useMasterList(masterType)` hook, not one per master type) so Rohit's single generic screen (per [rohit/04-admin-page-list.md](../rohit/04-admin-page-list.md)) works for all master types without per-type frontend code.
