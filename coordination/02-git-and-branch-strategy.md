# Coordination 02 — Git and Branch Strategy

## 1. Repository Model: Multi-Repo

Five independent Git repositories, per [manish/01-project-and-repository-setup.md](../manish/01-project-and-repository-setup.md) §1 — `citycalls-docs`, `citycalls-api`, `citycalls-admin-web`, `citycalls-customer-mobile`, `citycalls-vendor-mobile`. **No monorepo, no shared package repo.** Each repo has its own independent branch history, versioning, and release cadence. A change that spans repos (e.g. a new API field consumed by a new UI field) is **two separate PRs in two separate repos**, coordinated by the process in [08-change-request-process.md](08-change-request-process.md) — never a single cross-repo commit.

## 2. Branch Model (identical shape, applied independently in each repo)

| Branch | Purpose |
|---|---|
| `main` | Production-ready, deployable at every commit, for that repo |
| `develop` | Integration branch for that repo, staging deploys from here |
| `feature/{name}` | Standard feature work within that repo |
| `release/{version}` | Cut from that repo's `develop` for a release |

Prefixed branch names like `manish/backend-{feature}` or `rohit/admin-ui-{feature}` are no longer necessary for *repo* disambiguation (the repo itself is now the disambiguator — `citycalls-api` only ever has Manish's backend work, `citycalls-admin-web` only ever has Rohit's UI work plus Manish's adapter-layer commits). Within a repo, branch naming is simply `feature/{short-description}`.

## 3. Merge Flow (per repo)

Feature branch → PR into that repo's `develop` → CI passes (per [19-testing-strategy.md](../19-testing-strategy.md) §5) → review → merge. `develop` → `release/{version}` → `main` at that repo's own release time.

There is **no cross-repo merge order** in the git sense — coordination between repos happens through the contract-freeze-then-implement sequence in [04-api-handover-process.md](04-api-handover-process.md), not through git operations spanning repos.

## 4. Commit Message Standard (unchanged, applies within each repo)

`{type}({scope}): {summary}`, e.g. `feat(service-requests): add status transition validation` in `citycalls-api`, `fix(booking): correct slot picker validation` in `citycalls-customer-mobile`. Types: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`. Scope matches the module/feature area within that specific repo.

## 5. Pull Request Template (per repo)

Every PR description includes: what changed and why, which `citycalls-docs` file(s) it implements (linked by URL, since docs now live in a separate repo — not a relative path), whether it depends on or was blocked by a contract change in another repo, test evidence, and a checklist against [23-definition-of-done.md](../23-definition-of-done.md) §1.

## 6. Review Process

- `citycalls-api` PRs: Manish self-merges after CI passes (he's the sole owner of this repo, per [03-code-ownership.md](03-code-ownership.md)).
- UI-focused PRs in `citycalls-admin-web`/`citycalls-customer-mobile`/`citycalls-vendor-mobile` (screens/components): Rohit self-merges after CI passes.
- Adapter/data-layer PRs in the three frontend repos (files under Manish's ownership within those repos, per [03-code-ownership.md](03-code-ownership.md)): Manish self-merges after CI passes.
- Any PR to `citycalls-docs` that changes a contract (API shape, enum, naming convention, status/permission rule): **mandatory review by the other developer** before merge — this is the one place self-merge is not allowed, since a docs-repo contract change is what every other repo depends on.

## 7. Cross-Repo Contract Change Flow

1. Contract change PR opened against `citycalls-docs` (per [08-change-request-process.md](08-change-request-process.md)).
2. Reviewed and merged into `citycalls-docs`.
3. Each affected consuming repo (`citycalls-api` and/or the frontend repos) runs its `scripts/sync-contracts.sh` to pull the updated spec and opens its own PR implementing the change — these are independent PRs in independent repos, not required to land simultaneously, but tracked together in the daily progress format ([09-daily-progress-format.md](09-daily-progress-format.md)) until all affected repos have caught up.
4. A repo that hasn't yet synced a contract change is explicitly "behind" — tracked, not silently ignored (see compatibility tracking in [11-release-checklist.md](11-release-checklist.md)).

## 8. Merge Order Within a Phase

Per each phase's "Merge order" note in [22-phase-wise-development-plan.md](../22-phase-wise-development-plan.md): contract lands in `citycalls-docs` → `citycalls-api` implements → frontend repo(s) implement against the now-real endpoint. This is a *sequencing* order across repos, not a git merge order — each step is its own repo's own merge.

## 9. Tags & Versioning

Each repo is tagged independently (`vMAJOR.MINOR.PATCH` per repo) at its own releases — there is no single project-wide version number. Compatibility between repo versions (e.g. "admin-web v1.3.0 requires api v1.5.0+") is tracked per [11-release-checklist.md](11-release-checklist.md), since nothing enforces this automatically without a shared package/lockfile.
