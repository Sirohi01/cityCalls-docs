# Rohit 12 — Responsive Design Guidelines

## 1. Admin-Web Breakpoints

| Breakpoint | Range | Primary layout behavior |
|---|---|---|
| Desktop (default) | ≥1280px | Full sidebar nav, multi-column detail pages, wide tables |
| Tablet/laptop | 768–1279px | Collapsible sidebar (icon-only or overlay), tables scroll horizontally within their container (per artifact/dashboard skill guidance — page itself never scrolls horizontally) |
| Mobile (admin, secondary priority) | <768px | Sidebar becomes a drawer, tables convert to stacked-card view for key columns, forms single-column |

Admin panel is primarily a desktop tool (call executives, managers at a desk) — mobile responsiveness is a "doesn't break" bar, not a primary design target, unlike the two Flutter apps which are mobile-first by definition.

## 2. Flutter Apps — Device Range

Both apps target phones primarily (not tablets as a first-class target for v1); standard Flutter responsive patterns (`MediaQuery`, `LayoutBuilder`) applied for the range of common Android/iOS phone sizes, with tablet layout treated as "doesn't break, not specifically optimized" for v1.

## 3. Table Responsiveness (Admin-Web)

Per §3 of the `DataTable` component ([03-shared-component-inventory.md](03-shared-component-inventory.md)): below the tablet breakpoint, tables show a reduced column set (the 3-4 most important columns per §specifications in [08-table-and-filter-specifications.md](08-table-and-filter-specifications.md)) with a "view details" expansion for the rest, rather than horizontal-scrolling a 10-column table on a small screen.

## 4. Form Responsiveness

Multi-column forms (desktop) collapse to single-column below tablet breakpoint; field grouping/section order stays identical, just re-flowed — never reordered, so the mental model transfers between breakpoints.

## 5. Offline/Sync Indicator Placement (Vendor App)

`SyncStatusIndicator` per [11-loading-error-empty-states.md](11-loading-error-empty-states.md) §5 is persistently visible (e.g. a small status bar under the app header), not something a technician has to dig for while working in the field with one hand and possibly poor lighting/connectivity — a deliberate accessibility/usability call for this app's actual field-use context.

## 6. Touch Targets (both mobile apps)

Minimum 44x44pt touch targets throughout, per standard mobile accessibility guidance — particularly relevant for the vendor app's execution-flow buttons, used in the field, sometimes with gloves or in a hurry.

## 7. Testing Across Breakpoints

Every screen's manual verification (per [23-definition-of-done.md](../23-definition-of-done.md) §1) includes checking at least the desktop and tablet breakpoints for admin-web, and at least two physical device sizes (a smaller and a larger phone) for both Flutter apps — not just the simulator's default size.
