# PROGRESS.md — Rihaish

**Current build-step:** 05 — app-shell
**Last completed:** 04 — auth-rbac — DONE (2026-07-11)
**Next:** 05 — app-shell

### Recent steps

- **04 — auth-rbac** — DONE (2026-07-11) — User/Role/UserRole/LoginAlias/Session/LoginAttempt/OtpCode models + migration; pure logic (`rbac.ts` effective=role∩entitlements, `auth-identity.ts` host-boundary, `rate-limit.ts` lockout/backoff, `otp.ts` single-use+TTL, `password.ts` scrypt) all unit-tested; auth service `lib/auth.ts` (scope-independent, ESLint-allowlisted `db.unscoped()`) — login (phone/alias/email + guard PIN), sessions (idle-TTL, suspended→invalidate), effective perms; 3-layer guard `lib/authz.ts` (`requireAuth`/`requireRole`/`requirePermission`/`requireFeature`); residents seam + role seeder; routes login/logout/otp-request(403 seam)/me-flats/me-active-flat. `core.auth` registered. All gates green (117 unit tests).
- **03 — entitlements-engine** — DONE (2026-07-11) — Feature/Plan/SocietyEntitlement/SocietyPlan models + migration; feature registry (`lib/features.ts`, core.foundation/tenancy/entitlements) validated (missing-edge + cycle) at load; pure DAG (`lib/feature-dag.ts`) + resolver math (`lib/entitlements-compute.ts`: plan∪overrides−disabled−expired+core, enable/disable planning, limits); `lib/entitlements.ts` cached resolver + `requireFeature` (403) + enable/disable (atomic, DAG-guarded, cascade, audit) + registry seeder; probe route `/api/entitlements/check`. All gates green (82 unit tests).
- **02 — tenancy-core** — DONE (2026-07-11) — Society/SocietyDomain/AuditLog models + first migration; AsyncLocalStorage scope + Prisma scoping extension (`lib/db.ts`) with auto societyId injection, soft-delete→AuditLog, `db.unscoped()` boundary (ESLint-enforced); host resolution middleware (platform/society 404 isolation); screenshot token (staging+GET); read-only 423 seam; health tenancy self-test. All gates green.

> Full history: PROGRESS-HISTORY.md · Build order: ARCHITECTURE.md
