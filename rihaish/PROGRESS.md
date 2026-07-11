# PROGRESS.md — Rihaish

**Current build-step:** 06 — form-input-kit
**Last completed:** 05 — app-shell — DONE (2026-07-11)
**Next:** 06 — form-input-kit

### Recent steps

- **05 — app-shell** — DONE (2026-07-11) — `core.shell` registered (isCore, deps core.auth+core.entitlements). Pure nav registry `lib/nav.ts` (role×entitlement gate `visibleNav`/`groupNav`, guard shared-device hide) unit-tested; 9 shadcn/ui primitives (dropdown-menu, tooltip, dialog, sheet, command/cmdk, skeleton, sonner, scroll-area, separator) RTL-logical; society app shell `app/[locale]/app/layout.tsx` (host→society or 404, `buildShellData` assembles identity+entitlements+gated nav, cookie sidebar state) → client frame `AppShellClient` (Radix DirectionProvider, collapsible sidebar+tooltips, top bar 3 toggles+search+flat-switcher+bell+user-menu, ⌘K palette, mobile drawer, states PageSkeleton/EmptyState/ErrorState/Forbidden, read-only + impersonation banners, sonner Toaster); loading/error boundaries; `lib/server-auth.ts` RSC identity. All gates green (125 unit tests).
- **04 — auth-rbac** — DONE (2026-07-11) — User/Role/UserRole/LoginAlias/Session/LoginAttempt/OtpCode models + migration; pure logic (`rbac.ts` effective=role∩entitlements, `auth-identity.ts` host-boundary, `rate-limit.ts` lockout/backoff, `otp.ts` single-use+TTL, `password.ts` scrypt) all unit-tested; auth service `lib/auth.ts` (scope-independent, ESLint-allowlisted `db.unscoped()`) — login (phone/alias/email + guard PIN), sessions (idle-TTL, suspended→invalidate), effective perms; 3-layer guard `lib/authz.ts` (`requireAuth`/`requireRole`/`requirePermission`/`requireFeature`); residents seam + role seeder; routes login/logout/otp-request(403 seam)/me-flats/me-active-flat. `core.auth` registered. All gates green (117 unit tests).
- **03 — entitlements-engine** — DONE (2026-07-11) — Feature/Plan/SocietyEntitlement/SocietyPlan models + migration; feature registry (`lib/features.ts`, core.foundation/tenancy/entitlements) validated (missing-edge + cycle) at load; pure DAG (`lib/feature-dag.ts`) + resolver math (`lib/entitlements-compute.ts`: plan∪overrides−disabled−expired+core, enable/disable planning, limits); `lib/entitlements.ts` cached resolver + `requireFeature` (403) + enable/disable (atomic, DAG-guarded, cascade, audit) + registry seeder; probe route `/api/entitlements/check`. All gates green (82 unit tests).

> Full history: PROGRESS-HISTORY.md · Build order: ARCHITECTURE.md
