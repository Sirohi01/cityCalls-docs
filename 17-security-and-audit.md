# 17 — Security and Audit

## 1. Authentication

- Passwords: bcrypt/argon2 hashed, never logged or returned in any response.
- JWT access token (short-lived, default 15 min) + refresh token (default 7 days, rotated on use, revocable — a `sessions` collection tracks active refresh tokens per device so "log out all devices" is possible).
- Customer app: OTP-based login as an alternative to password (mobile OTP via the SMS/WhatsApp channel), rate-limited (see §4).
- Password reset via time-limited, single-use email link.

## 2. Authorization

- Every endpoint declares its required `{module, action, dataScope}` per [05-user-roles-and-permissions.md](05-user-roles-and-permissions.md); a middleware layer checks role-permission first, then applies the data-scope filter to the underlying query — **scope filtering happens in the query itself**, not as a post-fetch check, so a Branch Manager's list endpoint never even fetches another branch's rows.
- Ownership checks on detail/update endpoints (e.g. a Vendor Technician can only `PATCH` a Service Visit they're assigned to) are explicit, not inferred from the list-filtering logic alone — both layers are enforced independently as defense in depth.

## 3. Secrets Management

- All API keys/credentials (Cloudinary, SMTP, AiSensy, Gemini/OpenAI, JWT signing secret, MongoDB URI) live in environment variables, never committed to the repo (`.env` is gitignored; `.env.example` documents required keys with placeholder values).
- Secrets stored in the database for admin-configured integrations (e.g. a branch-level SMTP password) are encrypted at rest (application-level encryption, not plaintext), and API responses that return integration settings mask these fields (write-only from the client's perspective).

## 4. Rate Limiting

| Endpoint class | Limit (default, configurable) |
|---|---|
| Login | 5 attempts / 15 min per identifier, 20 / 15 min per IP |
| OTP request | 3 / 10 min per mobile number |
| Customer-facing public endpoints (booking, tracking) | 60 / min per IP |
| General authenticated API | 300 / min per user |

Exceeding a limit returns `429` with `Retry-After` per [10-api-standards.md](10-api-standards.md) §12.

## 5. Input Validation & Injection Prevention

- Every request body validated against a Zod schema before reaching business logic (§10 of [10-api-standards.md](10-api-standards.md)).
- MongoDB queries built exclusively through Mongoose's parameterized query builders — no raw string interpolation into queries, which would open NoSQL-injection risk.
- File uploads validated by content-type and size before acceptance (§9 of [10-api-standards.md](10-api-standards.md)); uploaded filenames are never used directly in storage paths (sanitized/UUID-based naming).
- Admin-panel rendering of user-submitted text (notes, remarks) is escaped by React by default; any place that must render rich text uses a sanitizer allowlist, never raw HTML injection.

## 6. Transport & Storage Security

- HTTPS enforced in all non-local environments (TLS termination at the load balancer/reverse proxy).
- MongoDB access restricted by network rules (VPC/firewall) plus authenticated connection string; no public database exposure.
- Signed, short-lived URLs for file access where the underlying storage supports it (Cloudinary signed URLs, or equivalent for fallback storage) rather than permanently public asset URLs, for anything containing customer-identifiable content (issue images, documents).

## 7. Audit Logging

Every sensitive action writes an `activity_logs` entry (schema in [09-database-architecture.md](09-database-architecture.md)) capturing user, role, action, module, record, old/new value, timestamp, IP, device, source app, and reason where applicable. "Sensitive" includes at minimum: any create/update/delete on Customer, Service Request, Vendor, Employee, financial documents, role/permission changes, integration settings changes, and every status transition. Audit logs are **append-only** — no update or delete endpoint exists for `activity_logs` from any role, including Super Admin, to preserve trustworthiness of the trail.

## 8. Data Privacy

- Customer contact details and call recordings are gated per §5 of [05-user-roles-and-permissions.md](05-user-roles-and-permissions.md).
- Marketing communication respects per-channel consent state, itself audit-logged on every change (who changed it, when, from what to what) — not just a boolean flag, per [01-business-requirements-document.md](01-business-requirements-document.md) §3.9.
- AI requests that send customer data to an external provider (Gemini/OpenAI) are logged with what was sent, per [14-integration-architecture.md](14-integration-architecture.md) §5 — this log itself never leaves the system.

## 9. Session & Device Security

- Forced logout on password change (all refresh tokens for that user revoked).
- Admin-initiated "revoke session" action available to Super Admin/Admin for any user, and to a user for their own sessions.
- Vendor/employee mobile app: device binding is not enforced in v1 (a user can log in from a new device without extra verification beyond password/OTP) — flagged in [21-open-decisions-and-clarifications.md](21-open-decisions-and-clarifications.md) if stricter device control is later required.

## 10. Incident/Compliance Note

Given India-based operation and TRAI/DND exposure from calls-heavy operations, the consent-audit-trail requirement in §8 is treated as a compliance control, not just a UX nicety — see the "missing functionality" flag raised in the project's first-output analysis (recorded in [21-open-decisions-and-clarifications.md](21-open-decisions-and-clarifications.md)).
