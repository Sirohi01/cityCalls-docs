# Rohit 01 — UI Project Overview

## 1. Your Scope

You own all UI across three clients, each its own independent Git repository with no shared code between them: `citycalls-admin-web` (Next.js/TypeScript/Tailwind), `citycalls-customer-mobile` (Flutter/Dart), and `citycalls-vendor-mobile` (Flutter/Dart — fully independent of `citycalls-customer-mobile` too, despite both being Flutter). Full ownership boundaries in [coordination/03-code-ownership.md](../coordination/03-code-ownership.md).

## 2. What You Build Against

Never a live API directly and never invented field names — always the frozen contracts in [11-complete-api-contracts.md](../11-complete-api-contracts.md) and [12-frontend-data-contracts.md](../12-frontend-data-contracts.md), consumed through typed hooks/repositories Manish provides (§2-3 of [12-frontend-data-contracts.md](../12-frontend-data-contracts.md)), against mock data initially (per [coordination/05-mock-data-contract.md](../coordination/05-mock-data-contract.md)) and switched to live per [coordination/04-api-handover-process.md](../coordination/04-api-handover-process.md).

## 3. Three Independent Repos, Two Different Stacks

Admin panel is TypeScript/React in its own repo; each mobile app is Flutter/Dart in its own separate repo — a genuinely different language and component model, and no shared package even between the two Flutter apps. Design language stays consistent across all three by following the same specification (see [02-design-system.md](02-design-system.md)), even though implementation doesn't share code — this consistency is a discipline you maintain by building each repo's local tokens/components to the same documented spec, not something enforced by a compiler.

## 4. Build Sequence

Follows [13-ui-development-sequence.md](13-ui-development-sequence.md), generally trailing Manish's API sequence ([manish/07-api-development-sequence.md](../manish/07-api-development-sequence.md)) by however long it takes him to freeze a contract — in practice you should rarely be blocked, since contract-freeze + mock happens before implementation per the handover process.

## 5. What "Done" Means for a Screen

Every screen meets [23-definition-of-done.md](../23-definition-of-done.md) §1: matches its form-field spec exactly, has loading/error/empty states, is responsive, and has been manually exercised against both mock and (once available) live data with no console errors.

## 6. Documents You'll Use Most

[03-shared-component-inventory.md](03-shared-component-inventory.md) (build once, reuse everywhere), [07-form-field-specifications.md](07-form-field-specifications.md) (exact field specs per form), [10-api-contract-usage-guide.md](10-api-contract-usage-guide.md) (how to consume the hooks Manish provides), [14-mock-data-and-api-adapter-plan.md](14-mock-data-and-api-adapter-plan.md) (the mock-to-live switch mechanism).
