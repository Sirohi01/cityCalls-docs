# Coordination 07 — Merge and Conflict Prevention

## 1. Why Conflicts Should Be Rare by Design

Given both the ownership split in [03-code-ownership.md](03-code-ownership.md) and the multi-repo structure itself, Manish and Rohit's day-to-day work touches almost entirely disjoint files — often in entirely separate repos. Git merge conflicts in the traditional sense (two people editing the same file in the same repo) should be rare; the more likely issue in a multi-repo setup is **cross-repo drift**, not a git conflict — this document covers both.

## 2. Common Conflict/Drift Sources & Prevention

| Source | Prevention |
|---|---|
| The OpenAPI spec in `citycalls-docs` changed by one developer without the other reviewing | Mandatory joint review on any spec change (§6 of [02-git-and-branch-strategy.md](02-git-and-branch-strategy.md) §6, since it's the closest thing to the old shared-types package) |
| A frontend repo's local mock/type copy silently drifting from the canonical spec after a contract change | Each repo's own contract-schema CI check (§6 of [05-mock-data-contract.md](05-mock-data-contract.md)) catches this — but only if that repo's sync script is actually re-run; don't assume it happened, check the daily progress tracking |
| Two of the three frontend repos (e.g. both mobile apps) independently implementing the same enum-label mapping slightly differently, since there's no shared `ui-tokens` package | Both reference the same source of truth — [rohit/02-design-system.md](../rohit/02-design-system.md) §3 — when building each repo's local token file; a visual QA pass comparing the two apps side-by-side catches drift a compiler can't |
| Both editing the same `docs/` file within `citycalls-docs` | Coordinate before large doc edits; small clarifying edits are low-risk, structural rewrites should be flagged in advance |
| Route/adapter files (`lib/api/**`) within a frontend repo touched by Rohit to "just add a quick field" | Not allowed — request the change from Manish instead (§4 of [01-team-working-agreement.md](01-team-working-agreement.md)) |
| Both branching from a stale `develop` within the same repo | Rebase/merge `develop` into a feature branch before opening a PR, not just before merging |
| Generated files (Dart models, TS types) manually edited by either developer to "make something work quickly" | Never — regenerate from the locally-synced OpenAPI spec copy instead; a manual edit will be silently overwritten (or silently diverge and never be overwritten, which is worse) by the next codegen run |

## 3. Pre-PR Checklist

- [ ] Branch is rebased on latest `develop` (within that repo).
- [ ] No edits outside your ownership map without prior communication.
- [ ] If a `citycalls-docs` contract file was touched, the other developer is tagged for review per [02-git-and-branch-strategy.md](02-git-and-branch-strategy.md) §6.
- [ ] If this PR depends on a `citycalls-docs` change, confirm that change is already merged there first — don't implement against an unmerged contract draft.
- [ ] CI passes locally before pushing (type-check, lint, tests) — don't rely on CI to discover a broken build.

## 4. Resolving an Actual Conflict

Standard git conflict resolution within a single repo; for conflicts inside generated files, resolve by regenerating from the locally-synced OpenAPI spec copy rather than manually merging generated output — manually resolved generated-file conflicts are a recurring source of silent drift. Since repos are independent, a conflict in `citycalls-admin-web` never involves `citycalls-api` or the mobile repos directly — if resolving it seems to require knowledge from another repo, that's a sign the underlying contract wasn't actually frozen clearly enough in `citycalls-docs`.

## 5. Large Refactors

A refactor touching many files across ownership boundaries (e.g. a naming-convention correction that ripples through backend and all three frontends) is planned and communicated before starting: the `citycalls-docs` change lands first and is reviewed jointly, then each affected repo gets its own dedicated branch and PR implementing the change against the new contract — these are still separate PRs per repo (multi-repo has no single "coordinated PR" spanning repos), but they're planned and executed together rather than one repo drifting ahead of the others for an extended period.
