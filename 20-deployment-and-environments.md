# 20 — Deployment and Environments

**Hosting provider is not yet decided** (confirmed open item — see [21-open-decisions-and-clarifications.md](21-open-decisions-and-clarifications.md)). This document is written cloud-agnostically; fill in provider-specific detail (§7) once chosen.

## 1. Environments

| Environment | Purpose | Data |
|---|---|---|
| Local | Individual developer machines | Docker Compose: API + MongoDB + Redis, seeded test data |
| Staging | Pre-production validation, UAT | Sanitized/synthetic data, mirrors production config shape |
| Production | Live system | Real data, restricted access |

## 2. Containerization

The `citycalls-api` repo is Dockerized (`Dockerfile` at repo root, multi-stage build: install → build TS → slim runtime image). Its `docker-compose.yml` brings up API + MongoDB + Redis for local development, matching what any cloud provider's container runtime would run in staging/production. The three frontend repos each have their own build/deploy pipeline independent of this one — no shared Docker Compose file spans repos.

## 3. Configuration Per Environment

Environment variables (never committed) per [17-security-and-audit.md](17-security-and-audit.md) §3: `MONGODB_URI`, `JWT_SECRET`, `JWT_REFRESH_SECRET`, `REDIS_URL`, integration keys (`CLOUDINARY_*`, `SMTP_*`, `AISENSY_API_KEY`, `GEMINI_API_KEY`/`OPENAI_API_KEY`), `NODE_ENV`, `CORS_ALLOWED_ORIGINS`. `.env.example` at repo root documents every required key with a placeholder.

## 4. Database

MongoDB Atlas (managed) is the default assumption for staging/production regardless of final compute provider choice, since it decouples database hosting from the app-hosting decision and gives built-in backup/monitoring. Local development uses a containerized MongoDB instance.

## 5. Backups

Automated daily backups (Atlas built-in, or equivalent if self-hosted), retained per a configurable window (default 30 days), with a documented, periodically-tested restore procedure — not just "backups exist" but a confirmed ability to restore from one.

## 6. Deployment Process (provider-agnostic shape)

CI builds and pushes a versioned container image on merge to `main`/`release/*` branches (per [coordination/02-git-and-branch-strategy.md](coordination/02-git-and-branch-strategy.md)); deployment to staging is automatic on merge to `develop`, production deployment is a manual-trigger step requiring explicit approval — never automatic on every merge to `main`, given this is a live operational business system.

## 7. Provider Decision (to be filled in)

Once decided, this section documents: compute target (e.g. AWS ECS/EC2, Render, DigitalOcean App Platform, Railway), CDN/static hosting for admin-web (e.g. Vercel, or served from the same container), mobile app distribution (Google Play / Apple App Store release process, code signing), domain/DNS, SSL/TLS provisioning, monitoring/alerting stack (e.g. Sentry for error tracking, uptime monitoring), and log aggregation.

## 8. Monitoring & Alerting (provider-agnostic requirements, regardless of §7 choice)

- Error tracking (e.g. Sentry) capturing unhandled exceptions from §4 of [18-error-handling-standards.md](18-error-handling-standards.md), tagged with `requestId`.
- Uptime monitoring on the API's health-check endpoint (`GET /api/v1/health`).
- SLA-breach and integration-failure rates surfaced as operational alerts (not just customer-facing reports), since a spike in `INTEGRATION_DISABLED`/`FAILED` notification states indicates a real ops problem even though it doesn't break the core workflow.

## 9. Rollback

Every deployed image is versioned and retained (last N releases); rollback is redeploying the previous image tag, not a code revert requiring a new build — kept fast because this is a live business system where a bad deploy needs a fast recovery path.
