# Manish 04 — Authentication and RBAC Plan

## 1. Auth Implementation

- `POST /auth/login`: verify password (bcrypt compare), issue access + refresh JWT pair, create a `sessions` document (device/IP recorded).
- `POST /auth/refresh`: verify refresh token against `sessions`, rotate it (invalidate old, issue new), issue new access token.
- `POST /auth/logout`: invalidate the session's refresh token.
- `POST /auth/otp/request` / `verify`: for customer app — generate OTP (6-digit, expires in 5 min, rate-limited per [17-security-and-audit.md](../17-security-and-audit.md) §4), verify and issue tokens same as login.
- Password reset: token-based link (JWT with short expiry, single-purpose claim), emailed via the Notification Engine.

## 2. Middleware Chain

`authMiddleware` (verifies JWT, attaches `req.user`) → `permissionMiddleware(module, action)` (checks `role_permissions` for `req.user.role`, attaches the resolved `dataScope` to `req.scope`) → controller (uses `req.scope` to build the Mongo query filter, per §2 of [17-security-and-audit.md](../17-security-and-audit.md)).

## 3. Data-Scope Query Filter Helper

```ts
// lib/scopeFilter.ts
function applyScopeFilter(query, scope, user) {
  switch (scope) {
    case 'OWN': return query.where({ createdBy: user.id }); // or assigneeId, per-entity
    case 'TEAM': return query.where({ teamId: user.teamId });
    case 'SUB_BRANCH': return query.where({ subBranchId: user.subBranchId });
    case 'BRANCH': return query.where({ branchId: user.branchId });
    case 'VENDOR': return query.where({ vendorId: user.vendorId });
    case 'ALL': return query;
  }
}
```
Every module's list/detail service function calls this helper — implemented once in `lib/`, not reimplemented per module, so the enforcement in §2 of [17-security-and-audit.md](../17-security-and-audit.md) is structurally guaranteed rather than relying on every controller remembering to filter.

## 4. Role Seed & Permission Matrix Loading

`role_permissions` seeded per §3 of [manish/03-database-model-implementation-plan.md](03-database-model-implementation-plan.md); loaded into an in-memory cache on API boot (refreshed on `PATCH /config/role-permissions`) so the permission check on every request is a cache lookup, not a DB query per request.

## 5. Multi-Role Note

Per [05-user-roles-and-permissions.md](../05-user-roles-and-permissions.md) §1, one role per `User` for v1 — `req.user.role` is a single value, simplifying the middleware significantly versus a multi-role union.

## 6. Testing

Auth/RBAC test suite (per [19-testing-strategy.md](../19-testing-strategy.md) §2) includes: a matrix test asserting every role can/cannot perform every `{module, action}` combination from [05-user-roles-and-permissions.md](../05-user-roles-and-permissions.md), and a data-scope test per role confirming out-of-scope records are invisible (404, not 403, per [18-error-handling-standards.md](../18-error-handling-standards.md) §5).
