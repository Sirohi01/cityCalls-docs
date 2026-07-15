# Coordination 01 — Team Working Agreement

## 1. Roles Recap

Manish: backend, API, DB, integrations, security, deployment, both mobile apps' functional/data layer, final merge. Rohit: all UI across admin-web and both Flutter apps. Full split in [00-project-overview.md](../00-project-overview.md) §6.

## 2. Working Rhythm

- Both developers work from the approved `docs/` set as the single source of truth — a disagreement about "what should happen" is resolved by checking/updating the doc, not by whichever code shipped first.
- Progress updates follow the format in [09-daily-progress-format.md](09-daily-progress-format.md).
- Neither developer blocks silently — if Rohit is blocked on a missing contract, or Manish is blocked on a missing UI spec, that's flagged the same day, not discovered at integration time.

## 3. Communication on Contract Changes

Any change to a shared contract (API shape, field name, enum value, DB field) follows [08-change-request-process.md](08-change-request-process.md) — no unilateral changes to files either developer doesn't own, per [03-code-ownership.md](03-code-ownership.md).

## 4. Decision Authority

- Product/business-rule questions → the user (Manish, as product owner) decides, recorded in [21-open-decisions-and-clarifications.md](../21-open-decisions-and-clarifications.md).
- Backend technical decisions (library choice, schema design within approved architecture) → Manish decides.
- Frontend technical decisions (component structure, state management within approved architecture) → Rohit decides.
- Shared-contract decisions (field names, API shapes) → joint, documented in the relevant shared doc before implementation.

## 5. Definition of "Ready to Build"

Neither developer starts implementing a module until: its relevant shared docs exist (workflow, status, roles, API contract, or UI spec as applicable) and — for UI work specifically — mock data matching the contract is available, per [05-mock-data-contract.md](05-mock-data-contract.md).

## 6. Conflict Resolution

Technical disagreements within one's own domain (Manish on backend, Rohit on UI) are that developer's call. Disagreements crossing the contract boundary are resolved by re-reading the relevant `docs/` file together; if the doc is genuinely ambiguous, it gets clarified there first (with the user's input if it's a product decision) rather than resolved ad hoc in code.

## 7. Availability & Handoff

Not currently specified (two-person team, informal availability) — add explicit working hours / response-time expectations here if the team scales or timezone differences emerge; flagged as an open item if needed later.
