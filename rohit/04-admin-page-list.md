# Rohit 04 — Admin Page List

Full screen inventory for the `citycalls-admin-web` repo, organized by module (matching [04-modules-and-feature-list.md](../04-modules-and-feature-list.md)). Each entry expands into a full form-field/table spec in [07-form-field-specifications.md](07-form-field-specifications.md)/[08-table-and-filter-specifications.md](08-table-and-filter-specifications.md) as it's built, per [13-ui-development-sequence.md](13-ui-development-sequence.md).

| Module | Pages |
|---|---|
| Auth | Login, Forgot Password, Reset Password |
| Dashboard | Role-specific landing dashboard (varies by role per [09-dashboard-and-chart-specifications.md](09-dashboard-and-chart-specifications.md)) |
| Organization | Branch list/detail/create-edit, Sub-Branch list/detail, Team list/detail |
| Users & Roles | User list/detail/create-edit, Role/Permission matrix editor |
| Employees | Employee list/detail/create-edit, Availability calendar |
| Vendors | Vendor list/detail/create-edit, Vendor documents, Vendor performance, Vendor invoices/payouts |
| Customers | Customer list/detail/create-edit, Address management, Product management, Duplicate review |
| Catalog | Service list/detail/create-edit, Brand/Product Type/Model management |
| Masters & Config | Generic Master management (per master type), Policy configuration, Numbering series configuration, Integration settings |
| Calls | Call list, Call entry (per call type), Call detail/timeline |
| Leads | Lead pipeline (kanban) view, Lead list, Lead detail, Bulk assign/import |
| Service Requests | Service Request list, Create/Book, Detail (with tabs: Overview, Assignment, Visits, Financial, Timeline), Dispatch/Assignment screen |
| Field Execution (admin visibility) | Service Visit detail view (read + limited edit) |
| Reopen | Reopen review screen |
| Finance | Estimate list/detail/create, Proforma list/detail, Invoice list/detail, Payment recording, Credit/Debit Note list/detail |
| Happy Calls | Happy Call list/entry |
| Notifications | Notification center (bell dropdown + full list page), Notification Template management |
| Marketing | Campaign list/create, Audience segmentation, Consent management, Delivery/analytics dashboard |
| AI | AI Settings |
| Reports | Report catalog landing, per-report detail view with filters/export |
| Audit | Audit Log viewer |
| Import/Export | Import wizard (per entity), Export configuration |

## Navigation Structure

Left sidebar, grouped by the module groupings above, filtered per-user by `usePermission()` (§3 of [manish/10-admin-functional-integration-plan.md](../manish/10-admin-functional-integration-plan.md)) — a role with no permission on a module doesn't see that nav group at all, not a disabled/greyed-out link.
