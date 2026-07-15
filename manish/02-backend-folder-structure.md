# Manish 02 — Backend Folder Structure

## 1. `src/` Layout (repo root of the standalone `citycalls-api` repo)

```
src/
├── app.ts                     # Express app assembly
├── server.ts                  # HTTP + Socket.IO server bootstrap
├── config/                    # env loading, constants
├── middleware/
│   ├── auth.middleware.ts     # JWT verification
│   ├── permission.middleware.ts # role + data-scope check (05-user-roles-and-permissions.md)
│   ├── validate.middleware.ts # Zod schema validation
│   ├── error.middleware.ts    # global error handler (18-error-handling-standards.md)
│   └── rateLimit.middleware.ts
├── modules/
│   ├── auth/
│   ├── users/
│   ├── organization/           # branch, sub-branch, team
│   ├── employees/
│   ├── vendors/
│   ├── customers/
│   ├── catalog/                # services, brands, product-types
│   ├── config/                 # masters, policies, numbering
│   ├── calls/
│   ├── leads/
│   ├── service-requests/
│   ├── field-execution/        # service-visits
│   ├── reopen/
│   ├── finance/                # estimates, invoices, payments
│   ├── notifications/
│   ├── marketing/
│   ├── ai/
│   ├── files/
│   ├── reports/
│   ├── audit/
│   └── import-export/
├── realtime/                   # Socket.IO setup, room/event definitions
├── jobs/                       # background job definitions + queue setup
├── lib/                        # shared internal utilities (numbering engine, policy resolver)
└── types/                      # internal-only types, local to this repo (no shared-types package exists)
```

## 2. Per-Module File Convention

Each `modules/{name}/` folder: `{name}.model.ts` (Mongoose schema), `{name}.controller.ts` (route handlers), `{name}.service.ts` (business logic, the only layer other modules call into), `{name}.routes.ts` (Express router), `{name}.validation.ts` (Zod schemas), `{name}.types.ts` (module-internal types).

## 3. Cross-Module Access Rule

A module imports another module's `.service.ts` exports only — never another module's `.model.ts` directly. This is the enforced boundary from [08-system-architecture.md](../08-system-architecture.md) §2, checked in code review (no automated lint rule for this in v1, flagged as a possible future addition).

## 4. `lib/` Engines

`lib/numbering.ts` (§4 of [09-database-architecture.md](../09-database-architecture.md)), `lib/policyResolver.ts` (§5 of [09-database-architecture.md](../09-database-architecture.md)), `lib/statusEngine.ts` (validates transitions against [07-status-transition-matrix.md](../07-status-transition-matrix.md) seed data) — these are the shared engines every domain module calls, kept outside `modules/` since they aren't domain-specific.

## 5. Test File Placement

Colocated: `{name}.service.test.ts` alongside `{name}.service.ts` for unit tests; `test/integration/` at the repo root for endpoint-level integration tests (one file per module), per [19-testing-strategy.md](../19-testing-strategy.md).
