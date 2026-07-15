# 23 — Definition of Done

## 1. Feature-Level DoD (applies to every ticket/task, any phase)

A feature is done when **all** of the following hold:

- [ ] Matches its approved contract in [11-complete-api-contracts.md](11-complete-api-contracts.md)/[12-frontend-data-contracts.md](12-frontend-data-contracts.md)/[rohit/07-form-field-specifications.md](rohit/07-form-field-specifications.md) — no undocumented field or endpoint shape shipped.
- [ ] Server-side validation matches [10-api-standards.md](10-api-standards.md) §10; client-side validation mirrors it (not a substitute).
- [ ] Every status/permission rule from [07-status-transition-matrix.md](07-status-transition-matrix.md)/[05-user-roles-and-permissions.md](05-user-roles-and-permissions.md) relevant to the feature is enforced server-side, tested per §2 of [19-testing-strategy.md](19-testing-strategy.md).
- [ ] Audit logging present for any sensitive action per [17-security-and-audit.md](17-security-and-audit.md) §7.
- [ ] Loading, error, and empty states implemented for every screen (not just the happy path) — see [rohit/11-loading-error-empty-states.md](rohit/11-loading-error-empty-states.md).
- [ ] Responsive on the breakpoints defined in [rohit/12-responsive-design-guidelines.md](rohit/12-responsive-design-guidelines.md).
- [ ] Unit/integration tests per §2 of [19-testing-strategy.md](19-testing-strategy.md) pass in CI.
- [ ] No hardcoded value that the requirements marked as configurable (cross-check against [04-modules-and-feature-list.md](04-modules-and-feature-list.md) and [01-business-requirements-document.md](01-business-requirements-document.md) §3.1).
- [ ] Reviewed by the other developer if it touches a shared contract file (per [coordination/03-code-ownership.md](coordination/03-code-ownership.md)).
- [ ] Manually verified working end-to-end (not just "tests pass") — actually exercised in a running environment.

## 2. Phase-Level DoD

A phase (per [22-phase-wise-development-plan.md](22-phase-wise-development-plan.md)) is done when:

- [ ] Every feature in that phase's scope meets §1.
- [ ] That phase's listed acceptance criteria are individually verified, not assumed.
- [ ] Mock data and live API responses are structurally identical for every endpoint in scope (contract test passes, per [19-testing-strategy.md](19-testing-strategy.md) §1 "Contract" row).
- [ ] Integration testing between Manish's and Rohit's work for that phase's modules is complete with no open contract mismatches.
- [ ] User (Manish, as product owner in the UAT sense) has walked through the phase's critical journeys and signed off.
- [ ] No known P0/P1 bug open in that phase's scope.
- [ ] Documentation updated if implementation revealed a gap or change versus the original doc (contracts are living, not frozen-and-forgotten — see [coordination/08-change-request-process.md](coordination/08-change-request-process.md)).

## 3. "Vague Statement" Ban (project-wide, from the original requirements)

A task description or PR description is not acceptable if it could be summarized as "add X module" or "handle Y" without specifying: what data exists, who can perform which actions, what happens next, which records change, which notifications fire, what history is stored, what failure cases exist. This mirrors the documentation-quality bar the docs themselves were held to in Phase 0 — code and its accompanying description must meet the same specificity standard.

## 4. Project-Level DoD (v1 launch readiness)

- [ ] Every P0 module from [02-product-requirement-document.md](02-product-requirement-document.md) §1 complete per §2 above.
- [ ] Security review passed against [17-security-and-audit.md](17-security-and-audit.md).
- [ ] Backup/restore drill completed successfully.
- [ ] Production environment provisioned and monitored per [20-deployment-and-environments.md](20-deployment-and-environments.md).
- [ ] [24-final-documentation-review-checklist.md](24-final-documentation-review-checklist.md) fully checked off.
