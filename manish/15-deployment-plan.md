# Manish 15 — Deployment Plan

Execution plan for [20-deployment-and-environments.md](../20-deployment-and-environments.md), owned by Manish.

## 1. Local → Staging → Production Setup Order

1. Local Docker Compose working reliably for all developers (Phase 1 prerequisite, per [manish/01-project-and-repository-setup.md](01-project-and-repository-setup.md)).
2. CI pipeline building and testing on every PR (Phase 1).
3. Staging environment provisioned once a hosting provider is chosen (§7 of [20-deployment-and-environments.md](../20-deployment-and-environments.md)) — target: before Phase 4 (Service Request module) is integration-tested, since staging is where joint Manish/Rohit integration testing per [coordination/10-integration-checklist.md](../coordination/10-integration-checklist.md) actually happens.
4. Automatic staging deploy on merge to `develop` wired up in CI.
5. Production environment provisioned and hardened (§6 of [17-security-and-audit.md](../17-security-and-audit.md)) ahead of Phase 11.
6. Manual-approval production deploy step wired up.

## 2. Mobile App Distribution

Google Play (internal testing track → closed testing → production) and Apple App Store (TestFlight → production) release tracks set up once the vendor/customer apps reach a demoable state (targeted around Phase 5-6) — code signing certificates and store listings are a lead-time item, started early even if the app isn't feature-complete yet, since store review/approval can take days.

## 3. Monitoring Setup

Error tracking (Sentry or equivalent) wired independently into `citycalls-api` and each of the two Flutter repos from Phase 1 onward (not deferred to Phase 11) so errors during development/staging are already visible — extending it to production is a config change (DSN/environment tag) in each repo, not new integration work.

## 4. Backup Verification

A restore drill (§5 of [20-deployment-and-environments.md](../20-deployment-and-environments.md)) is scheduled and run before go-live, and the procedure documented (steps, expected duration, who's responsible) so it's not being figured out for the first time during an actual incident.

## 5. Rollback Readiness

Verified as part of the first production deploy (deploy, confirm health, then explicitly test rolling back to the prior tag in a controlled way before considering deployment "done") rather than assumed to work and only tested during a real emergency.
