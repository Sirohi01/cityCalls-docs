# Manish 13 — Security and Logging Plan

Implementation plan for [17-security-and-audit.md](../17-security-and-audit.md).

## 1. Implementation Checklist

- [ ] `bcrypt`/`argon2` password hashing wired in `auth.service.ts`, never logging raw passwords (including in error stack traces — scrub request body before logging).
- [ ] JWT signing with separate access/refresh secrets, short/long expiry per §1 of [17-security-and-audit.md](../17-security-and-audit.md).
- [ ] `sessions` collection + revocation logic.
- [ ] Rate-limiting middleware (`express-rate-limit` or equivalent) applied per the table in §4 of [17-security-and-audit.md](../17-security-and-audit.md).
- [ ] Zod validation on every mutating endpoint, no exceptions.
- [ ] Mongoose-only query construction (no raw `$where` or string-built queries) — a lint rule or code-review checklist item, since Mongo has no automated equivalent of SQL-injection static analysis in v1.
- [ ] File upload validation (type/size) enforced server-side even when using the signed-upload flow (Cloudinary upload preset restrictions + a server-side confirm-step check).
- [ ] HTTPS enforced via reverse proxy config in every non-local environment.
- [ ] Secrets exclusively in env vars; `.env` gitignored; secret-scanning pre-commit hook considered (flagged, not mandatory for v1 two-person team).

## 2. Audit Logging Implementation

`lib/auditLog.ts` exposes `logActivity({entityType, entityId, userId, userRole, action, module, oldValue, newValue, reason})` — called from `updateStatus()` (§6 of [manish/06-workflow-engine-plan.md](06-workflow-engine-plan.md)) and from every module's create/update/delete service functions for the sensitive-action list in §7 of [17-security-and-audit.md](../17-security-and-audit.md). IP/device/source-app captured from request middleware and passed through, not re-derived at the log call site.

## 3. Consent Audit Trail

`customers.consent.{channel}` changes write a dedicated `consent_history` sub-array or a filtered `activity_logs` query (`entityType: CONSENT`) — decided during implementation, but either way queryable independently as "full consent change history per customer/channel," per the compliance note in §10 of [17-security-and-audit.md](../17-security-and-audit.md).

## 4. Logging Infrastructure

Structured JSON logging (e.g. `pino`) with `requestId` correlation (§8 of [18-error-handling-standards.md](../18-error-handling-standards.md)), shipped to whatever log aggregation is chosen in [20-deployment-and-environments.md](../20-deployment-and-environments.md) §8. Sensitive fields (passwords, tokens, full card numbers if ever handled) are redacted at the logger config level, not left to per-call-site discipline.

## 5. Security Review Cadence

A self-review against [17-security-and-audit.md](../17-security-and-audit.md) runs before every release per [coordination/11-release-checklist.md](../coordination/11-release-checklist.md); a fuller review (potentially third-party) is recommended before the v1 public launch, flagged in [22-phase-wise-development-plan.md](../22-phase-wise-development-plan.md) Phase 11.
