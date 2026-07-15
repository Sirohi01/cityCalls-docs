# Rohit 08 — Table and Filter Specifications

Per-entity table column and filter specs, built on the `DataTable`/`FilterBar` components from [03-shared-component-inventory.md](03-shared-component-inventory.md) and the list-endpoint conventions in [10-api-standards.md](../10-api-standards.md) §5-7.

## Service Request List Table

**Columns**: Number, Customer Name, Service, Branch, Status (badge), Priority (badge), Assignee, Scheduled Date, Created At. **Row action**: click → detail page.

**Filters**: Status (multi-select), Branch (select, scoped to caller's data scope), Priority (multi-select), Service Category (select), Date Range (createdAt), Assignee (select, branch-scoped), Search (`q`: matches number, customer name, customer mobile).

**Default sort**: `createdAt:desc`. **Saved filters**: supported (per-user, named presets) — persisted via `POST /users/{id}/saved-filters`.

## Customer List Table

**Columns**: Name, Mobile, City, Customer Type, Total Service Requests (denormalized count), Tags, Created At. **Filters**: Customer Type, City, Tags, Blacklisted (toggle), Search (`q`: name/mobile/email).

## Lead List Table

**Columns**: Lead Number, Customer Name, Stage (badge), Source, Priority, Owner, Follow-up Date, Created At. **Filters**: Stage (multi-select), Source, Owner (scoped), Priority, Follow-up Date Range, Search.

## Call List Table

**Columns**: Call Number, Call Type, Direction, Customer Name, Caller Number, Date/Time, Created By, Linked SR/Lead. **Filters**: Call Type, Direction, Date Range, Created By, Search.

## Vendor List Table

**Columns**: Company Name, Coverage Area (summary), Services Offered (summary), Active Jobs Count, Performance Rating, Status (Active/Blacklisted). **Filters**: Active/Blacklisted, Service Offered, Coverage Pin Code.

## Estimate/Invoice List Tables

**Columns**: Number, Customer Name, Service Request Number, Status (badge), Total, Created At. **Filters**: Status, Date Range, Branch, Search.

## Generic Conventions (apply to every table not explicitly listed above)

- Column visibility toggle available on every table (per [04-modules-and-feature-list.md](../04-modules-and-feature-list.md) M19 requirement).
- Export button always present, respects current filter state, per §1 of [15-excel-import-export-specification.md](../15-excel-import-export-specification.md).
- Pagination controls per [10-api-standards.md](../10-api-standards.md) §5 (page-number based, not infinite scroll, for admin tables — consistency with exportable/printable expectations).
- Empty state per [11-loading-error-empty-states.md](11-loading-error-empty-states.md) distinguishes "no records exist yet" from "no records match your filters."
