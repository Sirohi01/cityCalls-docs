# Coordination 03 — Code Ownership

## 1. Ownership Map (multi-repo)

| Repo | Path within repo | Owner | Notes |
|---|---|---|---|
| `citycalls-api` | entire repo | Manish | Full backend, no split |
| `citycalls-admin-web` | `app/**`, `components/**` (screens/UI) | Rohit | UI only |
| `citycalls-admin-web` | `lib/api/**`, `lib/hooks/**`, `lib/types/**`, `lib/validation/**` | Manish | Adapter/hook layer + locally-generated types, per [00-project-overview.md](../00-project-overview.md) §6 |
| `citycalls-admin-web` | `tokens/**` | Rohit | Design tokens, enum-label maps — maintained locally in this repo (no shared `ui-tokens` package exists) |
| `citycalls-customer-mobile` | `lib/screens/**`, `lib/widgets/**` | Rohit | UI only |
| `citycalls-customer-mobile` | `lib/data/**`, `lib/models/**` | Manish | Functional/data layer, API client, locally-generated Dart models |
| `citycalls-customer-mobile` | `lib/tokens/**` | Rohit | This app's own copy of design tokens |
| `citycalls-vendor-mobile` | same split as `citycalls-customer-mobile` | — | **Fully independent repo** — no code, tokens, or models shared with `citycalls-customer-mobile` even though both are Flutter apps built by the same two developers |
| `citycalls-docs` | `docs/**` (shared, `00`-`24`) | Joint | Any doc affecting both requires the other's awareness before merge (per [02-git-and-branch-strategy.md](02-git-and-branch-strategy.md) §6) |
| `citycalls-docs` | `docs/manish/**` | Manish | |
| `citycalls-docs` | `docs/rohit/**` | Rohit | |
| `citycalls-docs` | `docs/coordination/**` | Joint | |
| `citycalls-docs` | `docs/openapi/**` (canonical spec) | Manish authors; both consume | Changes still go through the shared-file review flow in §7 below since every repo depends on it |
| each repo | `infrastructure/**`, `scripts/**`, CI config | Manish | Rohit may add frontend-specific scripts (e.g. local codegen trigger) within the frontend repos he otherwise doesn't own |

## 2. No Genuinely Shared Directory Anymore

Previously, one shared `packages/shared-types` directory required joint stewardship. That directory no longer exists (confirmed decision, [manish/01-project-and-repository-setup.md](../manish/01-project-and-repository-setup.md) §2). The **canonical OpenAPI spec in `citycalls-docs`** now plays that role instead — it's the one artifact every repo depends on, even though it isn't code. Changes to it follow the same "joint review required" discipline that `packages/shared-types` used to require (§7 of [02-git-and-branch-strategy.md](02-git-and-branch-strategy.md)).

## 3. Rule

**No developer edits a file outside their ownership column above without first communicating with the owner**, even for a trivial-seeming fix — small unilateral edits in someone else's owned code (now scoped per-repo as well as per-path) are exactly the kind of quiet conflict source this document exists to prevent. Flag it to the owner instead; they make the fix or explicitly hand it over.

## 4. Consequence of No Shared Packages: Duplication Is Expected and Accepted

Design tokens, enum-label maps, and generated types now exist in **up to four independent copies** (admin-web, customer-mobile, vendor-mobile, plus the API's own Zod schemas) rather than one shared source. This is an accepted cost of the multi-repo/no-shared-package decision (see [21-open-decisions-and-clarifications.md](../21-open-decisions-and-clarifications.md) §1) — keeping these copies aligned is a **process responsibility** (contract-sync steps in [manish/01-project-and-repository-setup.md](../manish/01-project-and-repository-setup.md) §3, integration checklist in [10-integration-checklist.md](10-integration-checklist.md)), not something to solve by quietly reintroducing shared code.

## 5. Escalation

If ownership of a specific file/directory within a repo is genuinely ambiguous, default to whichever developer's *primary skillset* the file most resembles until it's added explicitly to this document — then add it.
