# Coordination 11 — Release Checklist

Run before every `release/{version}` → `main` promotion **in each repo** — since repos release independently, this checklist is run per repo, not once for "the project." A "CityCalls release" in practice means a coordinated set of compatible per-repo releases, tracked via §0 below.

## 0. Compatibility Matrix (multi-repo specific — check first)

Because there's no shared package/lockfile enforcing version compatibility, explicitly record which versions of each repo are meant to work together before releasing any one of them:

| Repo | Version being released | Compatible with |
|---|---|---|
| `citycalls-api` | e.g. `v1.5.0` | — |
| `citycalls-admin-web` | e.g. `v1.3.0` | `citycalls-api >= v1.4.0` |
| `citycalls-customer-mobile` | e.g. `v1.2.0` | `citycalls-api >= v1.5.0` |
| `citycalls-vendor-mobile` | e.g. `v1.1.0` | `citycalls-api >= v1.5.0` |

If a frontend repo is released against an API version it hasn't actually synced its contract copy against (per [manish/01-project-and-repository-setup.md](../manish/01-project-and-repository-setup.md) §3), that's a release blocker, not a detail — verify the `scripts/sync-contracts.sh` last-run state before releasing any frontend repo.

## 1. Per-Repo Checklist

- [ ] All modules targeted for this release pass [10-integration-checklist.md](10-integration-checklist.md), for this repo.
- [ ] Full E2E suite relevant to this repo (§3 of [19-testing-strategy.md](../19-testing-strategy.md)) green.
- [ ] No open P0/P1 bugs against this release's scope in this repo.
- [ ] Security review checklist ([17-security-and-audit.md](../17-security-and-audit.md)) reconfirmed if this release touches auth, permissions, or an integration.
- [ ] (`citycalls-api` only) Database migration/seed scripts (if any new masters, policies, or numbering series are needed) tested against a staging-equivalent dataset.
- [ ] (`citycalls-customer-mobile` / `citycalls-vendor-mobile` only) App build tested on physical devices, not just simulators/emulators, for this release.
- [ ] This repo's local contract copy (`openapi/` or equivalent synced file) matches the version of `citycalls-docs` this release is built against — confirmed, not assumed.
- [ ] `citycalls-docs` updated to reflect anything that changed during implementation (per [08-change-request-process.md](08-change-request-process.md)) — no doc left describing pre-implementation intent that diverged from what shipped.
- [ ] Environment variables/secrets for production confirmed present and correct (§3 of [20-deployment-and-environments.md](../20-deployment-and-environments.md)) — checked against this repo's `.env.example`, not assumed.
- [ ] Rollback plan confirmed for this repo (previous image tag / app version identified and ready, per §9 of [20-deployment-and-environments.md](../20-deployment-and-environments.md)).
- [ ] Release notes drafted for this repo.
- [ ] Both developers explicitly sign off — release is not a unilateral action even though either could technically merge to `main` in a repo they own.

## 2. Release Order (when releasing multiple repos together for one coordinated update)

`citycalls-docs` (if contracts changed) → `citycalls-api` → frontend repos (any order among themselves, since they don't depend on each other) — mirrors the build sequencing in [02-git-and-branch-strategy.md](02-git-and-branch-strategy.md) §8, applied at release time.

## 3. Post-Release (per repo)

- [ ] Monitor error tracking/uptime dashboard for the first hour after deploy (backend/web) or for the first wave of store rollout (mobile).
- [ ] Confirm the health-check endpoint (`citycalls-api`) or a sample of critical journeys (per §3 of [19-testing-strategy.md](../19-testing-strategy.md)) work against production immediately after deploy, not just staging.
- [ ] Tag the release in that repo's git history (`vMAJOR.MINOR.PATCH`), and update the compatibility matrix in §0 for the next release cycle.
