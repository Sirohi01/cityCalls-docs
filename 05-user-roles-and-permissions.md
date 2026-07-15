# 05 — User Roles and Permissions

## 1. Role List (canonical, `role` enum values in SCREAMING_SNAKE_CASE per [coordination/06-naming-conventions.md](coordination/06-naming-conventions.md))

| Role code | Display name | Belongs to |
|---|---|---|
| `SUPER_ADMIN` | Super Admin | Organization |
| `ADMIN` | Admin | Organization |
| `OPERATIONS_ADMIN` | Operations Admin | Organization |
| `BRANCH_ADMIN` | Branch Admin | Branch |
| `SUB_BRANCH_ADMIN` | Sub-Branch Admin | Sub-Branch |
| `BRANCH_MANAGER` | Branch Manager | Branch |
| `TEAM_LEAD` | Team Lead | Team |
| `EMPLOYEE` | Employee | Team/Branch |
| `TECHNICIAN` | Technician (internal) | Team/Branch |
| `CALL_EXECUTIVE` | Call Executive | Branch |
| `CUSTOMER_SUPPORT_EXECUTIVE` | Customer Support Executive | Branch |
| `HAPPY_CALL_EXECUTIVE` | Happy Call Executive | Branch |
| `SALES_EXECUTIVE` | Sales Executive | Branch |
| `MARKETING_EXECUTIVE` | Marketing Executive | Organization |
| `FINANCE_EXECUTIVE` | Finance Executive | Organization/Branch |
| `ACCOUNTANT` | Accountant | Organization/Branch |
| `VENDOR_OWNER` | Vendor Owner | Vendor |
| `VENDOR_MANAGER` | Vendor Manager | Vendor |
| `VENDOR_TECHNICIAN` | Vendor Technician | Vendor |
| `OUTSOURCED_PARTNER` | Outsourced Partner | External |
| `CUSTOMER` | Customer (individual) | Self |
| `BUSINESS_CUSTOMER` | Business Customer | Self |

**Role model decision**: one role per `User` for v1 (a person needing two roles gets two logins, or the org-chart is restructured) — see [21-open-decisions-and-clarifications.md](21-open-decisions-and-clarifications.md) if multi-role-per-user is later required; this is deliberately simple to avoid a permission-resolution edge case explosion in v1.

## 2. Data-Scope Levels

Every permission is `{module, action, dataScope}`. Data scope, from narrowest to widest:

| Scope | Meaning |
|---|---|
| `OWN` | Records the user created or is directly assigned to |
| `TEAM` | Records assigned to the user's Team |
| `SUB_BRANCH` | Records under the user's Sub-Branch |
| `BRANCH` | Records under the user's Branch (includes its Sub-Branches) |
| `VENDOR` | Records belonging to the user's Vendor company |
| `ALL` | Organization-wide (Super Admin, Admin, Operations Admin only by default) |

A user's effective visible record set = permission's data scope evaluated against their own Branch/Sub-Branch/Team/Vendor assignment on their `User` document. This is enforced as a **query-level filter in every list/detail endpoint**, not a UI-only restriction — see [17-security-and-audit.md](17-security-and-audit.md) §authorization.

## 3. Action Verbs (used consistently across the permission matrix)

`view`, `create`, `edit`, `delete`, `assign`, `reassign`, `approve`, `reject`, `export`, `import`, `downloadPdf`, `viewFinancial`, `viewContactDetails`, `viewRecordings`, `sendWhatsapp`, `sendEmail`, `bulkAction`, `manageSettings`, `viewAuditLog`.

## 4. Permission Matrix (module × role, high level)

Full CRUD-by-action detail per module lives in each module's API contract doc; this table sets the default scope so `manish` can seed `role_permissions` and `rohit` can drive nav/menu visibility identically.

