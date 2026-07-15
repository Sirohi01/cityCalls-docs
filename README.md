# citycalls-docs

Canonical documentation and API contract repository for the CityCalls platform.

This repo contains no application code. It is the single source of truth for:
- Business requirements, workflows, and status/permission rules ([00-project-overview.md](00-project-overview.md) onward)
- The canonical OpenAPI spec (`openapi/`) that every other CityCalls repo syncs a local copy from
- Coordination process between the two developers (`coordination/`)
- Per-developer implementation plans (`manish/`, `rohit/`)

## Related repositories

- `citycalls-api` — Node/Express/TypeScript backend
- `citycalls-admin-web` — Next.js admin panel
- `citycalls-customer-mobile` — Flutter customer app
- `citycalls-vendor-mobile` — Flutter vendor/employee app (independent of `citycalls-customer-mobile`)

See [00-project-overview.md](00-project-overview.md) for the full picture.
