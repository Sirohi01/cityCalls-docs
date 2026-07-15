# Manish 03 — Database Model Implementation Plan

## 1. Build Order (matches [22-phase-wise-development-plan.md](../22-phase-wise-development-plan.md))

1. `User`, `Branch`, `SubBranch`, `Team` (Phase 1)
2. Generic `Master` model (single schema, `masterType` discriminator, per §3 of [09-database-architecture.md](../09-database-architecture.md)) + seed registry for: `service_categories, brands, product_types, symptoms, defects, solutions, parts, units, tax_rates, priorities, lead_sources, call_types, appointment_slots, payment_methods`
3. `NumberingSeries`, `Policy` (Phase 1, unblocks numbering/reopen logic everywhere downstream)
4. `Employee`, `Vendor`, `VendorTechnician` (Phase 1/2)
5. `Customer`, `CustomerProduct` (Phase 2)
6. `Service` (dynamic catalog, Phase 2)
7. `Call` (Phase 3)
8. `Lead` (Phase 3)
9. `ServiceRequest`, `AssignmentHistory` (Phase 4)
10. `ServiceVisit` (Phase 5)
11. `Estimate`, `ProformaInvoice`, `Invoice`, `PaymentReceipt`, `CreditNote`, `DebitNote` (Phase 6)
12. `HappyCall`, `ReopenRecord` (Phase 7)
13. `NotificationTemplate`, `Notification`, `Campaign` (Phase 8)
14. `ActivityLog` (built alongside Phase 1, since audit logging is required from the first sensitive action onward — not deferred like other Phase-ordered models)
15. `Contract` (schema stub only, no phase — see §5 of [09-database-architecture.md](../09-database-architecture.md))

## 2. Generic Master Registry Pattern

```ts
// lib/masterRegistry.ts
export const MASTER_TYPES = [
  'SERVICE_CATEGORY', 'BRAND', 'PRODUCT_TYPE', 'SYMPTOM', 'DEFECT',
  'SOLUTION', 'PART', 'UNIT', 'TAX_RATE', 'PRIORITY', 'LEAD_SOURCE',
  'CALL_TYPE', 'APPOINTMENT_SLOT', 'PAYMENT_METHOD'
] as const;
```
Adding a new master type is adding a string to this array plus any `meta` shape validation specific to it — never a new Mongoose model/collection. `GET/POST /masters/{masterType}` routes validate `masterType` against this registry.

## 3. Role/Permission Seed

`role_permissions` seeded from the full matrix in [05-user-roles-and-permissions.md](../05-user-roles-and-permissions.md) §4-6, one document per `{role, module, action}` combination with its `dataScope` — seed script (`scripts/seed-role-permissions.ts`) generates this from a structured source table (kept in the script itself, matching the doc) rather than 300+ hand-written seed documents.

## 4. Status/Transition Seed

`status_transitions` seeded from [07-status-transition-matrix.md](../07-status-transition-matrix.md) §2-3, one document per `{entityType, fromStatus, toStatus}` with `allowedRoles[]`.

## 5. Index Creation

Indexes per §6 of [09-database-architecture.md](../09-database-architecture.md) declared on the Mongoose schemas directly (`schema.index({...})`), applied via `mongoose.connection.syncIndexes()` in a startup/migration script rather than manual `mongo shell` commands, so index definitions live in version control.

## 6. Schema Versioning

No formal migration framework mandated for v1 given team size — schema changes are additive where possible (new optional fields); any field removal/rename runs a one-off migration script under `scripts/migrations/{timestamp}-{description}.ts`, logged and run manually per environment (not auto-run on deploy) to avoid an unreviewed migration hitting production data.
