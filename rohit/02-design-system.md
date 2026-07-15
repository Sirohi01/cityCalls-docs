# Rohit 02 — Design System

## 1. Visual Direction (per [00-project-overview.md](../00-project-overview.md) tech stack)

Light theme, compact professional layout, responsive. Not a consumer-flashy design — this is an operations tool used all day by call executives, branch managers, and technicians; density and scan-ability matter more than visual flourish.

## 2. Design Tokens (this specification is the shared reference — each of the three frontend repos maintains its own local copy, since no shared package exists per [manish/01-project-and-repository-setup.md](../manish/01-project-and-repository-setup.md) §2)

| Token category | Notes |
|---|---|
| Color | Primary/secondary/neutral scales, semantic colors (success/warning/error/info) mapped to status groups — e.g. all "positive terminal" statuses (`PAID`, `CLOSED`) share a success color family, all "attention needed" statuses (`ESCALATED`, `ON_HOLD`, SLA-breached) share a warning/error family. Exact hex values TBD when Rohit begins visual design — this document defines the *system*, not final swatches. |
| Typography | One primary font family, a small type scale (e.g. 6-7 sizes), consistent line-height rules for dense tables vs. spacious detail pages |
| Spacing | 4px/8px base unit scale, consistent across admin-web (Tailwind config) and Flutter (matching `EdgeInsets`/sizing constants) |
| Radius/Elevation | Consistent corner-radius and shadow scale for cards/modals/dropdowns |
| Status Colors | One canonical color per status *category* (not per individual enum value) — new, active, waiting-on-customer, in-progress, blocked, success, escalated/failed — mapped from the full status list in [07-status-transition-matrix.md](../07-status-transition-matrix.md) so adding a new status doesn't require a new color decision |

## 3. Enum → Label/Color Mapping

Per §7 of [12-frontend-data-contracts.md](../12-frontend-data-contracts.md), each frontend repo's local `tokens/` implements the `{enumValue: {label, colorToken}}` map for every enum in [coordination/06-naming-conventions.md](../coordination/06-naming-conventions.md) §4, built to this document's spec — this is the one *documented* place status/enum presentation is decided, even though it's implemented three times (once per repo) rather than imported from one shared location.

## 4. Iconography

One consistent icon set (e.g. Lucide/Phosphor for web, matching or equivalent Material/Cupertino-adjacent set for Flutter) — chosen once, used everywhere, not mixed per screen.

## 5. Dark Mode

Not in scope for v1 (light theme only, per [00-project-overview.md](../00-project-overview.md) tech stack) — tokens are still structured so a dark variant could be added later without a redesign, but it's not built now.

## 6. Cross-Client Consistency Rule

A status badge, a priority indicator, or a role label looks and reads identically whether shown in admin-web, the customer app, or the vendor app — achieved by both platforms consuming the same token/label source (§3), not by independent per-platform styling decisions.
