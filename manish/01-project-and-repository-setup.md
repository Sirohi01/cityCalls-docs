# Manish 01 — Project and Repository Setup

## 1. Multi-Repo Initialization (no shared packages)

Per the confirmed decision in [21-open-decisions-and-clarifications.md](../21-open-decisions-and-clarifications.md) §1: CityCalls is **five independent Git repositories**, not a monorepo. Each is fully self-contained — its own dependencies, its own local copies of types/constants/validation schemas, its own CI pipeline, its own release cadence. Nothing is imported across repos as a package.

| Repo | Contents | Owner |
|---|---|---|
| `citycalls-docs` | This entire `docs/` tree — the canonical source of truth for the OpenAPI spec, naming conventions, workflow, and every contract | Joint (per [coordination/03-code-ownership.md](../coordination/03-code-ownership.md)) |
| `citycalls-api` | Node/Express/TS backend | Manish |
| `citycalls-admin-web` | Next.js/TS admin panel | Rohit (UI) + Manish (adapter/hook layer, per ownership split) |
| `citycalls-customer-mobile` | Flutter customer app | Rohit (UI) + Manish (functional/data layer) |
| `citycalls-vendor-mobile` | Flutter vendor/employee app — **fully independent of `citycalls-customer-mobile`**, no shared Flutter package between the two mobile apps either | Rohit (UI) + Manish (functional/data layer) |

Each repo's internal folder structure:

```
citycalls-api/
├── src/
│   ├── modules/            # per 08-system-architecture.md §2
│   ├── middleware/
│   ├── lib/
│   ├── realtime/
│   ├── jobs/
│   └── config/
├── openapi/                # local copy of the contract spec, synced from citycalls-docs — see §3
├── scripts/
├── docker-compose.yml
└── .env.example

citycalls-admin-web/
├── app/                    # Next.js App Router
├── components/
├── lib/
│   ├── api/                 # hand-written/generated API client, local to this repo
│   ├── hooks/
│   ├── types/                # local TS types, generated from the synced OpenAPI copy
│   └── validation/           # local Zod schemas
├── mocks/                    # local mock JSON, per coordination/05-mock-data-contract.md
└── tokens/                   # local design tokens, per rohit/02-design-system.md

citycalls-customer-mobile/ and citycalls-vendor-mobile/ (each independently)
├── lib/
│   ├── screens/
│   ├── widgets/
│   ├── data/                 # repositories, API client
│   ├── models/                # local Dart models, generated from the synced OpenAPI copy
│   └── tokens/                # local design tokens (each app maintains its own copy)
└── mocks/
```

## 2. Why No Shared Packages (explicit trade-off, recorded so it isn't second-guessed later)

The project deliberately gives up automatic type/constant sharing in exchange for full independence: any repo can be built, tested, deployed, and released without touching another repo's code or dependency graph. The cost is that consistency (field names, enum values, design tokens) is maintained **by process and documentation**, not by the compiler — every repo's local types/constants must be kept in sync with `citycalls-docs` manually, per the sync steps in §3 and [coordination/04-api-handover-process.md](../coordination/04-api-handover-process.md). This is a conscious choice, not an oversight — do not "fix" it later by reintroducing a shared package without re-confirming with the user.

## 3. Contract Sync Mechanism (replaces the old shared-types package)

`citycalls-docs` holds the canonical OpenAPI spec (`docs/openapi/citycalls.yaml`, referenced throughout [11-complete-api-contracts.md](../11-complete-api-contracts.md)). Every other repo pulls a **local copy** of this file via a small sync script (`scripts/sync-contracts.sh` in each consuming repo — fetches the file from the `citycalls-docs` repo, either via git submodule-free raw fetch, a published release artifact, or a manual copy step, decided during Phase 1 setup) and runs its own local codegen against that copy:
- `citycalls-api`: validates its Zod schemas against the synced spec (source of truth for what it *implements*, since Manish authors changes here first).
- `citycalls-admin-web`: generates local TS types + Zod schemas from the synced copy.
- `citycalls-customer-mobile` / `citycalls-vendor-mobile`: each independently generates its own local Dart models from the synced copy.

A contract change always lands in `citycalls-docs` first (per [coordination/08-change-request-process.md](../coordination/08-change-request-process.md)), then each consuming repo re-syncs and regenerates on its own schedule — this is an explicitly async propagation, not an atomic cross-repo update, so a brief window where repos are on different contract versions is expected and tracked via the compatibility notes in [coordination/11-release-checklist.md](../coordination/11-release-checklist.md).

## 4. Initial Setup Steps (per repo)

1. `git init` each of the five repos separately, each with its own `.gitignore`, `README.md`.
2. `citycalls-api`: `npm init`, TypeScript strict config, Express skeleton, `src/modules/` per [08-system-architecture.md](../08-system-architecture.md) §2, `docker-compose.yml` (API + MongoDB + Redis) per [20-deployment-and-environments.md](../20-deployment-and-environments.md) §2.
3. `citycalls-admin-web`: `create-next-app` (TypeScript + Tailwind, App Router).
4. `citycalls-customer-mobile` and `citycalls-vendor-mobile`: `flutter create` each independently, iOS + Android targets.
5. Every repo: `.env.example` documenting its own required variables per [17-security-and-audit.md](../17-security-and-audit.md) §3, its own CI pipeline (lint, type-check, test), its own `scripts/sync-contracts.sh`.
6. `citycalls-docs`: no app code — just `docs/`, plus `openapi/` holding the canonical spec.

## 5. Branch Protection

Each repo protects its own `main` and `develop` independently — no direct push, PR + CI-pass required, per [coordination/02-git-and-branch-strategy.md](../coordination/02-git-and-branch-strategy.md) (rewritten for multi-repo).

## 6. First Commit Checklist (per repo)

- [ ] Structure matches §1 for that repo.
- [ ] `.gitignore` correct for that repo's stack.
- [ ] `.env.example` present, `.env` gitignored.
- [ ] `scripts/sync-contracts.sh` present and tested against `citycalls-docs`.
- [ ] CI pipeline runs on first PR.
- [ ] README documents how to run this repo standalone, including which `citycalls-docs` contract version it was last synced against.

## 7. Dependency on Documentation

Setup does not begin until [00-project-overview.md](../00-project-overview.md) through [10-api-standards.md](../10-api-standards.md) are approved, since folder structure in §1 derives from [08-system-architecture.md](../08-system-architecture.md) §2 and naming from [coordination/06-naming-conventions.md](../coordination/06-naming-conventions.md).
