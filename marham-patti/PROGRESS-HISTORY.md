> ⚠️ **PUBLIC FILE** — pushed to the public progress repo (`projects-abovenext/marham-patti-progress`). **NO secrets, hostnames, real subdomains, API endpoints, credentials, or keys — ever.** Placeholders only.

# PROGRESS-HISTORY.md — Marham Patti (append-only archive)

> **This is the COMPLETE, append-only archive** of every finished build-step, newest at the top. The agent appends the FULL entry here as part of finishing each step (see PROGRESS.md recording protocol). Read this file **only when recovering an older step's detail** — never cover-to-cover. The SHORT rolling tracker is **PROGRESS.md**.
>
> **Entry format (per step):** `## Build-step NN — <title>` then: work-type (FEATURE/FIX), phase label, branch, dependency note, schema/migration/RLS/auth/permission/**feature-flag** impact, problem, what changed (data model, endpoints, UI Simple/Pro, **flags exposed**, i18n keys), and ALL verification-gate numbers (typecheck, lint, turbo build, jest `N/N (S suites; +X)`, i18n parity, isolation, **flag/vertical smoke**, **performance/Lighthouse**, white-label).
>
> **Numbering:** canonical per ARCHITECTURE.md §14 (continuous 1→N; phase = label).

---

## Build-step 04 — Foundation: RBAC + Permission Catalog

- **Work-type:** FEATURE (authorization layer; always-on — the layer the flags sit atop). **Phase:** Foundation (item 4 of 39). **Branch:** `feat/04-rbac-permissions` (stacked on `feat/03-auth`; controller merges to `staging` at checkpoint).
- **Depends on:** 02, 03. **Date:** 2026-07-03.
- **Schema/migration:** NEW migration `20260703110000_rbac_permissions`. Models: `Permission` (`key @id`, `scope PermScope`, `createdAt`; `@@map("permissions")` — GLOBAL, RLS-OFF), `AppRole` (`id`, `tenantId? @map(tenant_id)`, `name`, `isProtected @map`, `isSystem @map`, `createdAt`; `@@index([tenantId])`; `@@map("app_roles")`), `RolePermission` (`roleId @map(role_id)`, `permissionKey @map(permission_key)`; `@@id([roleId,permissionKey])`, `@@index([roleId])`, `@@index([permissionKey])`; FKs to AppRole+Permission ON DELETE CASCADE; `@@map("role_permissions")`). Enum `PermScope {TENANT, NON_TENANT}`. Partial unique indexes: `app_roles_global_name_key (name) WHERE tenant_id IS NULL`, `app_roles_tenant_name_key (tenant_id,name) WHERE tenant_id IS NOT NULL`.
- **RLS (bespoke, not `apply_tenant_rls`):** `permissions` = RLS-OFF global catalog (like `tenants`). `app_roles` ENABLE+FORCE RLS, custom `app_roles_isolation`: USING = own-tenant rows **OR** global (`tenant_id IS NULL`) rows; WITH CHECK = own-tenant when a context is set, **or** global rows ONLY when no context (the boot reconcile, on the bare client). `role_permissions` ENABLE+FORCE RLS, `role_permissions_isolation`: inherits parent-role visibility via an `EXISTS(app_roles …)` subquery. **Critical detail:** custom (placeholder) GUCs read back as **empty string `''`** (not NULL) once SET LOCAL in a session, so every "no context" guard uses `NULLIF(current_setting('app.tenant_id',true),'') IS NULL`. (First pglite run caught the global-write RLS violation this fixed.)
- **Feature-flags:** ➖ (this IS the auth layer the flags sit atop; §3.4 documents flag-FIRST-then-permission precedence — the flag hook lands in 05).
- **Do-NOT-break honored:** auth principal (03) unchanged; `apply_tenant_rls` (02) untouched; catalog is ADDITIVE-ONLY (sync never deletes strays); boot hook is crash-proof.

### Problem
Coarse `Role` (03) is insufficient — every module needs fine-grained, string-keyed, server-enforced + UI-reflected permissions; the catalog grows as modules land (must sync every boot); SaaS needs custom tenant roles.

### What changed
- **`@mp/shared` (`permissions.ts`):** `PERMISSIONS` catalog (33 keys across patients/appointments/clinical/lab/pharmacy/billing/accounts/inventory/reporting/AI/brand + NON_TENANT `flags.manage`/`tenants.manage`/`platform.admin`), each tagged `PermScope`. Helpers `PERMISSION_KEYS`, `getPermission`, `permissionKeysByScope`, `assignablePermissionKeys(isTenantRole)`, and `ROLE_PERMISSION_DEFAULTS` (documented TENANT defaults per built-in role; SUPER_ADMIN intentionally absent → reconciled to the FULL set). Dependency-light (no Prisma) so both API + web import it.
- **API `permissions/` module:** repos (`PermissionRepo`, `AppRoleRepo` + Prisma impls — global roles on the bare client with no context, custom roles inside `runWithTenant`); `PermissionService` (`onBoot`→`syncPermissionCatalog` upsert-add/update + `reconcileSystemRoles`: one GLOBAL system role per `Role`, SUPER_ADMIN = full assignable set incl. NON_TENANT + later-added keys, every other role = TENANT-only defaults with NON_TENANT defensively filtered; additive, idempotent, per-role try/catch so a malformed row never aborts the rest; per-role effective-key cache cleared on mutation); role-manager ops (`listRoles`/`createCustomRole`/`setRoleKeys` — rejects unknown/NON_TENANT keys + protected/system edits); `RolesGuard` (global APP_GUARD, opt-in: `@Roles(...)` + `@RequirePermission('key')`, 403 on miss, skips `@Public`); `@RequirePermission`/`@Roles` decorators; `RbacController` (`GET rbac/me` effective keys, `GET rbac/permissions` catalog, admin-only `GET/POST rbac/roles`, `PUT rbac/roles/:id/permissions`). Wired into `AppModule` AFTER `AuthModule` so `RolesGuard` runs after `JwtAuthGuard`; `onApplicationBootstrap` runs the reconcile (crash-proof).
- **Web (`/admin/roles`):** Permission UI base — lean client island lists roles + catalog, toggles TENANT keys on CUSTOM roles (system/protected read-only), creates custom roles; hides controls on 403 / no session. `getJson`/`putJson` added to `lib/api.ts`; EN+UR i18n keys added; role-manager CSS. Route builds (`ƒ /admin/roles`).
- **i18n keys added:** `rbacTitle`, `rbacIntro`, `systemBadge`, `customBadge`, `newRolePlaceholder`, `createRole`, `save`, `saved`, `noAccess`, `signInRequired` (EN+UR parity).

### Acceptance mapping
1. Catalog sync idempotent on a non-fresh DB; stray rows never deleted → `permission.service.spec` (sync).
2. SUPER_ADMIN → full assignable set incl. later-added key; no NON_TENANT key to any tenant role; system roles get documented defaults → service spec (reconcile).
3. `@RequirePermission` blocks a lacking user (403) / allows a holder; `@Roles` blocks/allows → `roles.guard.spec` (unit) + `rbac.e2e.spec` (HTTP: 401 no-token, 403 DOCTOR, 200 ADMIN).
4. Custom tenant role created + keys set; protected roles not destructively editable; T2 roles isolated from T1 → service spec + `rbac-isolation.spec` (pglite).
5. Boot reconcile never crashes on a malformed row (logged) → service spec (crash-proof boot).
6. Gates green; isolation proven; flags ➖.

### Verification gates
- **typecheck:** 16/16 ✅ · **lint:** 10/10 ✅ · **turbo build:** 10/10 ✅ (route `ƒ /admin/roles` present).
- **jest:** `@mp/db` **27/27** (4 suites; +1 suite `rbac-isolation.spec`, 8 new tests) · `@mp/api` **60/60** (10 suites; +3 suites `permission.service`/`roles.guard`/`rbac.e2e`, ~28 new tests). Combined **87/87**.
- **Isolation:** RBAC T1/T2 proven in-CI via pglite on the REAL 01→04 migrations — `permissions` readable context-less (global), `app_roles` shows own+global/hides other-tenant custom roles, WITH CHECK blocks cross-tenant + tenant→global writes yet ALLOWS context-less global writes (boot), `role_permissions` inherits parent visibility.
- **i18n parity:** EN+UR for all new keys ✅. **Flag/vertical smoke:** ➖ (05). **Performance/Lighthouse:** ➖ (stub until 13). **White-label:** role-manager page uses shared shell classes/tokens.

---

## Build-step 03 — Foundation: Authentication

- **Work-type:** FEATURE (auth subsystem; always-on, never a flag). **Phase:** Foundation (item 3 of 39). **Branch:** `feat/03-auth` (stacked on `feat/02-tenancy-rls` → `feat/01-foundation-monorepo`; controller merges to `staging` at checkpoint).
- **Depends on:** 01, 02. **Date:** 2026-07-03.
- **Schema/migration:** NEW migration `20260703100000_auth`. Models: `User` (`id`,`tenantId @map(tenant_id)`,`branchId? @map(branch_id)`,`role Role`,`email?`,`passwordHash? @map`,`phone?`,`totpSecret? @map`,`totpEnabled @map`,`status UserStatus`,`failedLogins @map`,`lockedUntil? @map`,`createdAt`; `@@unique([tenantId,email])`,`@@unique([tenantId,phone])`,`@@index([tenantId])`; `@@map("users")`), `RefreshToken` (`userId @map`,`tokenHash @unique @map`,`familyId @map`,`revokedAt? @map`,`expiresAt @map`; `@@index([userId])`,`@@index([familyId])`; `@@map("refresh_tokens")`), `OtpChallenge` (`tenantId @map`,`phone`,`codeHash @map`,`purpose OtpPurpose`,`attempts`,`expiresAt @map`,`consumedAt? @map`; `@@index([tenantId,phone,purpose])`; `@@map("otp_challenges")`). Enums `Role` (17 values incl. SUPER_ADMIN…PATIENT), `UserStatus {ACTIVE,DISABLED}`, `OtpPurpose {LOGIN,VERIFY_PHONE,RESET}`. **RLS:** `SELECT apply_tenant_rls('users')` + `('otp_challenges')`. `refresh_tokens` is **intentionally un-scoped** (no `tenant_id`; addressed only by its unique high-entropy hash; tenant re-derived from the owning user under RLS). **Auth:** this IS the auth subsystem. **Feature-flags:** ➖ (auth always-on).
- **Do-NOT-break honored:** `runWithTenant` unchanged; OTP delivery goes through `OtpTransport` (the exact interface the 09 engine will implement — stub swaps out with no API change); auth stays always-on.

### Problem
Two principal classes (trained staff vs password-averse patients) need different auth, plus mandatory 2FA for money/admin roles, full session hygiene (rotation, reuse detection, lockout, reset), a tenant-bound principal that drives `runWithTenant`, and auditability.

### What changed
- **@mp/config:** added `JWT_ACCESS_SECRET`/`JWT_REFRESH_SECRET` (min 32; dev fallbacks **rejected in production** via superRefine), `ACCESS_TOKEN_TTL` (15m), `REFRESH_TOKEN_TTL_DAYS` (30), `LOCKOUT_THRESHOLD`/`LOCKOUT_DURATION_MIN`, `OTP_TTL_SECONDS`/`OTP_MAX_ATTEMPTS`, `APP_HOSTS` (+`appHosts()` helper).
- **@mp/db:** shared `hashPassword`/`verifyPassword` (argon2id via `@node-rs/argon2`, prebuilt — no node-gyp) used by BOTH API and seed so they never drift; re-exports `Role`/`UserStatus`/`OtpPurpose`; seed now creates the Ganatra **TENANT_OWNER** (no 2FA yet → first login forces enrollment; `SEED_OWNER_PASSWORD` overridable).
- **API auth module (`apps/api/src/auth`):** repository seams (`UserRepo`/`RefreshRepo`/`OtpRepo`/`TenantRepo`) with Prisma impls threading `runWithTenant`; `TokenService` (access JWT + mfa/reset step-tokens + **rotating opaque refresh `<tenantId>.<random>` hashed at rest, rotation + reuse→revoke-family**); `TotpService` (otplib, ±1 window); `OtpService` (6-digit, argon2-hashed, single-use, attempt-capped, rate-limited); `AuthService` orchestration (lockout, MFA gating for `MFA_ROLES`=SUPER_ADMIN/TENANT_OWNER/ADMIN/FINANCE, tenant resolution from host/slug, patient upsert-on-verify, stateless reset); `AuthEventService` (structured recorder seam → 06 audit); `StubOtpTransport`.
- **Endpoints (no `/api` prefix, all `@Public`):** `POST /auth/staff/login` (→ tokens or `mfa_required{mfaToken,enrollmentRequired}`), `/auth/staff/2fa`, `/auth/staff/2fa/enroll` + `/enroll/confirm` (accept mfaToken OR bearer), `/auth/staff/refresh`, `/auth/staff/password/reset/request`+`/confirm`, `/auth/patient/otp/request`+`/verify`, `/auth/logout`, and protected `/auth/me`.
- **Wiring:** global deny-by-default `JwtAuthGuard` (APP_GUARD, `@Public` opt-out) populates `{userId,tenantId,branchId,role}`; new `TenantContextInterceptor` (APP_INTERCEPTOR) establishes ALS tenant context from the principal AFTER guards (subscription wrapped inside the async-local scope). AppModule imports AuthModule **before** TenancyModule so the auth guard runs first.
- **Web (Simple/Pro n/a — public auth surfaces):** lean server-shell + client-island pages `/login` (staff→2FA/enroll), `/login/patient` (phone→OTP, big inputs, **Urdu default**), `/login/reset` (request + token-confirm). `AuthShell` + inline SVG mark + EN/UR toggle; `qrcode` **dynamically imported** for enrollment only (kept out of the login bundle). **i18n:** self-contained EN+UR dict (`lib/auth-i18n.ts`) — full framework in 08.
- **Flags exposed:** none (auth is always-on infra).

### Verification gates
- **typecheck:** 16/16 ✅ · **lint:** 10/10 ✅ · **turbo build:** 10/10 ✅ (login First-Load JS ~108 kB; QR lib excluded via dynamic import).
- **jest:** db **20/20** (3 suites; +5 — new `auth-isolation.spec.ts` runs the real 02+03 migrations on pglite and proves **T1/T2 user isolation**, otp_challenges scoped, refresh_tokens reachable by hash) + api **32/32** (7 suites; +18 — `auth.service.spec` covers Acceptance §1 rotation/reuse+logout, §2 mandatory-2FA enroll+verify, §3 OTP request/verify/expired/used/over-attempt, §4 lockout+reset, §5 tenant binding; `auth.e2e.spec` proves deny-by-default guard + `@Public` reachability over the real module graph; `tenant-context.interceptor.spec` proves ALS spans the async handler).
- **i18n parity:** EN+UR present for every auth key ✅. **Isolation:** users T1/T2 proven in-CI ✅. **Flag/vertical smoke:** ➖ (none). **Performance/Lighthouse:** login built lean (server shell + single client island, system fonts, one inline SVG, no blocking JS) to meet the 90+ mobile target; automated Lighthouse gate lands with the 13 UI foundation. **White-label:** login chrome uses brand CSS vars/tokens (no hardcoded colour); full theming in 07.

### Notes / follow-ups
- Auth events currently log via `AuthEventService`; **06 (audit-log)** absorbs them into the durable trail by swapping the sink (no call-site change).
- Password reset is a stateless short-TTL JWT; delivery is dev-logged until **09 (notifications)** provides the email/WhatsApp transport (same `OtpTransport`-style seam).
- On the shared app host, staff/patient tenant comes from `tenantSlug` (web default `NEXT_PUBLIC_TENANT_SLUG=ganatra-clinic`); custom-domain host path stays dormant.

---

## Build-step 02 — Foundation: Tenancy + RLS + Domain Resolver

- **Work-type:** FEATURE (core, most security-critical foundation). **Phase:** Foundation (item 2 of 39). **Branch:** `feat/02-tenancy-rls` (stacked on `feat/01-foundation-monorepo`; controller merges to `staging` at checkpoint).
- **Depends on:** 01. **Date:** 2026-07-03.
- **Schema/migration:** NEW migration `20260703090000_tenancy_rls`. Models `Tenant` (global registry — `id`,`name`,`slug @unique`,`status TenantStatus`,`customDomain @unique` **dormant**,`createdAt`; `@@map("tenants")`) and `Branch` (`id`,`tenantId @map("tenant_id")`,`name`,`isPrimary`,`createdAt`; `@@index([tenantId])`; `@@map("branches")`), enum `TenantStatus {ACTIVE,SUSPENDED}`. **Convention set (load-bearing):** snake_case DB columns via `@map`/`@@map` so RLS can reference `tenant_id` unquoted; every later business table carries `tenantId @map("tenant_id")` (+`@@index`), optional `branchId @map("branch_id")`, and calls `SELECT apply_tenant_rls('<t>')` in its migration. **RLS:** ON. Reusable SQL fn `apply_tenant_rls(text)` = `ENABLE` + **`FORCE` ROW LEVEL SECURITY** + policy `tenant_isolation USING/WITH CHECK (tenant_id = current_setting('app.tenant_id', true))`; applied to `branches` (tenants stays unbounded — the resolver reads it before context exists). **Auth/permission:** ➖ (principal→tenant wiring lands in 03). **Feature-flags:** ➖ (isolation is always-on, never gated).

### Problem
Isolation cannot be retrofitted. Needed `tenant_id` on every location-scoped table + Postgres RLS so even a buggy query can't cross tenants, a transaction-scoped tenant seam replacing the 01 `runWithTenant` placeholder, request-level tenant context, a host→tenant resolver (dormant custom-domain path), and tenant #1 seeded.

### What changed
- **`@mp/db`** (`packages/db/src/index.ts`): real `runWithTenant(tenantId, fn(tx))` — opens `prisma.$transaction`, runs `SELECT set_config('app.tenant_id', $1, true)` (parameterised → injection-safe; `is_local=true` → auto-resets, no cross-request leak), passes the tx client to `fn`. Added `assertTenantId` (rejects empty/malformed BEFORE any DB call — fail-closed on a mis-wired call site), `TENANT_GUC` const, `TenantContextError`. Signature finalised as load-bearing (`fn` receives `Prisma.TransactionClient`, not the bare client) — no external call sites existed to churn.
- **Migration** authored offline (`prisma migrate diff --from-empty` for table DDL) + hand-written `apply_tenant_rls()` plpgsql fn (`%I`-quoted identifier, `DROP POLICY IF EXISTS` for idempotency) + `SELECT apply_tenant_rls('branches')`.
- **Seed** (`prisma/seed.ts`, idempotent): upserts Ganatra Clinic (bare client — `tenants` has no RLS), then creates primary "Main Branch" **inside `runWithTenant`** (proves the seam E2E against a real DB). Wired `prisma db seed` (via `tsx`) into `deploy.sh` step 6b after `migrate deploy`.
- **API tenancy module** (`apps/api/src/tenancy/`): `tenant-context.ts` (AsyncLocalStorage — `runInTenantContext`/`getTenantContext`/`getTenantId`); `tenant-domain.resolver.ts` (`resolveTenantFromHost` → `custom-domain`|`app-host`|`unknown`; shared app host deliberately yields NO tenant — host never sole isolation); `tenant-scoped.decorator.ts` (`@TenantScoped()`); `tenant-context.guard.ts` (`TenantContextGuard`, global `APP_GUARD` — 403 on scoped route with no context); `tenant-context.middleware.ts` (binds `req.tenantId`→ALS for the whole async request); `TenancyModule` wired into `AppModule` (middleware `forRoutes('*')`).
- **UI (Simple/Pro):** ➖ (no surface this step). **i18n keys:** ➖.

### Verification gates
- **Typecheck:** 16/16 ✅  **Lint:** 10/10 ✅ (0 errors, 0 warnings)  **Build (turbo):** 10/10 ✅  **Jest:** `@mp/db` 15/15 (2 suites) + `@mp/api` 14/14 (4 suites) ✅.
- **Isolation (MANDATORY) ✅ proven in-CI without external infra:** `tenant-isolation.spec.ts` boots in-process Postgres (**pglite/WASM**), executes the **actual `migration.sql`**, and asserts as a non-superuser `app_user` (via `SET LOCAL ROLE`, since superusers bypass RLS): (a) each tenant sees ONLY its rows / the other is invisible; (b) `WITH CHECK` rejects cross-tenant INSERT; (c) own-tenant INSERT allowed & still invisible to the other; (d) **no context → 0 rows + INSERT rejected (fail-closed)**; (e) interleaved different-tenant txns don't leak; (f) `apply_tenant_rls()` reusable on a fresh table. `run-with-tenant.spec.ts`: `assertTenantId` accept/reject table + `runWithTenant` rejects bad ids before touching the DB. (Jest runs `@mp/db` with `NODE_OPTIONS=--experimental-vm-modules` for pglite's WASM dynamic import.)
- **Flags/vertical smoke:** ➖ (isolation is always-on). **Performance/Lighthouse:** ➖ (no public surface). **White-label:** dormant `customDomain` field reserved & unused.

### Notes / decisions
- **pglite** added as `@mp/db` devDep to make the RLS proof runnable in CI (real Postgres engine, no server/docker). The isolation test runs the real migration SQL, so it also validates the migration syntactically.
- **`FORCE ROW LEVEL SECURITY`** chosen so the policy binds even the table-owner role → the `DATABASE_URL` role MUST NOT be a superuser/BYPASSRLS role (documented in the migration). Seeds satisfy `WITH CHECK` because they run inside `runWithTenant`.
- `current_setting('app.tenant_id', true)` (missing_ok) returns NULL when unset → `tenant_id = NULL` → 0 rows / rejected INSERT = fail-closed; a request without tenant context is never silently global.
- Spec §2.5 names an "interceptor"; realised as **middleware** for the ALS bind (correct place to wrap the whole async request) plus a **guard** for enforcement. Principal→tenant population of `req.tenantId` lands in 03 (auth).
- `prisma validate` returns non-zero locally only because `DATABASE_URL` is unset (not a schema error); `prisma generate`, `migrate diff`, and the pglite migration run all confirm validity. CI sets a placeholder `DATABASE_URL`.

---

## Build-step 01 — Foundation: Monorepo + DB + Prisma base + health endpoint + deploy.sh + CI

- **Work-type:** FEATURE (greenfield platform base). **Phase:** Foundation (item 1 of 39). **Branch:** `feat/01-foundation-monorepo` (off the initial commit; controller merges to `staging` at checkpoint).
- **Depends on:** nothing (first step). **Date:** 2026-07-03.
- **Schema/migration:** empty Prisma baseline (`packages/db/prisma/schema.prisma` — datasource postgresql + generator only, NO business models). Baseline migration `00000000000000_baseline` (no-op `SELECT 1;`) + `migration_lock.toml (postgresql)`. **RLS:** ➖ (no business tables — begins in 02). **Auth/permission:** ➖ (begins 03/04). **Feature-flags:** ➖ (engine scaffolded only; real engine in 05).

### Problem
There was no codebase. Every later step assumes a pnpm/Turborepo monorepo, Prisma+Postgres, a health endpoint, the `@mp/*` shared packages (incl. `@mp/flags`, `@mp/brand`), a working `deploy.sh` (with the learned hard rules + Lighthouse-gate hook), and CI. Built once, correctly.

### What changed
- **Monorepo:** pnpm workspace (`pnpm-workspace.yaml`) + Turborepo (`turbo.json`: build/typecheck/lint/test/dev; `^build` ordering so packages build before apps). Root `tsconfig.base.json` (CommonJS/node to avoid ESM↔CJS friction with NestJS decorators), flat ESLint (`eslint.config.mjs`, lenient at foundation), Prettier, `.gitignore`, `.npmrc`, `.env.example`.
- **Packages (`packages/*`):**
  - `@mp/config` — zod-validated env; **lazy** `loadConfig()` (throws on boot if invalid, never on import so build/typecheck don't trip); `isStaging()` fail-closed helper; vars incl. `APP_ENV`.
  - `@mp/db` — shared Prisma singleton (hot-reload-guarded) + **frozen `runWithTenant(tenantId, fn)` seam** (placeholder; real `SET LOCAL app.tenant_id` RLS wiring lands in 02) + `disconnect()`.
  - `@mp/shared` — `Result`/`ok`/`err`, `nowIso`, `isNonEmptyString`, `APP_NAME`.
  - `@mp/flags` — capability-engine SCAFFOLD: `FlagKey`, five `VERTICAL_PRESETS`, `CapabilityContext`, `isFeatureEnabled` (fail-closed). Real engine 05.
  - `@mp/brand` — white-label SCAFFOLD: canonical DEFAULT theme = ARCHITECTURE.md §11 tokens (Brand Teal `#06888D` … Alert `#B4453C`), identity (name/logo/sender), fonts (display/body/urdu), `brandCssVars()`/`brandCssVarString()`.
  - `@mp/ui` — design-system SCAFFOLD: `cn()`, `themeCssVars()` re-export. Real shadcn/ui system 13.
  - `@mp/ai` — provider-agnostic gateway SCAFFOLD: `createAiGateway` (`isConfigured()=false`), `AI_ASSIST_ONLY` guarantee. Real suite 35–39.
- **Apps (`apps/*`):**
  - `api` — NestJS 11; `GET /health` → `{status:'ok', buildId, ts}` mounted at ROOT (**no `/api` prefix**; verified `/api/health` 404s); `main.ts` validates env via `loadConfig` on boot; Jest (`ts-jest`) health spec.
  - `web` — Next.js 15 App Router PWA; tokenized placeholder home rendered entirely from `@mp/brand` tokens (white-label honored, no hardcoded brand); `manifest.webmanifest` + `sw.js` app-shell + client SW registration; `icon.svg`; brand CSS vars injected on `<html>`. Standalone tsconfig (bundler/esnext/jsx-preserve); committed 2-line `next-env.d.ts`.
  - `worker` — placeholder job runner (config-validate + heartbeat + graceful SIGTERM/SIGINT).
- **Deploy/infra:** `deploy.sh` per §15 — Node detection (nvm → aaPanel `/www/server/nodejs/*` → system), corepack/pnpm, source server `./.env`, pull/reset existing clone (never clone, **branch fence blocks main/master/production/prod** + dangerous-arg fence), `pnpm install --prod=false --frozen-lockfile`, `prisma generate` → `migrate deploy`, fresh `BUILD_ID`, `turbo build`, **`run_lighthouse_gate()` placeholder hook** (real gate 13), **`pm2 restart` (not reload)** via `ecosystem.config.js` (`mp-api`/`mp-web`/`mp-worker`), graceful `/health` retry loop. `ecosystem.config.js` PM2 defs. `.github/workflows/ci.yml` mirrors deploy ordering (install → prisma generate → typecheck → lint → build → test) + Lighthouse stub job. Existing `deploy-staging.yml`/`deploy-production.yml` left intact.

### Verification gates
1. **typecheck:** green — 16/16 turbo tasks. Clean-clone web typecheck (no `.next`) verified exit 0 with committed 2-line `next-env.d.ts`.
2. **lint:** clean — 0 errors, 0 warnings (removed stray `eslint-disable` directives).
3. **turbo build:** green — 10/10 (web `next build` static; api/worker/packages `tsc`).
4. **jest:** 2/2 (1 suite; +2 new) — HealthController.
5. **i18n:** ➖ (no user-facing UI strings yet; two-tier i18n base lands in 08).
6. **Isolation/RLS:** ➖ (no business tables; `runWithTenant` seam frozen for 02).
7. **Feature-flag / vertical smoke:** ➖ (no gated capability yet; flag engine scaffolded, real in 05).
8. **Performance/Lighthouse:** ➖ (no real public surface yet; placeholder web First Load JS ~102 kB. Perf foundation + Lighthouse gate land in 13).
9. **Design self-check:** placeholder is tokenized (brand palette swatches, serif display, light/dark via `color-scheme`), not raw HTML; full design system 13.
10. **White-label:** ✓ web reads brand tokens/identity from `@mp/brand`; no hardcoded logo/color/name.
11. **Accountability:** ➖ (no state changes yet; AuditLog begins 06).
12. **Offline:** SW shell installs + network-first navigation fallback (real offline core 10).
13. **.env.example:** ✓ every var a placeholder incl. `APP_ENV`; **no secrets/real hosts**.

### Notes / manual-run caveats
- `prisma migrate deploy` and a full end-to-end `deploy.sh` run require a live Postgres 16 + the aaPanel server; not runnable in this build sandbox (no Postgres/PM2). Schema validated (`prisma validate` ✓) and the empty baseline applies cleanly on a fresh DB in CI/deploy. API boot + `/health` + no-`/api`-prefix verified locally against dist.
- `next build` rewrites `next-env.d.ts` to add a `.next/types/routes.d.ts` reference (that path is gitignored); the committed 2-line version is intentional so clean-clone typecheck (which runs before build in CI) stays reproducible.

---