| Module | SUPER_ADMIN | ADMIN | BRANCH_MANAGER | TEAM_LEAD/EMPLOYEE | CALL_EXECUTIVE | SALES_EXECUTIVE | FINANCE_EXECUTIVE | VENDOR_OWNER/MANAGER | VENDOR_TECHNICIAN | CUSTOMER |
|---|---|---|---|---|---|---|---|---|---|---|
| Org (Branch/Sub-Branch/Team) | full/ALL | full/ALL | view+edit/BRANCH | view/TEAM | — | — | — | — | — | — |
| Employees & Vendors | full/ALL | full/ALL | manage/BRANCH | view/TEAM | — | — | — | self-profile/VENDOR | self-profile/OWN | — |
| Masters & Config | full/ALL | full/ALL | limited overrides/BRANCH | — | — | — | tax/pricing view | — | — | — |
| Customers | full/ALL | full/ALL | view+edit/BRANCH | view/TEAM (assigned only) | create+edit/BRANCH | view+edit/OWN leads' customers | viewFinancial/BRANCH | view assigned/VENDOR | view assigned job's customer/OWN (contact gated, see §5) | self/OWN |
| Calls | full/ALL | full/ALL | view/BRANCH | view assigned/OWN | create+edit/BRANCH | create (sales calls)/OWN | — | — | — | initiate via app (creates booking, not a Call record) |
| Leads | full/ALL | full/ALL | view/BRANCH | — | create/BRANCH | full/OWN+BRANCH | — | — | — | — |
| Service Requests | full/ALL, bypass-assign | full/ALL, bypass-assign | assign+manage/BRANCH | view+update assigned/OWN | create/BRANCH | — | viewFinancial/BRANCH | accept/reject/manage assigned/VENDOR | update assigned/OWN | create (own)/OWN, view own/OWN |
| Field Execution data | view/ALL | view/ALL | view/BRANCH | enter (own jobs)/OWN | — | — | — | view vendor's/VENDOR | enter (own jobs)/OWN | view own service's/OWN |
| Estimates/Invoices/Payments | full/ALL | full/ALL | view+approve/BRANCH | — | — | create estimate/OWN lead | full/BRANCH or ALL per config | view own vendor invoice/VENDOR | — | view+pay own/OWN |
| Notifications & Templates | manage/ALL | manage/ALL | — | — | — | — | — | — | — | receive only |
| Marketing (WhatsApp/Email) | manage/ALL | manage/ALL | — | — | — | send transactional/BRANCH | — | — | — | receive per consent |
| AI Settings | manage/ALL | manage/ALL | — | — | — | — | — | — | — | — |
| Reports & Dashboards | all/ALL | all/ALL | branch-scoped/BRANCH | own/OWN | — | own+branch leads/BRANCH | financial/BRANCH or ALL | vendor-scoped/VENDOR | — | — |
| Audit Logs | view/ALL | view/ALL | view/BRANCH | — | — | — | — | — | — | — |

`OPERATIONS_ADMIN`, `SUB_BRANCH_ADMIN`, `CUSTOMER_SUPPORT_EXECUTIVE`, `HAPPY_CALL_EXECUTIVE`, `ACCOUNTANT`, `OUTSOURCED_PARTNER`, `BUSINESS_CUSTOMER` inherit the nearest row above with narrower scoping (e.g. `SUB_BRANCH_ADMIN` = `BRANCH_MANAGER` row scoped to `SUB_BRANCH` instead of `BRANCH`; `HAPPY_CALL_EXECUTIVE` = `CALL_EXECUTIVE` row restricted to Happy Call sub-type only). The seed script in `manish/03-database-model-implementation-plan.md` enumerates every role's exact row rather than repeating all 22 here.

## 5. Sensitive-Field Gating (beyond module-level permission)

Some fields are hidden per-role even when the parent record is visible:

- **Customer full contact number**: visible to Call Executive, Branch Manager+, Finance (billing), and the assigned Employee/Vendor Technician only once the job is `Accepted` (not before, to reduce leakage during the assignment-broadcast window if that pattern is used — see [06-complete-workflow-document.md](06-complete-workflow-document.md)).
- **Call recordings**: `viewRecordings` permission, default granted to Branch Manager+ and the Happy Call Executive reviewing that call; not granted to Employee/Technician by default.
- **Financial line items (cost/margin) vs. customer-facing price**: Technicians see the customer-facing price only, not internal cost/margin, unless `viewFinancial` is explicitly granted.
- **Vendor payout/commission terms**: visible only to `VENDOR_OWNER` for their own vendor and Finance/Admin — never to a `VENDOR_TECHNICIAN`.

## 6. Assignment/Bypass Authority (cross-reference)

Only `SUPER_ADMIN` and `ADMIN` may assign directly to any level (Branch/Sub-Branch/Team/Employee/Vendor) bypassing the normal hierarchy, per the Business Rules in [01-business-requirements-document.md](01-business-requirements-document.md) §3.3. `BRANCH_MANAGER` may assign within their own Branch to Sub-Branch/Team/Employee/Vendor, but not to another Branch. Enforcement detail and the resulting `assign` permission row is in [06-complete-workflow-document.md](06-complete-workflow-document.md) and [07-status-transition-matrix.md](07-status-transition-matrix.md).

## 7. Customer vs. Business Customer

Both are `role: CUSTOMER` or `BUSINESS_CUSTOMER` in the same `users`/`customers` model family (not separate apps or separate collections) — the distinction is `customerType: INDIVIDUAL | BUSINESS` on the Customer document, driving whether GSTIN/business-name fields are required and whether multiple linked contacts are allowed. See [09-database-architecture.md](09-database-architecture.md).
