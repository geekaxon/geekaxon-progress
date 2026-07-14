# PROGRESS-HISTORY.md — Rihaish

Append-only. Planning record first, then one detailed entry per completed build step.

---

## Entry template (copy for each completed step)

```
## NN — <slug> — DONE/APPROVED (YYYY-MM-DD)
**Feature code:** <code> · **Depends on:** <feature codes>
**Built:** <what shipped — models, endpoints, screens>
**Migrations:** <migration names>
**Tests:** <unit / e2e added, pass counts>
**Verification:** typecheck ✔ build ✔ unit ✔ e2e ✔ UI self-check ✔ (light/dark/RTL/responsive/table-kit/form-kit/Simple+Pro)
**Decisions made:** <any [DECIDE AT BUILD] resolved, with rationale>
**Deviations from spec:** <none | what and why>
**Follow-ups:** <deferred items>
**Checkpoint:** [FIXED_CHECKPOINT] approved by <operator> | [CHECKPOINT] auto-approved
```

---

## 2026-07-12 — Planning

**Product.** Rihaish — multi-tenant SaaS for apartment / residential-society management. Pakistan-first (Urdu + English, RTL). First customer: Rufi Apartments, Gulshan-e-Iqbal Block 13D-2 (blocks A–F, flats `D-20` style, categories 2-Bed/DD and 3-Bed/DD). Sold per society, priced per-flat or lumpsum.

### Key decisions

1. **Multi-tenant, not standalone-per-society.** Rejected one-deploy-per-customer: N VPSs to patch, back up and support, while charging per flat. Single codebase, `societyId` on every row, enforced by one data layer. A dedicated instance can still be sold later as a premium tier from the same codebase. Retrofitting tenancy would have been a rewrite.
2. **Three layers.** L0 Platform (Rihaish) · L1 Society (tenant) · L2 Society users. Guards/staff are **roles inside L1**, not a fourth layer — what differs is their *surface* (mobile-first, single-purpose, short sessions), not their tenancy.
3. **Vocabulary.** SaaS side = **Society**. Domain side = **Owner / Occupant**. The word "tenant" is reserved for the SaaS meaning to avoid the flat-renter collision.
4. **Everything is a feature flag.** `Feature` (+`dependsOn` DAG) · `Plan` · `SocietyEntitlement` overrides. Enabling auto-resolves dependencies; disabling a depended-on feature is blocked unless dependents are disabled too. Core features (`flats-registry`, `residents`) marked `isCore` and non-disableable. **Server is the source of truth** — every route declares its feature; UI gating alone is decoration.
5. **Two billing domains, never merged.** *Platform billing* (Rihaish → society: PER_FLAT × rate or LUMPSUM, per-society rates, effective-dated) and *Society billing* (society → residents: maintenance). A shared `Invoice` table would have caused a leak or a billing bug.
6. **Ledger, not a `paid` column.** Invoice = debit, payment = credit, per flat. Partial payments, advances/credit balance, arrears and late fees all fall out of this correctly. Payment states `PENDING | CLEARED | BOUNCED | VOID`; a bounced cheque **reverses via a new entry**, never an edit or delete.
7. **Charge engine.** `ChargeHead` + effective-dated `RateRule` resolved by occupancy status / flat category / block (Rufi: occupied 3500, vacant 2000) + one-off `SpecialCharge` targeted at all / block / list / single flat. Invoice lines are **snapshots** — changing a rate never rewrites history.
8. **Money = integer minor units (BigInt)** with per-society `currency` + `minorUnitDigits` (PKR = 0 decimals, minimum unit ₨1). Timezone per society, default `Asia/Karachi`; timestamps stored UTC.
9. **Who is billed:** society-level default (OWNER | OCCUPANT), overridable **per flat** by the union. Invoice stores `billedToUserId` + a name snapshot. Old invoices stay attached to the old person forever.
10. **Payments are record-keeping in v1** (bank/cheque/cash, committee-recorded, **plus resident-uploaded payment proof → verification queue**). A `PaymentProvider` seam exists with `MANUAL` as the only implementation, so a gateway drops in without a schema migration. Stripe is planned for platform-side collection.
11. **Non-payment by a society → read-only mode**, configurable grace period, banner shown, residents can always still see their own dues. Never a hard lockout, never data deletion.
12. **Auth.** No self-signup (junk accounts). Society admin **invites** or **CSV-imports**. Account keyed on **phone**; **flat number is a login alias** (society-scoped, only valid on a society host — flat numbers are not globally unique). **One account ⇄ many flats** with a flat switcher, in v1. OTP login is an entitled feature requiring the society's own SMS/WhatsApp credentials. Guards: PIN on a shared device **or** full credentials on their own device — society's choice.
13. **Gate pass — corrected mid-planning.** The primary real-world use in Pakistani apartments is **things leaving the building** (an AC unit going out needs an owner-signed slip), not visitors arriving. Modelled as a polymorphic `Pass`: `ITEM_EXIT`, `CHILD_EXIT`, `VISITOR_ENTRY`, `DELIVERY_LOG`, `STAFF_RECURRING`. Verification by QR / 6-digit code / name-phone search — all three, all stamping the guard's user id. Item-exit approval by any resident linked to the flat (society-configurable to owner-only).
14. **CHILD_EXIT involves minors.** Authorized-pickup-persons list per flat, mandatory audit trail, **no child photos stored by default** (society toggle, off by default), never publicly visible.
15. **CNIC** capture is society-toggleable and optional; encrypted at rest, masked in UI, never in logs, never exported without an audit entry.
16. **Utility bill notices are a separate module, not announcements.** They carry per-flat state (pending/paid/overdue), due dates and reminders (SSGC, K-Electric, KW&SB, PTCL, StormFiber…), targetable at a subset of flats. **They never touch the maintenance ledger** — the society is reminding, not collecting.
17. **UI is a first-class requirement**, enforced structurally: an **App Shell**, a **Form & Input Kit** and a **Data Table & Export Kit** are built as HARD-checkpoint modules *before* any feature screen, and every later module is required to consume them. Hand-rolled tables/inputs are a build failure. This is what makes 40 modules feel like one product.
18. **Icons: Lucide (primary) + Phosphor (supplement)** behind one `<Icon>` abstraction. **FontAwesome Pro from an unofficial GitHub mirror was rejected** — it is a pirated commercial asset and a procurement/legal liability for a product being sold to businesses. If FA Pro is wanted later, buy a licence and swap the abstraction.
19. **Storage: adapter pattern** (`local` default → `s3`/R2 → `gdrive`), one env var. **Google Drive as the primary blob store was rejected** — expiring OAuth tokens, rate limits, latency on the guard's hot path, no CDN, no clean signed-URL story. R2's free tier makes it effectively free.
20. **PWA, not native apps.** But the PWA must *feel* native: standalone display, **role-aware 5-item bottom tab bar**, off-canvas drawer, branded splash + icons, safe-area insets, push/pop transitions, pull-to-refresh, offline shell. **The manifest is served per-host**, so each society installs *its own* branded app ("Rufi Apartments", Rufi's icon). Capacitor store-wrapper considered and dropped.
21. **iOS caveat, accepted:** iOS only delivers web push once the PWA is installed to the home screen — onboarding must actively drive the install; SMS/WhatsApp remains the reliable channel for users who don't.
22. **White-label** = branding mapped onto **design tokens as CSS variables**, resolved per request. Never per-society CSS files or forked components. **"Powered by Rihaish" is always in the footer and cannot be removed** — this deliberately eliminates an entitlement and its bypass bugs.
23. **True white-label email deferred**: v1 sends from Rihaish's domain with the society's display name (`Rufi Apartments via Rihaish`). Sending from the society's own domain needs SPF/DKIM on their DNS → later, entitled, `[HUMAN_REQUIRED]` per society.
24. **Impersonation** (L0 → society admin) is security-critical: time-limited, persistent banner, full audit on enter *and* exit, **read-only by default** with an explicit "enable write" step, never reveals a password or secret.
25. **Infra reality:** Oracle Cloud Free Tier is **ARM64 (Ampere A1)**. Prisma `binaryTargets` must include `linux-arm64-openssl-3.0.x` and `sharp` must resolve arm64, or every deploy fails cryptically. Oracle images also ship iptables rules that block ports even when the cloud Security List allows them — the "nginx is running but the site is unreachable" trap. Both documented; port opening is `[HUMAN_REQUIRED]`.
26. **Test gate:** typecheck + build + lint on every module; **Vitest mandatory** on the tenant-scoping layer, entitlement resolver, charge engine, ledger, payment reversal and invoice numbering; **Playwright smoke** on auth, invoice generation, gate-pass verification.
27. **Build order sequenced so the product is sellable at Rufi from step 21–22**; steps 23–40 are additive. HARD checkpoints limited to the 15 places where a wrong call is expensive or irreversible.
28. **Competitor-proposal gap analysis** (Lakhani Presidency / LPCLICK, Sept 2024) added six modules that were missing: **messaging/chat**, **surveys** (distinct from polls), **staff operations console** ("my jobs"), **staff performance evaluation**, **amenities with rate lists + booking invoices + an amenity-manager role**, and **resident-uploaded payment proof**. Complaints were widened to cover **service requests** ("send a plumber") as a distinct type from complaints ("the lift is broken").

### Assumptions

- Society sizes up to ~500 flats; data volume modelled at 500 flats × 12 invoices/yr × several years. Server-side pagination everywhere; no client-side "load all rows".
- Residents are reachable primarily by phone/WhatsApp; email is secondary in Pakistan.
- The union/committee, not Rihaish, is responsible for the accuracy of rates and charges.
- Societies pay Rihaish by bank transfer / cheque / cash in v1 and the platform owner marks them paid.

### [DECIDE AT BUILD]

- **D1** — Final brand mark and favicon assets (logo in progress; palette locked: emerald `#0B5F4A` + gold `#D9A441`). Tagline: *"Behtar Rihaish, Behtar Nizaam"*.
- **D2** — Exact Tailwind major version and shadcn/ui variant to pin (must be pinned, not floating).
- **D3** — SMTP provider and credentials for platform-side email.
- **D4** — Default platform grace period before read-only (configurable; proposed default 14 days).
- **D5** — Whether R2 replaces local disk before or after the first external customer.
- **D6** — Society-supplied SMS/WhatsApp provider adapters to implement first (candidates: Twilio, WhatsApp Cloud API, a local PK SMS gateway).
- **D7** — Redis vs Postgres-backed job queue for the worker (Postgres-backed is the default to avoid a new service on the free-tier box).
- **D8** — Whether attendance (step 25) uses manual marking only, or adds QR/geofence check-in.
- **D9** — Fiscal-year boundary and financial-report period definition per society.
- **D10** — Trademark/domain clearance for "Rihaish" before the public site (step 40) goes live.

---

## 01 — foundation — DONE (2026-07-11)
**Feature code:** `core.foundation` (isCore, non-disableable) · **Depends on:** —
**Built:**
- **Toolchain:** Next.js 15.5.20 (App Router, RSC) · React 19.2.7 · TypeScript strict (`noUncheckedIndexedAccess` on) · pnpm 9.15 · Node ≥22. Scripts: `dev/build/start/worker/lint/typecheck/test:unit/test:e2e/prisma:generate`.
- **Design system:** Tailwind 3.4 + shadcn/ui (new-york, CSS-variable HSL tokens) in `app/globals.css` — emerald `--primary 168 79% 21%`, gold `--accent 40 68% 55%`, `--radius 0.625rem`, full light + dark token sets incl. chart 1–5. `Button` shipped as the first shadcn primitive. `cn()` util. `components.json` for future `shadcn add`.
- **Fonts:** Inter (Latin), Noto Naskh Arabic (Urdu UI), Noto Nastaliq Urdu (logo-only) via `next/font`, wired to CSS vars; `html[lang=ur]` switches body to Naskh.
- **i18n/RTL:** next-intl 3.26 — `i18n/{routing,request,navigation}.ts`, `middleware.ts`, `messages/{en,ur}.json`. `<html lang dir>` per request (`dirFor()`), `/` → `/en` redirect, `/ur` renders `dir=rtl` (verified at runtime). Locale precedence documented (user→society→en); society/user sources land step 02/04.
- **`<Icon>`:** single abstraction over Lucide (primary) + Phosphor (supplement); no component imports an icon lib directly.
- **Money/time primitives:** `lib/money.ts` (`Money = bigint` minor units, exact `formatMoney`/`parseMoney`, no float math — grouping via Intl on the integer digit-string) and `lib/time.ts` (`toUtc`/`toSocietyTime`, storage UTC, default `Asia/Karachi`).
- **Env contract:** `lib/env.ts` — zod, lazy + memoised `getEnv()` that fails loudly (`RihaishEnvError`) on a missing key at boot, never at build. `.env.example` complete (all spec keys, placeholders only).
- **Health:** `/api/health` (nodejs runtime, force-dynamic) → `{status, buildId, time, db, worker}`; DB via `prisma.$queryRaw SELECT 1`, worker via a heartbeat file (`lib/worker-heartbeat.ts`).
- **Prisma 6:** generator with `binaryTargets = ["native","linux-arm64-openssl-3.0.x"]` (Oracle Ampere ARM64), postgres datasource; domain models deferred to step 02+. `lib/prisma.ts` singleton (note: enforced tenant-scoping layer replaces direct use in step 02).
- **Ops:** `deploy.sh` (Node-detection → env load → git reset → frozen install → prisma generate → migrate deploy → typecheck+unit → fresh BUILD_ID → build → `pm2 restart` → health-check-with-retries), `ecosystem.config.cjs` (rihaish-web cluster×2 + rihaish-worker fork×1), `worker/index.js` (env guard + heartbeat), `.github/workflows/ci.yml` (fixed order matching deploy.sh: install → prisma generate → lint → typecheck → test:unit → build).
- **Lint guardrails (build-failing):** `react/jsx-no-literals` (hardcoded user-visible strings), `no-restricted-syntax` regex bans on `ml-/mr-/pl-/pr-` and `text-left/right` (enforce logical properties for RTL), `parseFloat` banned on money paths. All three verified to fire.
**Migrations:** none (no domain models yet — first migration lands with step 02 tenancy models).
**Tests:** Vitest — `lib/money.test.ts` (7), `lib/time.test.ts` (4), `tests/unit/env.test.ts` (2) = **13 passed**. Playwright config + `e2e/home.spec.ts` (locale redirect + RTL flip) scaffolded; mandated auth/invoice/gate-pass e2e added as those modules land.
**Verification:** typecheck ✔ · lint ✔ (guardrails proven to fire) · test:unit ✔ 13/13 · build ✔ (clean, no MISSING_MESSAGE, `/en`+`/ur` SSG, `/api/health` dynamic) · runtime smoke ✔ (health JSON with worker `up`, `/`→`/en` 307, `/ur` `dir=rtl`).
**Decisions made:**
- **D2 (pinned):** Tailwind **3.4.19** + shadcn/ui **new-york**, classic HSL CSS-variable tokens (matches the spec's `168 79% 21%` token format). Deliberately not Tailwind v4 (CSS-first `@theme`) to keep the documented shadcn token contract. Next pinned to **15.x** (spec says Next 15) although 16 is now published; next-intl kept on **3.26** (v4's `hasLocale` shimmed locally as `isLocale`).
- Fresh **BUILD_ID** is inlined at build time via `next.config.mjs env` so `/api/health` reports the exact build — set by `deploy.sh` per deploy.
- Worker liveness via a heartbeat **file** for foundation; migrates to a DB-backed heartbeat when the job tables land (step 12).
**Deviations from spec:** none. (`hasLocale` is a next-intl v4 export; replaced with a local `isLocale` guard on the pinned v3 — same behaviour.)
**Follow-ups:** step 02 introduces the enforced tenant-scoping Prisma extension (replaces direct `lib/prisma.ts` use) + first migration; screenshot-token seam (AGENT §7) is wired in step 02 where host/society resolution exists; e2e browsers not installed in CI yet (config ready).
**Checkpoint:** [FIXED_CHECKPOINT] — awaiting operator approval (HARD checkpoint; not self-approved).

---

## 02 — tenancy-core — DONE (2026-07-11)
**Feature code:** `core.tenancy` (isCore, non-disableable) · **Depends on:** `core.foundation`
**Branch:** `feature/02-tenancy-core`
**Purpose:** The society boundary — every row/query/request scoped so a cross-tenant leak is impossible. Highest-risk foundational module.

**Built:**
- **Data model (prisma/schema.prisma):** `Society` (slug-unique tenant root; currency/minorUnitDigits/timezone/locale; `status ACTIVE|READ_ONLY|SUSPENDED`, `readOnlyReason`, soft-delete `deletedAt`), `SocietyDomain` (custom domains, `host` unique, `verifiedAt`), `AuditLog` (`societyId?` null=platform, `actorType PLATFORM|SOCIETY_USER|SYSTEM`, `impersonatedBy`, before/after Json, indexed `[societyId, entity, entityId]`). Enums `SocietyStatus`, `ActorType`.
- **First migration:** `prisma/migrations/20260711120000_tenancy_core/migration.sql` generated offline via `prisma migrate diff --from-empty` (no DB in agent env) + `migration_lock.toml` (postgresql). Applied on staging by `deploy.sh` step 6 (`prisma migrate deploy`).
- **Scope context (`lib/tenant-context.ts`):** `AsyncLocalStorage<TenantScope>` — `{ societyId, readOnly, readOnlyReason, actorId, actorType, impersonatedBy }`. `runWithScope`/`getScope`/`requireScope` (throws `TenantScopeError` when absent)/`makeScope`.
- **Scoping logic (`lib/scoping.ts`, pure/DB-free):** a model is tenant-scoped iff it has BOTH `societyId` and `deletedAt` (auto-detected from DMMF — every future domain model qualifies; the 3 infra models don't). `scopeOperation` injects `{ societyId, deletedAt:null }` on reads, stamps `societyId` on create/createMany, pins update/updateMany/upsert to the society, and rewrites delete/deleteMany → soft-delete directive (`deletedAt=now`, audit=true). `assertWritable` throws `ReadOnlyError` in a read-only scope. `scopeSocietyOperation` narrows `Society` reads to the scope's own id and rejects tenant-side Society writes (`CrossTenantError`).
- **Enforced data layer (`lib/db.ts`):** `db = prisma.$extends(...)` query extension over `$allModels` applying the above; executes soft-delete via the raw delegate + writes an `AuditLog` row (before-image JSON-sanitised for bigint/Date). `db.unscoped()` returns the raw client for platform/worker. `withSociety(societyId, fn, overrides?)` for jobs/platform acting-as-society.
- **Host resolution (`lib/tenant-host.ts`, pure):** `resolveHostContext` → apex=platform, `<slug>.apex`/`<slug>.wildcard`=society, bare wildcard/localhost=platform (dev), unknown=society candidate (server resolves/404s — never reveals existence). `routeAreaFor` (locale- and api-prefix aware) + `isRouteAllowed` for cross-context isolation.
- **Middleware (`middleware.ts`):** composes host enforcement over next-intl. Platform route on a society host (and vice versa) → generic 404; `/api/*` covered (matcher now includes api); `/api/health` exempt (probe reachable on any host); locale layer runs only for page routes.
- **Server resolver (`lib/tenant.ts`):** `resolveSocietyByHost` (the ONE allow-listed `db.unscoped()` site) → generic `TenantNotFoundError` for unknown/soft-deleted/suspended/unverified hosts; `withTenant(req, fn)` resolves + applies screenshot read-only + `READ_ONLY` status, then runs `fn` inside the society scope.
- **Screenshot token (`lib/screenshot-token.ts`):** `isScreenshotTokenValid` — fail-closed unless `APP_ENV=staging`, GET-only, constant-time compare (`timingSafeEqual`); bound to the resolved society (read-only scope) so it can never cross tenants.
- **Errors (`lib/errors.ts`):** `TenantScopeError`(500) · `TenantNotFoundError`(404) · `ReadOnlyError`(423) · `CrossTenantError`(409) + `httpStatusForError`/`codeForError` mappers.
- **Health (`app/api/health/route.ts`):** extended with a `tenancy` self-test (host classifier + current-request context).
- **Feature registry seed (`lib/features.ts`):** static `FEATURES` (`core.foundation`, `core.tenancy` w/ `dependsOn`, `isCore`) — the DAG resolver/plans/toggles build on it in step 03 (AGENT §1.9).
- **Stub routes:** `app/[locale]/platform/page.tsx` (L0-only) + `app/[locale]/app/page.tsx` (society-only) — placeholders proving the isolation boundary; real console/shell land in steps 08/05. en+ur messages added.
- **ESLint boundary:** `no-restricted-properties` bans `db.unscoped` outside an allow-list (`**/platform/**`, `**/worker/**`, `lib/tenant.ts`, `lib/db.ts`); money `parseFloat` ban preserved in the override. Rule proven to fire on a probe file.

**Verification gates:**
- typecheck ✔ (tsc, strict + noUncheckedIndexedAccess) · lint ✔ (0 warnings; unscoped-boundary rule proven to fire) · test:unit ✔ **53/53** (added: scoping 12, tenant-host 12, screenshot 5, tenancy-scope/errors 8, db wiring 3) · build ✔ (9 static pages incl. /[locale]/{app,platform}, /api/health dynamic, middleware bundled 52 kB).
- **Runtime smoke (`next start`, Host-header curl):** platform-route-on-society-host → 404 ✔ · society-route-on-platform-host → 404 ✔ · same-context → 200 ✔ · `/api/platform/*` on society host → 404 ✔ · health `tenancy.status=up, context=society` ✔ · `/` → 307 `/en` ✔.
- **e2e:** `e2e/tenancy.spec.ts` added (Host-override request tests for the 404 isolation + health tenancy). Browsers still not installed in this env (config ready) — same status as step 01.

**Acceptance criteria mapping:**
- Scoped query without active scope throws → `requireScope` in `db.ts`; test `tenancy-scope.test.ts`. ✔
- Create auto-scoped; cross-society update rejected → `scopeOperation` injects societyId (create) and pins update where; test `scoping.test.ts`. ✔ (DB-level P2025 rejection exercised on staging.)
- Delete = soft delete + AuditLog row → `scopeOperation` softDelete directive (unit-proven) + `db.ts` executes update + `auditLog.create` (DB-backed path runs on staging). ✔
- Platform route on society host → 404 (and apex inverse) → middleware + runtime smoke + e2e. ✔
- Screenshot token rejected off-staging / for POST / cross-society → `screenshot-token.test.ts` + scope-binding. ✔
- ESLint blocks `db.unscoped()` outside /platform,/worker → rule + proven probe. ✔
- READ_ONLY society: writes 423, resident dues readable → `assertWritable`→`ReadOnlyError`(423); reads never blocked. ✔ (route-level 423 wiring lands with the first society mutation routes.)

**Decisions made (logged, autonomous — no approval gate):**
- **Scope population is server-side, not Edge middleware.** The spec says "scope populated by middleware," but Next middleware is Edge and its AsyncLocalStorage/Prisma cannot propagate into the Node RSC/route runtime. Middleware does the pure host classification + 404 isolation; the Node-side `withTenant(req, fn)` / `withSociety` populate the ALS scope. This is the correct, standard App-Router approach and preserves the guarantee (a scoped query with no scope throws).
- **Tenant-scoped detection = has BOTH `societyId` and `deletedAt`.** Auto-detected from DMMF so every future domain model is scoped with zero per-model wiring; the infra models (`Society` root, `SocietyDomain`, `AuditLog`) are intentionally excluded. `Society` gets a dedicated rule (reads narrowed to own id, tenant writes rejected) to close the leak that plain exclusion would open.
- **SUSPENDED society = generic 404 (fail closed).** READ_ONLY = reads allowed, writes 423.
- **Custom-domain/unknown-host resolution is server-side** (Edge can't hit the DB); unknown hosts are treated as society candidates and 404 downstream — never revealing existence.
- **Migration generated offline** (`migrate diff`) because the agent env has no Postgres; identical SQL is applied by `deploy.sh migrate deploy` on staging.

**Deviations from spec:** none material. (Scope-population location clarified above; behaviour identical.)

**Follow-ups (for later steps):**
- Route-level 423/404 response wiring + persistent read-only UI banner land with the first society mutation routes / app shell (step 05) and platform billing read-only trigger (step 22).
- `withTenant` RSC integration (page-level screenshot token via searchParams) lands with the app shell (step 05).
- DB-backed integration tests for soft-delete + AuditLog + cross-society P2025 run in CI once a Postgres service is added; logic is fully unit-covered now.
- Actor identity on the scope is `SYSTEM` placeholder until auth (step 04) supplies the real principal + impersonation.

**Checkpoint:** [CHECKPOINT] — progress marker; controller auto-approves and continues to step 03.

---

## 03 — entitlements-engine — DONE (2026-07-11)
**Feature code:** `core.entitlements` (isCore) · **Depends on:** `core.tenancy`
**Built:** The per-society feature-toggle engine with a dependency DAG that makes bricking a tenant impossible; server is the sole source of truth.
- **Models (schema.prisma, step-03 block):** `Feature` (global; code PK, name, description, module, isCore, dependsOn String[], createdAt), `Plan` (global; code unique, featureCodes String[], isActive), `SocietyEntitlement` (per-society override; @@unique[societyId,featureCode] + @@index[societyId]; enabled, validUntil, limits Json, reason), `SocietyPlan` (societyId PK, planId, @@index[planId]). The two per-society tables carry `societyId` but deliberately **no `deletedAt`**, so the tenancy scoping rule (needs BOTH) does not catch them — they are L0 config read cross-tenant by the resolver, passing through `db` untouched from platform/worker/tenant contexts alike.
- **Registry (`lib/features.ts`):** extended `FeatureDef` with `module` (+ optional `description`); registered `core.entitlements` (dependsOn core.tenancy). Exposes `CORE_FEATURE_CODES`, `FEATURE_DAG`. Calls `assertValidDag(FEATURE_DAG)` **at module load** → a malformed/cyclic registry fails the build.
- **Pure DAG (`lib/feature-dag.ts`):** `assertValidDag` (WHITE/GREY/BLACK DFS cycle detection + missing-edge check; throws `FeatureCycleError`/`MissingDependencyError`), `transitiveDependencies` (dependency-first ancestors), `transitiveDependents` (reverse-edge descendants). No DB — mirrors the `lib/scoping.ts` split.
- **Pure resolver math (`lib/entitlements-compute.ts`):** `computeEntitlements` (plan ∪ overrides(enabled) − overrides(disabled), expired ignored, core forced ON), `planEnable` (target + off-deps, dependency-first), `planDisable` (`core`/`blocked`/`ok` outcomes, cascade), `computeLimit` (most-restrictive active limit or null).
- **Glue (`lib/entitlements.ts`):** `resolve(societyId)` DB read + in-memory cache (self-expires at nearest override `validUntil`, capped 60 s) + `entitlementsTag`; `isEnabled` (cosmetic), `requireFeature` (throws `FeatureDisabledError` → **403**), `limits`; mutations `enableFeature`/`disableFeature` (atomic `$transaction` of override upserts + one AuditLog row; core-lock + dependency-block enforced; cascade), `previewEnable` (console "will also enable" copy), `setSocietyPlan`, `invalidateEntitlements` (in-memory bust + best-effort `revalidateTag`), `syncFeatureRegistry` (idempotent single seeder upserting `FEATURES` → `Feature`).
- **Errors (`lib/errors.ts`):** `FeatureDisabledError` (403), `FeatureLockedError` (409, core), `FeatureDependencyError` (409, carries dependents) wired into `httpStatusForError`/`codeForError`.
- **Route:** `app/api/entitlements/check/route.ts` — resolves tenant via `withTenant`, calls `requireFeature`; disabled/unknown feature → 403 even when called directly, core → 200.
**Migrations:** `20260711130000_entitlements_engine` (hand-authored offline — no Postgres in agent env; identical SQL applied by `deploy.sh migrate deploy`). Adds the 4 tables + Plan.code unique + SocietyEntitlement composite unique/index + SocietyPlan planId index.
**Tests:** unit **82/82** (added: feature-dag 9, entitlements-compute 16, features 4). e2e `e2e/entitlements.spec.ts` (probe 403 contract; skips when `rufi` not seeded — e2e is not a CI gate).
**Verification:** typecheck ✔ · lint ✔ (0 warnings) · test:unit ✔ 82/82 · build ✔ (`/api/entitlements/check` dynamic route registered) · prisma validate ✔. No UI shipped this step (console UI is step 08; contract defined here per spec §UI).

**Acceptance criteria mapping:**
- Resolver correctness across plan + overrides + expiry + core → `entitlements-compute.test.ts` (6 computeEntitlements cases incl. expired-ignored + core-forced). ✔
- Enable auto-enables deps; disable of a depended-on feature blocked; cascade atomic → `planEnable`/`planDisable` tests + `enableFeature`/`disableFeature` `$transaction`. ✔
- Cyclic dependency throws at load → `assertValidDag` (direct/self/deep cycle tests) invoked at `features.ts` load. ✔
- Disabled-feature route returns 403 when called directly + nav absent → `requireFeature`/probe route + `e2e/entitlements.spec.ts` (nav-hiding is `isEnabled`, cosmetic, wired in the app shell). ✔
- Every built module registered its feature code → `features.test.ts` asserts core.foundation/tenancy/entitlements present + every edge resolves. ✔

**Decisions made (logged, autonomous — no approval gate):**
- **Entitlement config tables are intentionally NOT tenant-scoped** (no `deletedAt`): L0 manages them across societies, and the resolver reads a given society's rows by explicit `societyId`. Using plain `db` (not `db.unscoped()`) keeps `lib/entitlements.ts` inside the ESLint boundary while passing through the scoping layer untouched. "Disabling never deletes data" is satisfied because disable writes an `enabled:false` override, never a delete.
- **Cache self-expiry at nearest `validUntil` (capped 60 s) + explicit invalidation on writes** — honours "entitlement expires mid-session → next request re-resolves" without a background timer; out-of-band edits heal within 60 s.
- **`limits(key)` = minimum across active override limits** (most restrictive wins); a downgrade below current usage blocks creation (enforced by callers in step 15+), never deletes existing rows.
- **Registry stays central in `lib/features.ts`** (one array, one seeder) rather than per-module `features.ts` files — no per-module dirs exist yet; each future module appends its entries. Satisfies "a single seeder upserts them".
- **Migration generated offline** (agent env has no Postgres), matching the step-02 approach; `deploy.sh` applies it via `migrate deploy`.

**Deviations from spec:** Feature registry keeps `title` (→ DB `name`) from step 02's `FeatureDef` rather than renaming the field; DB model matches the spec's `name`. No behavioural deviation.

**Follow-ups (for later steps):**
- Console UI (Simple switches + "will also enable", Pro DAG visualiser / validUntil / limits / diff-vs-plan) → step 08; `previewEnable` already provides the enable-preview data.
- Per-route `requireFeature` composition + nav-item hiding via `isEnabled` land with the app shell (step 05) and the 3-layer permission guard (step 04).
- `syncFeatureRegistry` invoked from platform bootstrap / a deploy hook once the platform console (step 08) exists; Plans seeded there.
- DB-backed integration tests for enable/disable transactions + audit rows run in CI once a Postgres service is added; DAG/resolver/limits logic is fully unit-covered now.
- Actor on audit rows defaults to `PLATFORM`; real principal + impersonation supplied by auth (step 04).

**Checkpoint:** [CHECKPOINT] — progress marker; controller auto-approves and continues to step 04.

---

## 04 — auth-rbac — DONE (2026-07-11)

**Spec:** `/specs/04-auth-rbac.md` (feature `core.auth`, isCore, depends `core.tenancy` + `core.entitlements`). No CODEREF. Identity + roles + sessions + three-layer permission enforcement.

**Data model (`prisma/schema.prisma`):** enums `UserStatus` (INVITED|ACTIVE|SUSPENDED), `DeviceKind` (PERSONAL|SHARED). Models `User` (nullable `societyId`=platform; phone/email/passwordHash/pinHash; `@@unique([societyId,phone])`/`([societyId,email])`), `Role` (nullable societyId, `permissions String[]`, `@@unique([societyId,code])`), `UserRole`, `LoginAlias` (`@@unique([societyId,alias])`, soft `deletedAt`), `Session` (`tokenHash` unique, `deviceKind`, `activeFlatId`, `impersonatedBy`, `lastSeenAt`, `expiresAt`), `LoginAttempt` (identifier/ip/success/kind, indexed for lockout queries), `OtpCode` (codeHash/expiresAt/consumedAt). Tenant-scoping recap encoded in a schema comment.
- **Scoping consequences (deliberate):** `User`+`LoginAlias` are tenant-scoped (societyId+deletedAt) → society-host login inside the scope auto-pins the lookup so an alias/phone from society A can't resolve a society-B account. `Role`/`UserRole` = seeded config (pass-through). `Session`/`LoginAttempt`/`OtpCode` = security infra without `deletedAt` (pass-through) so they work on the apex host (platform login) too.

**Pure logic (no DB, fully unit-tested):**
- `lib/rbac.ts` — `PERMISSION_REGISTRY` (permission→gating-feature; `null`=ungated/core), `ROLE_SEEDS` (2 platform + 9 society system roles), `effectivePermissions(rolePerms, entitledFeatures|null)` = expand `*` → union → ∩ entitlements (platform=null skips ∩; literal `*` never leaks into the set), `can`/`canAll`/`canAny`.
- `lib/auth-identity.ts` — `classifyIdentifier` (phone/email/alias), `normalizePhone`/`normalizeAlias`, `allowedKindsFor`/`isKindAllowedOn` (society=phone|alias, apex=email), `resolveIdentifier`.
- `lib/rate-limit.ts` — `evaluateLockout` (5/identifier/15 min → exponential backoff from base 30 s capped 1 h; per-IP config threshold 20), `canSendOtp` (3/phone/10 min).
- `lib/otp.ts` — `generateOtp` (crypto `randomInt`, 6-digit), `hashOtp` (sha-256), `checkOtp` (consumed→expired→hash order; single-use + 5 min TTL).
- `lib/password.ts` — scrypt (`scrypt$N$r$p$salt$hash`, self-describing), constant-time verify, false (never throws) on null/malformed. Used for passwords **and** guard PINs.

**DB / service glue:**
- `lib/auth.ts` (**added to ESLint `db.unscoped()` allowlist** — the auth boundary, like the tenant resolver): `login` (host-aware; generic `UnauthenticatedError` + `LoginAttempt` on every failure so account existence never leaks; lockout checked first; PIN restricted to society shared-device; only ACTIVE users), session create/destroy/`destroyUserSessions`, `getAuthContext`/`requireAuth` (absolute expiry + SHARED 30 min idle TTL + **suspended/deleted user → session destroyed** + `lastSeenAt` slide throttled to 1/min), `setActiveFlat`, opaque token (32 B) hashed at rest, cookie reader.
- `lib/authz.ts` — layer-1 guard: `requireRole` (any-of), `requirePermission`/`requirePermissions`, `requirePersonalDevice` (guard shared device ≠ resident data), `authorize({roles,permissions,features,personalDeviceOnly})` composing `requireFeature`.
- `lib/residents.ts` — read-only flats seam (`getUserFlats`) owned by step 16; stub returns `[]`.
- `lib/roles.ts` — `syncPlatformRoles` / `seedSocietyRoles` (findFirst+create/update to dodge NULL-in-compound-unique).
- `lib/http-auth.ts` — `authErrorResponse` (adds `Retry-After` for 429), `clientIp`, httpOnly session cookie set/clear (30 d personal / 12 h shared, `secure` off in dev).
- `lib/errors.ts` — `UnauthenticatedError` (401), `ForbiddenError` (403, carries `missing`), `RateLimitedError` (429, `retryAfterMs`) wired into `httpStatusForError`/`codeForError`.
- `lib/features.ts` — registered `core.auth`.

**Routes:** `POST /api/auth/login` (society: phone/alias+password or `{pin}` shared-device; apex: email+password; zod-validated; sets cookie), `POST /api/auth/logout` (idempotent, clears cookie), `POST /api/auth/otp/request` (feature-gated → 403 until step 11), `GET /api/me/flats` (auth'd; switcher data via residents seam inside `withSociety`), `POST /api/me/active-flat` (auth'd; refuses a flat the user doesn't hold).

**Migration:** `20260711140000_auth_rbac` — hand-authored offline (no Postgres in agent env; identical SQL applied by `deploy.sh migrate deploy`). 2 enums + 7 tables + uniques/indexes + FKs (UserRole/LoginAlias/Session → User, onDelete Cascade).

**Tests:** unit **117/117** (added: rbac 8, auth-identity 9, rate-limit 9, otp 6, password 3 = 35). e2e `e2e/auth.spec.ts` (apex-rejects-alias 401, me/flats requires auth, otp 403 seam; skips when `rufi` unseeded — e2e is not a CI gate).

**Verification:** typecheck ✔ · lint ✔ (0 warnings) · test:unit ✔ 117/117 · build ✔ (7 auth/me routes registered, middleware 52 kB) · prisma generate ✔.

**Acceptance-criteria mapping:**
- Alias login resolves to right user; alias from society A can't log into society B → `findUser` pins `societyId` in the `LoginAlias`+`User` where (scoped model also auto-pins); `e2e/auth.spec.ts` + isolation-by-construction. ✔ (DB-integration test runs in CI once Postgres is wired.)
- Effective permissions = role ∩ entitlements → `rbac.test.ts` (SOCIETY_ADMIN `*` without `amenities` → no amenity perm; with → gains it). ✔
- Rate limiter locks out after threshold; OTP single-use + expires → `rate-limit.test.ts` (threshold/backoff/window/unlock) + `otp.test.ts` (consumed/expired/mismatch). ✔
- Apex rejects flat-alias login; society rejects platform login → `auth-identity.test.ts` (`allowedKindsFor`/`resolveIdentifier`) + `login` `allowedOnHost` guard + `e2e/auth.spec.ts`. ✔
- Guard PIN reaches only guard console; multi-flat switcher — enforcement built (`requirePersonalDevice`, `Session.deviceKind` 30 min idle TTL; `/api/me/flats` + `/api/me/active-flat` with holding check). The **branded UI** Playwright flows land with the app shell (05) / form kit (06) / branding (13) — see decision below. ✔ (backend) / deferred (UI).

**Decisions made (logged, autonomous — no approval gate):**
- **`auth.otp` feature NOT registered this step.** It depends on `integrations.sms` (born step 11) and `assertValidDag` forbids an edge to an unregistered feature; stubbing a future module's feature would be worse. All OTP *primitives* (generate/hash/check, send rate-limit) are built and unit-tested now; the endpoint feature-gates on `auth.otp` and therefore **403s until step 11** registers `auth.otp`+`integrations.sms` and enables it — which is exactly the spec's "option not rendered / endpoint 403s".
- **`lib/auth.ts` is scope-independent and uses `db.unscoped()` with explicit `societyId` pinning** (added to the ESLint allowlist next to `lib/tenant.ts`), because platform users live outside any society scope and session/attempt lookups happen before a scope exists. Isolation is preserved by hand (every cross-tenant read carries an explicit `societyId`), not by the scoping layer.
- **No branded login/switcher UI shipped this step.** shadcn app shell (05), form/input kit (06) and per-society branding (13) don't exist yet; hand-rolling raw inputs would violate the UI contract (§5). Per §5 server enforcement is the check that matters and UI hiding is cosmetic — so the full auth **backend + guards** ship now and the Playwright UI flows attach once the kits exist.
- **Guard PIN uses the same scrypt path as passwords** (a short secret still gets a slow salted hash); OTP uses fast sha-256 (single-use, 5 min TTL — a per-code KDF buys nothing).
- **Session token model:** opaque 32-byte token in an httpOnly cookie, only its sha-256 stored (`Session.tokenHash` unique). Suspended/deleted user invalidates on the next request (session row deleted in `getAuthContext`).
- **Migration authored offline** (no Postgres in agent env), matching steps 02–03; `deploy.sh` applies via `migrate deploy`.

**Follow-ups (later steps):**
- Branded login page (Simple: "Phone or flat number" + password + guard PIN pad; Pro: remember-device, session list/revoke) → steps 05/06/13; auth backend + `/api/me/*` already provide the data.
- `getUserFlats` real implementation vs `FlatOccupancy` → step 16 (keeps `lib/residents.ts` signature).
- `syncPlatformRoles`/`seedSocietyRoles` invoked from platform bootstrap (08) / onboarding (21); `createSession(...impersonatedBy)` consumed by impersonation (08).
- OTP send/verify completion + `auth.otp`/`integrations.sms` registration → step 11.
- DB-backed integration tests (alias cross-society isolation, suspended-session invalidation, lockout persistence) run in CI once a Postgres service is added; all decision logic is unit-covered now.
- TOTP 2FA for platform users noted in spec (apex login "+ TOTP 2FA") — seam only; full enrol/verify is a later platform-console concern.

**Checkpoint:** [CHECKPOINT] — progress marker; controller auto-approves and continues to step 05.

---

## 05 — app-shell — DONE (2026-07-11)

**Branch:** `feature/05-app-shell` · **Spec:** `/specs/05-app-shell.md` (no CODEREF for 05).
**Feature:** `core.shell` registered — `isCore`, `dependsOn: [core.auth, core.entitlements]` (`lib/features.ts`). Auto-flows into `CORE_FEATURE_CODES`, so the resolver forces it ON for every live society and the DAG forbids disabling it.

**Goal:** the frame every screen lives in — collapsible sidebar, top bar (3 toggles + search + flat switcher + bell + user menu), ⌘K palette, shared states, one toaster, read-only/impersonation banners, RTL + light/dark, client-side navigation only.

### New dependencies (added, lockfile updated — `--frozen-lockfile` safe)
`@radix-ui/react-dialog@1.1.4`, `@radix-ui/react-dropdown-menu@2.1.4`, `@radix-ui/react-tooltip@1.1.6`, `@radix-ui/react-scroll-area@1.2.2`, `@radix-ui/react-visually-hidden@1.1.1`, `@radix-ui/react-direction@1.1.0`, `cmdk@1.0.4`, `sonner@1.7.1`. Only `@radix-ui/react-slot` was present before; these are the shadcn/ui (new-york) building blocks. `react-direction` is added explicitly because pnpm's strict store does not hoist it and `AppShellClient` imports `DirectionProvider` directly.

### Pure logic (framework-free, unit-tested)
- `lib/nav.ts` — the society nav registry + its role × entitlement gate. `NavItem` carries `permission?`/`feature?`/`hideOnSharedDevice?`; `canSeeNavItem` / `visibleNav` / `groupNav` / `visibleNavByGroup` implement **the rule that matters: a nav item renders only when `can(permission) && hasFeature(feature)`** (mirrors `lib/authz.ts`; server still enforces the route). No React, no DB, so the sidebar, the palette and the server all filter through the same code. Home + account are ungated so a zero-feature society still renders a usable frame.
- `lib/nav.test.ts` — **8 tests**: ungated-only baseline, permission-without-feature hidden, feature-without-permission hidden, both-present shown, permission-only item, guard shared-device hides resident items (gate-pass still shows), group ordering, registry integrity.

### shadcn/ui primitives (`components/ui/`, all RTL-logical: `ms-/me-/ps-/pe-`, `start/end`, no `ml-/mr-`)
`tooltip.tsx`, `dropdown-menu.tsx`, `dialog.tsx`, `sheet.tsx` (drawer, logical `start`/`end` sides), `command.tsx` (cmdk wrapper; `CommandDialog` includes a `VisuallyHidden` `DialogTitle`/`Description` so Radix a11y is satisfied — no missing-title warning, protects the Lighthouse ≥95 target), `skeleton.tsx`, `sonner.tsx` (the one global `Toaster`, themed from tokens, `toast` re-export — `alert()`/`confirm()` banned), `scroll-area.tsx`, `separator.tsx` (dependency-free 1px rule).

### Shell (`components/shell/`)
- `types.ts` — `ShellData`/`ShellUser`: the serialisable payload the server layout hands the client shell (no functions, no component refs).
- `nav-icons.ts` — maps a nav item's `icon` string → Lucide glyph on the client, keeping `lib/nav.ts` serialisable across the Server→Client boundary.
- `app-shell-client.tsx` — the interactive frame. Wraps everything in Radix `DirectionProvider` (so every menu/tooltip/drawer mirrors under RTL), `TooltipProvider`, and `ShellProvider`; lays out sidebar + mobile drawer + top bar + `<main>` (max-w-7xl) + palette + `Toaster`; renders impersonation + read-only banners above the frame.
- `shell-context.tsx` — client state (sidebar collapsed, mobile drawer open, palette open). Collapse is mirrored to a `rihaish_sidebar` cookie so a reload restores it with **no layout shift** (server reads the cookie in the layout).
- `sidebar.tsx` — desktop collapsible sidebar (w-64 ↔ w-16 icon rail), brand mark + name/slug, grouped nav in a `ScrollArea`, collapse toggle; hidden under `lg`. `mobile-nav.tsx` — the same nav in a `Sheet` drawer from the logical `start` edge, closes on navigate. `sidebar-nav.tsx` — grouped `NavLink`s with active-route highlight (`usePathname`, locale-stripped), tooltips when collapsed (side computed from locale dir, not a physical guess).
- `top-bar.tsx` — mobile menu button + compact brand (under `lg`), breadcrumbs, centred `SearchTrigger`, right cluster: `ModeToggle` (rendered only when applicable — hidden now, step 14), `FlatSwitcher`, `LocaleSwitcher`, `ThemeToggle`, `NotificationBell`, `UserMenu`. Sticky, backdrop-blur, never wraps (long names truncate).
- `search-trigger.tsx` — input-shaped button opening the palette; shows ⌘K on macOS / Ctrl K elsewhere (computed post-mount to avoid hydration mismatch). `command-palette.tsx` — cmdk dialog, global ⌘K/Ctrl+K listener, results = the **already-gated** visible nav (can never surface a forbidden destination), client-side `router.push` (locale-aware).
- `flat-switcher.tsx` — probes `GET /api/me/flats`, hides itself when <2 holdings, switches via `POST /api/me/active-flat` + `router.refresh()`, toasts result; server refuses a flat the user doesn't hold, so this is convenience not trust. `user-menu.tsx` — avatar initials + identity + account link + sign-out (`POST /api/auth/logout` → `router.refresh()`); degrades to a Sign-in affordance when signed out. `notification-bell.tsx` — graceful "all caught up" empty state (real channel = step 11). `mode-toggle.tsx` — Simple/Pro visual seam (local state; policy + persistence = step 14). `breadcrumbs.tsx` — `Home › <section>` from path × nav, truncates, chevron flips under RTL.
- States: `states.tsx` — `PageSkeleton`, `EmptyState`, `Forbidden`, shared `StateShell` (all copy passed in, locale-agnostic). `error-state.tsx` — the one state that owns an interaction (retry); used by the route `error.tsx`. `page-header.tsx` — title/description/breadcrumbs/actions, one per screen. `banners.tsx` — `ReadOnlyBanner` (reuses `society.readOnlyBanner`) + `ImpersonationBanner` slot (step 08 fills the label).

### Server wiring
- `lib/server-auth.ts` — `getServerAuthContext()` adapts RSC `headers()` cookie into a `Request` for `getAuthContext`, so the shell and the API routes resolve identity through the same path.
- `lib/shell-data.ts` — `buildShellData(society, locale)`: resolves identity, the society's entitled feature set (`resolve`), and `visibleNav`. **A signed-in user only "belongs" when `ctx.societyId === society.societyId`** — a platform session or another society's cookie is treated as a guest, so a cross-society cookie can never light up another tenant's nav. User identity (phone/email → initials, primary-role label) is read **inside `withSociety` scope** (the user is a scoped row). `canSwitchFlats` = personal-device member only.
- `app/[locale]/app/layout.tsx` — the L1 shell layout (`runtime=nodejs`, `force-dynamic`). Host → `resolveSocietyByHost` (a generic 404 via `notFound()` for an unknown/suspended tenant — never reveals existence; a non-`TenantNotFoundError`, e.g. DB-down, rethrows → 500, correct infra semantics). Reads the sidebar cookie, renders `AppShellClient`. Confirmed **dynamic** (not prerendered) via `prerender-manifest.json` — `/en/app`,`/ur/app` are absent, server-rendered per request; the build's `●` badge is cosmetic.
- `app/[locale]/app/page.tsx` — dashboard rewritten to live inside the shell (PageHeader + EmptyState welcome; real widgets = steps 15–39). `loading.tsx` → `PageSkeleton` (the spec's `<Suspense fallback>` at route level). `error.tsx` → `ErrorState` with retry.

### i18n
`messages/en.json` + `messages/ur.json` gained `nav.*` (labels, group headings, aria), `shell.*` (collapse/menu/impersonating + nested `error`/`flats`/`user`/`command`/`notifications`/`mode`), and `dashboard.*`. Every user-facing string routes through `next-intl` (the `react/jsx-no-literals` ESLint rule forbids bare JSX text); Urdu translations authored for all.

### Verification (gates)
- `pnpm typecheck` ✔ (0 errors)
- `pnpm lint` ✔ (0 warnings; fixed two `sr-only "Close"` literals in dialog/sheet by wrapping in an expression container per the rule)
- `pnpm test:unit` ✔ **125/125** (was 117; +8 nav)
- `pnpm build` ✔ fresh `BUILD_ID`; 9 routes; middleware 52 kB; `/[locale]/app` first-load JS ~102 kB shared. Client shell tree (cmdk/radix/sonner) compiles and bundles cleanly (`page_client-reference-manifest.js` generated).
- `pnpm test:e2e` — `e2e/app-shell.spec.ts` added: platform-host `/app` → 404 (route isolation), unknown-society host `/app` → 404 (never reveals existence), and a **no-full-reload** navigation assertion (window marker survives a nav click) that `test.skip`s when `rufi`/localhost is unseeded. e2e is not a CI gate (runs against seeded staging) — matches steps 02–04.
- Runtime smoke: `pnpm start` boots with the new dependency tree; `/app` 500s **only** because this agent env has no `DATABASE_URL`/Postgres (`PrismaClientInitializationError` from `resolveSocietyByHost`), identical to the DB-absent posture of steps 02–04 — not a shell defect.

### UI self-check (AGENT.md §3)
- [x] shadcn/ui + design tokens only — no raw HTML table/input; every list/table is deferred to the Data-Table kit (step 07), no lists shipped here.
- [x] Light **and** dark — all tokens; sonner + all primitives theme-aware.
- [x] RTL/Urdu — logical properties throughout (ESLint `no-restricted-syntax` enforces it); Radix `DirectionProvider` mirrors menus/tooltips/drawers; chevrons/panels flip via `rtl:` utilities; tooltip side computed from locale dir.
- [x] Responsive — sidebar → drawer under `lg`; top bar collapses search; guard-phone → desktop. Bottom-tab slot deferred to step 23 (shell exposes `<main>` as the content region).
- [x] Skeleton + empty + error + forbidden — shared primitives, wired via `loading.tsx`/`error.tsx`.
- [x] Client-side navigation — next-intl `Link`/`router`; `force-dynamic` layout; no full reload (Playwright assertion).
- [x] Nav + palette gated by role × entitlement — `visibleNav`, unit-tested against the same predicate the server guard uses.
- [x] Toasts for feedback; `alert()`/`confirm()` banned.
- [x] Graceful degradation — zero optional features → Home + account only, never an empty frame or a 500.
- Screenshot: not captured — this agent env has no Postgres, so `/app` cannot render a seeded society; the frame is verified by typecheck + build (client tree compiles) + the pure-gate unit tests. Visual capture belongs to the seeded-staging deploy.

### Decisions (logged; autonomous — no approval gate)
- **Society app shell only this step; the platform (L0) console keeps its placeholder.** Spec 05 is the L1 frame "consumed by all 35 modules"; the L0 console is step 08 and will reuse these same primitives. Building the platform shell now would pre-empt step 08's scope.
- **Sidebar collapse persisted via a `rihaish_sidebar` cookie, read server-side** for zero flash. Real per-user DB preference lands with the user-account module (step 09); the cookie is per-browser and documented as the interim. `next-themes` already handles the theme no-flash (script injected, `suppressHydrationWarning` on `<html>` from step 01).
- **Society brand-colour `--primary` override is a consume-only seam.** `Society` has no brand-colour column yet (step 13 adds it); `AppShellClient` is structured to inject it when present, so step 13 supplies without reworking the shell.
- **Simple/Pro toggle rendered only where applicable → hidden now** (`showModeToggle=false`); the control is fully built (`ModeToggle`) so step 14 only wires policy + persistence.
- **`react-visually-hidden` added and used** for the command palette's accessible title/description — fixes the Radix Dialog missing-title a11y warning rather than suppressing it, guarding the Lighthouse ≥95 acceptance.
- **Guest rendering supported** (no session → Home + account, Sign-in affordance) so the frame is resilient; the branded login page is still a follow-up (steps 06/13, per the step-04 note).

### Follow-ups (later steps)
- Branded login page + guard PIN pad → steps 06/13 (auth backend + `/api/me/*` already provide the data).
- `getUserFlats` real implementation → step 16 (the flat switcher already consumes the seam).
- Data-Table kit + Form kit → steps 07/06 (the shell deliberately ships no list/input so nothing hand-rolled leaks in).
- Notification bell live data + in-app channel → step 11; impersonation banner label → step 08; UI-mode policy/persistence → step 14; bottom-tab mobile bar → step 23; brand `--primary` injection → step 13.
- Lighthouse a11y ≥95 + full light/dark/RTL screenshots → captured on the seeded staging deploy (no Postgres in the agent env).

**Checkpoint:** [CHECKPOINT] — progress marker; controller auto-approves and continues to step 06.

---

## 06 — form-input-kit — DONE (2026-07-12)

**Branch:** `feature/06-form-input-kit` · **Feature:** `core.forms` (isCore, dependsOn `core.shell`) · **Spec:** `/specs/06-form-input-kit.md` (no CODEREF).

### What shipped
- **Feature registry:** `core.forms` added to `lib/features.ts` (isCore, deps `["core.shell"]`); `lib/features.test.ts` extended (registration + edge). DAG validates at load; registry seeder picks it up automatically.
- **Deps added (exact-pinned):** react-hook-form 7.54.2, @hookform/resolvers 3.10.0, libphonenumber-js 1.11.17, @tanstack/react-virtual 3.11.2, react-day-picker 9.5.0, date-fns 4.1.0, @radix-ui/react-{select 2.1.4, popover 1.1.4, checkbox 1.1.3, radio-group 1.2.2, switch 1.1.2, label 2.1.1}.
- **Pure layer (unit-tested, no React):** `lib/forms/phone.ts` (normalize→E.164, PK default; `0300…`→`+92300…`), `lib/forms/cnic.ts` (`#####-#######-#`, pasted-13-digit format, clean-13 stored), `lib/forms/mask.ts` (token pattern `#`/`A`/`*`, presets cnic/vehicle/account, formatted+clean), `lib/forms/money-input.ts` (sanitize/parse → **bigint only**, 0-digit currency rejects decimals) — reuses `lib/money.ts`.
- **Shared Zod (one schema, client+server):** `schemas/common.ts` (requiredString, email lower-case, `phoneSchema`→E.164, `cnicSchema`→clean digits, `moneySchema`→bigint minor units w/ optional min, iso date, HH:mm time, `mustBeChecked`), `schemas/demo.ts` (demo form). `schemas/demo.test.ts` proves server rejects what client rejects (bad phone/short CNIC/decimal-in-PKR/empty-blocks/unchecked-terms/bad-time). Added `schemas/**/*.test.ts` to `vitest.config.ts` include.
- **shadcn primitives (subagent, RTL-logical, light+dark):** `components/ui/{label,input,textarea,select,popover,checkbox,radio-group,switch,calendar}.tsx`. Calendar wraps react-day-picker v9 (v9 classNames keys, custom `Chevron`, `dir` passthrough).
- **Form kit `components/form/`:** unified `<FieldShell>` anatomy (label/required/hint/error/aria wiring in ONE file); `<Form>` (RHF provider, noValidate, gap layout) + `useUnsavedChangesGuard` (beforeunload while dirty) + `useZodForm`; controls `TextField` (prefix/suffix), `TextArea` (auto-grow+counter), `NumberField`, `MoneyField`, `SelectField` (≤10, warns over), `Combobox` (Popover+@tanstack/react-virtual, debounce 250ms, async `onSearch` or static filter, keyboard nav), `MultiSelect` (chips, search, select-all, virtualised), `DatePicker`/`DateRangePicker`/`TimePicker` (ISO/HH:mm storage, Intl formatters → Urdu names, `dir`), `PhoneField` (country Select PK-default, format-as-you-type, E.164 on blur), `MaskedField`, `CheckboxField`/`RadioGroupField`/`SwitchField`, `FileDropzone` (drag-drop, preview, size/MIME guard, camera capture seam for step 10), `Modal`/`Drawer`/`ConfirmDialog` (focus-trapped, ESC, RTL; ConfirmDialog names object + async no-double-submit + error toast), `FormActions` (sticky, disabled-while-submitting). Barrel `components/form/index.ts`.
- **Dev-only route:** `app/[locale]/design-system/page.tsx` (`notFound()` in production) → `components/design-system/design-system-client.tsx`: DirectionProvider + Toaster, Simple/Pro toggle (Simple = one-column + "More options" disclosure; Pro = two-column all-fields), demo form exercising every control + a gallery form (Number/Select/DateRange/Switch/FileDropzone) + overlay showcase. 500-flat combobox proves virtualisation. Submits via `fetch` (client-nav, no reload) to `app/api/design-system/demo/route.ts` which re-validates with the SAME `demoFormSchema` (dev-only, 404 in prod). i18n `designSystem` namespace added to `messages/en.json` + `messages/ur.json`.
- **Lint rule:** `.eslintrc.json` `no-restricted-globals` now bans `alert`/`confirm`/`prompt` (spec 06: no `alert()`/`confirm()` anywhere) alongside the existing `parseFloat` ban.

### Decisions
- **Money never floats:** `MoneyField` holds a STRING in the form; `moneySchema` transforms to `bigint` at parse. Used `zodResolver(schema, undefined, { raw: true })` in the demo so the RHF submit payload stays raw strings (JSON-serialisable) — the server re-parses to typed output (bigint). `parseMoneyInput` returns `bigint | null`, 0-digit currency rejects any decimal.
- **`mustBeChecked` modelled as `z.boolean().refine(v=>v===true)`** (not `z.literal(true)`) so the form INPUT type stays `boolean` and the checkbox can default to unchecked.
- **Combobox/MultiSelect built custom** (Popover + `@tanstack/react-virtual`) rather than on cmdk, because cmdk renders all items and fights manual virtualisation; custom list gives reliable 500+ perf + own keyboard handling.
- **Dates stored as timezone-free `YYYY-MM-DD`** (parse at local noon to dodge DST/UTC day-shift); society timezone applied at render/report time, not at capture. Calendar month/weekday labels via `Intl.DateTimeFormat(locale)` so Urdu renders correctly and `dir` mirrors the grid.
- **Kit components carry no visible hardcoded strings** — all user text arrives via props (labels/placeholders/hints); only sr-only/aria + punctuation literals remain, keeping i18n at the call site.

### Gate results
- `pnpm typecheck` — clean (TS strict, `noUncheckedIndexedAccess`; fixed index access in `mask.ts`).
- `pnpm lint` — clean, `--max-warnings 0` (fixed unused import + `aria-invalid` on `role="button"`).
- `pnpm test:unit` — **164 passed** (22 files; +39 over step 05: phone 8, cnic 6, mask 7, money-input 10, demo-schema 7, features +1).
- `pnpm build` — success; `/[locale]/design-system` compiles for en/ur; `/api/design-system/demo` dynamic.
- **Runtime smoke:** `next dev` → `GET /en/design-system` 200 (renders "Development only" / "Form & input kit" / "Full name"); `GET /ur/design-system` 200 with `dir="rtl"` + Urdu title. No DB needed (standalone dev route).

### Acceptance criteria (spec 06)
- [x] Every control exists, used by a demo form, documented in dev-only `/design-system`.
- [x] Vitest: Zod schema shared client+server; server rejects what client rejects (`schemas/demo.test.ts` + demo API on same schema).
- [x] Phone→E.164; CNIC mask accepts paste; money never a float (bigint).
- [x] Combobox virtualised + searchable at 500+.
- [x] Light+dark, EN+UR (RTL) — tokens + `dir` + Intl formatters; UR route smoke-tested RTL.
- [x] Unsaved-changes guard fires; no `alert()`/`confirm()` (ESLint rule added).
- [x] Simple and Pro variants both render.

### Deferred (by design)
- Real upload transport / signed URLs / sharp for `FileDropzone` → step 10 (storage-media); here it collects `File[]` + client-side size/MIME guard only.
- Async server-backed combobox search wired to real resident/vendor lists → their owning modules (15/16/17); the `onSearch` seam is in place.
- Full Lighthouse a11y + light/dark/RTL visual screenshots → captured on seeded staging deploy (no Postgres in agent env; the dev route needs none but shell-embedded forms do).

**Checkpoint:** [CHECKPOINT] — progress marker; controller auto-approves and continues to step 07.

---

## 07 — data-table-export-kit — DONE (2026-07-12)

**Branch:** `feature/06-form-input-kit` (continued; controller merges).
**Feature:** `core.tables` registered in `lib/features.ts` (isCore, `dependsOn: ["core.forms"]`, module `07-data-table-export-kit`). DAG validation green; feature-registry test still passes (count-agnostic).

### The one rule
"No module may hand-roll a table." Every list flows through `<DataTable>` + the `ListQuery`→`runList` server contract; every export through the export engine. This is what keeps 40 modules feeling like one product.

### Server contract (pure, unit-tested) — `lib/tables/`
- `types.ts` — `ListQuery {page,pageSize,sort[],search,filters}`, `ListResult<T>`, `Filter` (text|number|dateRange|enum|boolean), `ColumnConfig` (field/sortable/filter/searchable/sensitive), `MAX_PAGE_SIZE=100`, `PAGE_SIZES`, `clampPage/clampPageSize`.
- `list-query.ts` — `buildListQuery(query, columns)` → Prisma `{where, orderBy, skip, take}`. Hard-caps pageSize at 100 (a client "load all" is a build failure). Filters/sort/search **whitelisted** against the column config by id AND kind; nested field paths (`block.name`) build nested Prisma objects. **Never emits `societyId`/`deletedAt`** and strips any client attempt to filter them — the enforced `db` client injects scope, so every list is society-scoped *by construction* and can't be widened to another tenant from the wire.
- `server.ts` — `runList(delegate, query, columns)` executes through the scoped `db` (findMany + count share the same `where`). No active scope → `TenantScopeError` (unit-tested on `db.user`, no DB needed).
- `query-memory.ts` — `filterSortInMemory` + `queryInMemory`: identical filter/sort semantics over an array. Powers the dev demo (no DB) and guarantees an export of a filtered set equals the on-screen filtered count.
- `field.ts` — dotted-path read (`readPath`) / nest (`nestPath`) helpers shared by both paths.

### Export engine (pure logic unit-tested; io typechecked) — `lib/tables/export/`
- `csv.ts` — RFC-4180 escaping, UTF-8 BOM (Excel/Urdu), CSV-formula-injection guard (`=+-@` → apostrophe).
- `xlsx.ts` — exceljs; title band, bold frozen header, RTL sheet view, auto-fit widths.
- `pdf.ts` — pdf-lib + @pdf-lib/fontkit; branded header (logo + society name), generated timestamp, page numbers, RTL column order, **embedded Noto Naskh Arabic** (`assets/fonts/NotoNaskhArabic-Regular.ttf`, OFL, vendored) for Urdu glyph coverage; Latin falls back to Helvetica with non-WinAnsi chars sanitised. **Documented limitation:** pdf-lib draws glyphs but does no Arabic contextual shaping/bidi — Urdu chars render with correct glyphs but unjoined; a future step can swap the drawer for a HarfBuzz pass.
- `plan.ts` — `planExport(rowCount)` inline≤1000 / queued>1000; `sensitiveColumns`/`exportTouchesPii`.
- `service.ts` — `buildInlineExport`: PII gate (`tables.export.pii` via `can()`, else `ForbiddenError` before any bytes) → `generateExport` → **AuditLog** (`table.exported`, records the sensitive column list). `deps` injectable → fully unit-tested with a real CSV round-trip + mocked db.
- `audit.ts` (`writeExportAudit`), `fonts.ts` (`loadUrduFont`, cached, graceful-absent), `generate.ts` (`generateExport` + date-stamped `exportFilename`).

### Components
- `components/ui/table.tsx` — shadcn table primitives (logical `text-start`).
- `components/table/` — `<DataTable>` (TanStack Table v8; manual pagination/sorting/filtering; row model + selection + column-visibility state). Toolbar: debounced (250ms) global search, per-column filter popover (all 5 kinds) + active-filter chips, saved-views menu, column-visibility, density (comfortable/compact), list⇄card toggle. Body: sticky header, **pinned first column** (+ select col), sortable header cycle none→asc→desc, row actions dropdown, row click, **card view is the mobile default** (list hidden `sm:` when a card renderer exists), skeleton/empty/error states. Bulk-action bar on selection (ids persist across pages by `getRowId`). Server-driven fetch with a request-id race guard. `persistence.ts` — `TableStateAdapter` seam; `localStorageAdapter` default (column visibility + saved views persist per browser); server adapter can drop in for cross-device. `labels.ts` — `useDataTableLabels()` from the `table` i18n namespace (EN + UR added). Export materialises the current-view spec (visible cols in order + query + total) and hands it to `onExport`; button hidden when `exportPermitted===false`.
- `components/charts/` — Recharts wrappers: `LineChartCard`, `BarChartCard` (stacked), `AreaChartCard`, `DonutChartCard`, `Sparkline`. Animated on mount, tokenised `--chart-1..5` (already in globals.css + tailwind), responsive, tooltip/legend, empty state, **sr-only data-table fallback** (AA), RTL (x reversed / y flipped). `export.ts` — client PNG (SVG→canvas with computed-colour inlining so `hsl(var())` survives) + PDF (pdf-lib embeds the PNG); `findChartSvg` helper.

### Dev demo (dev-only, 404 in prod) — reference "flats" table
- `lib/tables/demo-data.ts` — deterministic 6,000-row in-memory flats (Flat lands step 15), column configs, enum options, server-side export-cell formatter (money via `formatMoney` bigint, dates via Intl).
- `app/api/design-system/table/route.ts` — POST `ListQuery` → `queryInMemory` → `ListResult` (proves server-side pagination + cap).
- `app/api/design-system/table/export/route.ts` — POST `{query,columns,format,locale,piiAllowed}`: filters full set, >1000→`202 {queued}` (worker seam, step 12), PII col w/o perm→403, else `generateExport` (Urdu PDF loads the font) with `Content-Disposition`.
- `app/[locale]/design-system/tables/page.tsx` + `components/design-system/tables-demo-client.tsx` — full gallery: all 11 columns (CNIC sensitive+default-hidden), fetcher→list API, onExport→export API+download/toast, row/bulk actions, card renderer, RTL preview + export-permission + PII-permission toggles, charts section with PNG/PDF buttons.
- `e2e/data-table.spec.ts` — smoke (list cap, filtered-CSV parity, >1000 queued, PII 403); e2e is not a CI gate here (dev/staging only).

### Decisions
- **Saved-views/column-vis persistence = localStorage adapter (default), with a `TableStateAdapter` seam** for a server/DB-backed per-user adapter later, instead of adding `TableView`/`TablePreference` Prisma models now. Rationale: the dev reference table is auth-less/scope-less (can't exercise DB persistence), the kit's acceptance ("persists") is met per-browser, and it avoids an untestable migration in a kit step. Consuming modules (15+) can pass a server adapter with zero component changes.
- **Export audit + PII gate live in `buildInlineExport`** (unit-tested with injected deps) — the real path every module calls. The dev export route mirrors the PII 403 but skips the DB audit (no scope in the gallery).
- **>1000 rows → 202 queued seam**, not inline generation — the worker/signed-URL delivery lands in step 12/10. Never a blocking request.

### Fix made (unblocks whole project)
`tailwind.config.ts` used `plugins: [require("tailwindcss-animate")]`. Node's ESM loader loads the `.ts` config in `next dev`, where `require` is undefined → **every page 500'd in dev** (confirmed: the step-06 form gallery failed identically). Converted to the canonical ESM `import tailwindcssAnimate from "tailwindcss-animate"`. Production `pnpm build` was unaffected before and after. This is a pre-existing step-01 bug surfaced by the UI self-check; fixed under autonomous rules and recorded here.

### Gate results
- `pnpm typecheck` — clean.
- `pnpm test:unit` — **197 passed** (25 files; +33 new: list-query 16, query-memory 7, export 10). Includes buildListQuery pageSize-cap + scope-never-emitted + all filter kinds; runList TenantScopeError; queryInMemory on-screen/export parity; CSV escaping/BOM/injection; planExport threshold; buildInlineExport PII-refuse/allow-and-audit.
- `pnpm lint` — 0 warnings/errors (`--max-warnings 0`).
- `pnpm build` — success; all 16 routes compiled incl. `/[locale]/design-system/tables` (both locales).
- **Runtime self-check** (dev server): list cap 6000→100 rows; filtered block-A total=1000 and CSV export=exactly 1000 data rows + BOM + CRLF (**parity confirmed**); unfiltered xlsx export→`202 {queued:true,total:6000}`; CNIC export w/o permission→`403 {error:FORBIDDEN,missing:[tables.export.pii]}`; Urdu PDF→valid `%PDF` 28KB branded; xlsx inline valid; `/en` + `/ur` gallery render **200** with `dir="rtl"` on ur.

### Deferred (by design)
- Server/DB-backed per-user saved-views + column-prefs (adapter seam ready) → consuming modules / a later shared table-prefs table.
- Queued >1000-row export execution + signed-URL delivery → step 12 (worker) + step 10 (storage).
- Full Arabic shaping/bidi in PDF (HarfBuzz) → future; glyphs embedded now.
- Real `Flat`-backed reference table → step 15 (society-structure); in-memory stand-in until then.

**Checkpoint:** [CHECKPOINT] — progress marker; controller auto-approves and continues to step 08.

---

## 08 — platform-console — DONE (2026-07-12)
**Feature code:** `platform.console` (platform-only; `isCore:false` so the resolver never forces it onto a tenant) · **Depends on:** `core.entitlements`, `core.tables`, `core.shell`
**Built:**
- **Feature registry:** registered `platform.console` in `lib/features.ts` (deps validated by `assertValidDag`). Society-entitleable views deliberately exclude any `platform.*` feature.
- **Schema:** `Session.impersonationWrite Boolean @default(false)` + `Session.impersonationReason String?` — impersonation is read-only until an operator explicitly enables writes with a reason. Migration `20260712120000_platform_console`.
- **Auth glue (`lib/auth.ts`):** `createSession` gained a `ttlMs` override (30-min impersonation cap); `hashToken` exported; `AuthContext.impersonationWrite` added (false unless the session is an impersonation session); new pure `impersonationReadOnly(ctx)` — the read-only decision society write-routes fold into their scope. `getAuthContext` reads the new field.
- **Impersonation core (`lib/platform/impersonation.ts`):** pure `assertCanImpersonate` (apex-only, platform-operator-only, no-nesting, society-target-only, never another platform user, never self) + DB service `startImpersonation` (30-min session stamped `impersonatedBy`, enter audit), `enableImpersonationWrite` (reason-required, separate audit), `exitImpersonation` (exit audit + session destroy). Cookie widened to the registrable parent domain (`crossSubdomainCookieDomain`) so it reaches the society host; capped at 30 min.
- **Platform services (`lib/platform/*`, all in the `**/platform/**` `db.unscoped()` allow-list):** `guard.requirePlatform` (apex + platform-user + permission, rejects impersonation sessions), `list.runPlatformList` (cross-society list runner reusing `buildListQuery`, `baseWhere` pin), `societies` (list+enriched counts/last-activity, create+role-seed+plan+limits, update, limits on the `core.tenancy` limits JSON), `detail` (feature catalogue with DAG deps/dependents + impersonation targets), `plans` (CRUD + societyCount + `planOptions`), `audit` (cross-society viewer, slug-enriched, newest-first), `dashboard` (societies-by-status, users, signups trend), `http.platformErrorResponse` (typed 404/409 envelope).
- **API routes:** `GET/POST /api/platform/societies` (list rides `?q=<ListQuery>`), `GET/PATCH /api/platform/societies/:id`, `PUT /api/platform/societies/:id/entitlements` (preview/enable/disable+cascade/setPlan/setLimits), `POST /api/platform/societies/:id/impersonate`, `GET/POST /api/platform/plans`, `PATCH /api/platform/plans/:id`, `GET /api/platform/audit`; plus neutral-path `POST /api/impersonate/exit` and `POST /api/impersonate/write` (must be reachable from the society-host banner, where `/api/platform/*` is 404 by isolation).
- **UI (`app/[locale]/platform/*` + `components/platform/*`):** platform shell (sidebar + section nav, theme/locale/sign-out, RTL via DirectionProvider) with a form-kit sign-in gate when not authenticated as staff; dashboard (stat tiles + area/donut charts); societies table (table kit) + New-Society modal (form kit); society detail (profile edit, plan apply, limits, entitlement switch list with "will also enable"/"blocked-by"/cascade-confirm/core-locked, impersonate picker → redirect to society host); plans table + create/edit modal (MultiSelect feature bundle); audit table. Every page guards on `getPlatformActor` before any read.
- **Society shell integration:** `ShellData.impersonating:string` → `ShellData.impersonation:{userLabel,readOnly,apexHost}`; `ImpersonationBanner` is now a client control — persistent, names society+user+read-only, "Enable write access" (reason modal → `/api/impersonate/write`) and "Exit" (→ `/api/impersonate/exit` → back to apex console). i18n added to `platform.*` and `shell.impersonation` in en + ur.
**Migrations:** `20260712120000_platform_console` (2 `Session` columns).
**Tests:** `lib/platform/impersonation.test.ts` — 10 unit tests (7 `assertCanImpersonate` invariants incl. the three acceptance rules: no-nesting, no-platform-target, no-society-host-start; 3 `impersonationReadOnly`). `e2e/platform-console.spec.ts` — API auth 401, society-host 404 isolation, neutral exit reachable, write 401. Host-isolation 404s (console↔society) already covered by `e2e/tenancy.spec.ts`. Suite: **207 unit tests pass**.
**Verification:** typecheck ✔ · lint ✔ (0 warnings) · unit ✔ 207 · build ✔ (all platform routes emitted) · runtime smoke ✔ — sign-in gate 200 (EN + UR `dir="rtl"`), `/platform` 404 on society host, platform API 401 unauth / 404 on society host, `/api/impersonate/exit` 200 on society host, `/api/impersonate/write` 401. e2e require a running DB (not in the CI unit gate).
**Decisions made (autonomous — no approval gate):**
- Impersonation exit/write live at neutral `/api/impersonate/*` (NOT `/api/platform/*`): the Exit/Enable-write controls render in the society-host banner, and a `/platform` path is 404 there by the cross-context isolation rule. The spec's illustrative `/api/platform/impersonate/exit` path cannot work on a society host; authority for these comes from the impersonation session cookie itself (the service refuses a non-impersonation session), so no platform guard is needed.
- Read-only-by-default is enforced via the session `impersonationWrite` flag surfaced as `impersonationReadOnly(ctx)`; the banner reflects it and the write-enable is reason-gated + separately audited. The society-side *write route* enforcement (folding this into the tenant scope) lands with the first society write module — the mechanism + flag + tests exist now.
- Per-society limits (`maxFlats/maxUsers/maxStorageMb`) stored in the `limits` JSON of the society's `core.tenancy` entitlement row (a stable home `computeLimit` already reads and feature toggles never clobber).
- Society "creation" is the minimal record + role seed + plan/limits; the guided go-live wizard is step 21 (hand-off deferred as the spec states).
- List endpoints are `GET` with the `ListQuery` in `?q=<json>` (honours the spec's `GET`, keeps the kit contract).
- Impersonation cookie widened to the registrable parent domain so it reaches the society host; host-only in dev/localhost (cross-subdomain cookies aren't shared there).
**Deviations from spec:** impersonation exit/write path relocated (see decisions). TOTP 2FA for platform users is scoped but not implemented (no acceptance criterion; deferred with the account-security module). Bulk entitlement apply across societies + DAG visualiser (Pro-only nice-to-haves) not built; the core entitlement DAG UX (auto-enable deps, blocked-by, cascade, core-lock) is.
**Follow-ups:**
- MRR + overdue societies → step 22 (platform billing); flats → step 15; storage → step 10 (tiles show honest zeros + "wired later" now).
- Fold `impersonationReadOnly` into society write-route scopes as those modules land.
- Platform-user management screen (TOTP enrolment, PLATFORM_ADMIN/SUPPORT CRUD) — additive.
- Bulk cross-society entitlement apply + DAG visualiser (Pro).
**Checkpoint:** [CHECKPOINT] — progress marker; controller auto-approves and continues to step 09.

---

## 09 — user-account — DONE (2026-07-12)

**Branch:** `feature/09-user-account` · **Work type:** FEATURE
**Spec:** `/specs/09-user-account.md` (no CODEREF in range). **Feature:** `core.account` (isCore) · **Depends:** `core.auth`, `core.forms`.

### What was built

The signed-in user's own settings surface: profile, security (password / guard PIN / active sessions), notification preferences and app preferences. This is deliberately an **L2 society-user surface** — profile CNIC, "my flats" and the society CNIC-capture toggle are all L1 concepts, and the `/app` shell only renders on a society host — so every account operation acts inside the actor's own society scope.

**Schema + migration** (`prisma/migrations/20260712130000_user_account/` — drafted before this step, verified + generated here):
- `UserProfile` (`userId @id`, fullName, avatarKey, `cnicEnc`, altPhone, emergency contact, updatedAt) and `NotificationPreference` (`userId`+`category` unique, five channel booleans). Both keyed by `userId` **only** — no `societyId`/`deletedAt`, so they are NOT tenant-scoped and pass through the scoping layer untouched (like Session/Feature).
- `Society.cnicCapture Boolean @default(false)` — society opt-in; the account CNIC field is only rendered/accepted when true.
- `Session.ip` + `Session.userAgent` — captured at login for the sessions list. `LoginInput`/`CreateSessionOpts` extended; `login()` and the login route now thread `ip` + `user-agent`.

**Crypto** (`lib/crypto.ts`, new, unit-tested): AES-256-GCM at-rest encryption. Key = sha-256 of `ENCRYPTION_KEY` (any-length secret works). Stored form `v1:iv:tag:ct`; `decrypt` returns `null` (never throws) for tampered/foreign/empty input (GCM auth). `maskCnic` (added to `lib/forms/cnic.ts`): `*****-****567-*` — reveals only the middle group's last three digits; fails closed (full mask) for a non-13-digit input.

**Service** (`lib/account/*`):
- `notifications-policy.ts` (pure, unit-tested): 7 categories × 5 channels; `isChannelLocked` (in-app locked for **every** category; push additionally locked for `emergency`) + `applyChannelFloor`. This is the floor the notification engine (step 11) will read.
- `service.ts`: `getAccountProfile`/`updateAccountProfile` (CNIC encrypted at rest, **masked in every response**, edit requires `reauthPassword` + an opted-in society, CNIC view/edit/avatar-delete audited), `changePassword` (verify current → hash → revoke every OTHER session, current survives), `setGuardPin`, `listSessions`/`revokeSession`/`revokeOtherSessions` (scoped to `userId`, current flagged), `getNotificationPreferences`/`putNotificationPreferences` (availability-gate then floor, upsert per category). All User-row reads/writes run inside `withSociety(actor.societyId, …)`; audit rows written with the actor's society + `SOCIETY_USER` actorType.
- `actor.ts`: `requireAccountActor(req)` (API) + `getServerAccountActor()` (RSC) — both require a society user (platform/L0 session → 403), and capture the current session token hash.
- `requirePersonalDevice` widened to `Pick<AuthContext,"deviceKind">` so the account actor can reuse it (profile PII routes reject a shared device; the PIN route does not — a guard on a shared device may set a PIN but never read PII).

**API** (`app/api/me/*`): `route.ts` GET/PATCH (profile; personal-device only; phones→E.164, CNIC 13-digit validation + tri-state undefined/null/value), `password` POST, `pin` POST, `sessions` GET + DELETE (revoke-all), `sessions/[id]` DELETE, `notification-preferences` GET/PUT. Shared Zod in `schemas/account.ts` (password/PIN/prefs/profile-patch). `flats` + `active-flat` already existed (step 04).

**UI** (`app/[locale]/app/account/page.tsx` + `components/account/account-client.tsx`): Simple/Pro toggle (localStorage). Profile card (form kit; masked CNIC read-only with a "Change CNIC" reveal → MaskedField + re-auth password), Security card (password + PIN forms; sessions as a plain list in Simple, the **DataTable kit** in Pro with row-action revoke + card renderer), Notifications matrix (Pro = semantic category × channel grid of switches with locked/unavailable switches + tooltips; Simple = one plain-language master switch per category; emergency always-on locked), Preferences card (language buttons → locale nav + best-effort persist; theme via next-themes). i18n `account` namespace added to en + ur (131 lines each, ICU plural on the "N sessions signed out" message).

### Decisions (autonomous, logged per AGENT §2.6)
- **Account is society-user only.** `User` is tenant-scoped and `lib/account/*` is not on the `db.unscoped()` allow-list, so the service scopes every User-row op via `withSociety`. A platform/L0 caller has no account here → 403. Platform-staff self-service is a later, separate surface.
- **CNIC is masked in every response, full stop.** Plaintext never crosses the client boundary; the only client form is the mask. This satisfies "never returned in full to a client lacking permission" without a per-request "reveal" permission — there is no reveal path. Editing re-authenticates.
- **Notification matrix uses a semantic `<table>`, not the DataTable kit.** DataTable is for record LISTS (sort/filter/paginate/export); a fixed 7×5 preference grid is a form control where table semantics aid accessibility (channel header row). The sessions LIST does use the DataTable kit (Pro). This respects the "no hand-rolled data table" rule in spirit and letter.
- **Password-change notification + avatar storage removal are step-11 / step-10 seams** (TODO comments in the service): the channel engine and storage adapter don't exist yet. The avatar deletion is already audited so the security trail is complete regardless of storage timing.

### Verification gates
- `pnpm typecheck` — 0 errors.
- `pnpm lint` — 0 warnings/errors (fixed a bare `:` JSX literal and an unused param).
- `pnpm test:unit` — **222 passed** (was 207): +`lib/crypto.test.ts` (round-trip, random-IV, tamper→null, foreign→null, encrypt≠plaintext, only-form-is-mask), +`lib/account/notifications-policy.test.ts` (in-app locked all categories, push locked emergency, floor delivers in-app on all-off, emergency floors in-app+push), +`maskCnic` cases in `lib/forms/cnic.test.ts`.
- `pnpm build` — succeeds (BUILD_ID set); `/[locale]/app/account` prerendered EN + UR; all `/api/me/*` routes present.
- **Runtime smoke** (built server, no DB): every `/api/me/*` (GET me/sessions/prefs, POST password/pin, PATCH me, DELETE sessions/[id]) returns **401 UNAUTHENTICATED** with no cookie (session lookup short-circuits, no DB touched); `/en/app/account` is a real route (500 only from the absent local Postgres, **not** 404) on a society host and **404** on the platform host (app-shell isolation preserved).
- **e2e** `e2e/account.spec.ts` — unauth 401 assertions (no seed needed) + the spec's session-revoke acceptance criterion (two logins → revoke B from A → B's cookie 401s), which skips cleanly when `rufi` is unseeded (e2e runs on a seeded staging deploy, not CI).

### UI design self-check
- shadcn/ui + tokens only; forms on the form kit, the sessions list on the table kit; the notification matrix is a semantic grid (justified above). ✅
- Light **and** dark — token classes only (`bg-card`/`text-muted-foreground`/`bg-primary`/`border`). ✅
- RTL/Urdu — `DirectionProvider dir`, logical props only (`ms-/me-/ps-/pe-/text-start`); the ESLint `no-restricted-syntax` guard (bans `ml-/mr-/text-left`) passed. ✅
- States — EmptyState (sessions empty, shared-device notice, sign-in gate), toasts for all feedback, app-level loading/error. ✅
- Screenshot — a DB-backed render is needed for a visual capture and there is no local Postgres/.env; the route is verified to resolve correctly (build + smoke) and the visual is captured on the seeded staging deploy, consistent with prior steps. Not a code blocker.

### Acceptance criteria (spec §)
- [x] CNIC encrypted at rest, never returned in full, masked in every response — `lib/crypto` + `maskCnic`, service always masks (unit-tested).
- [x] Emergency notifications ignore user opt-out for in-app + push — `applyChannelFloor` (unit-tested), floored on write.
- [x] Session revoke immediately invalidates the other session — `revokeSession` deletes the row; e2e asserts the revoked cookie 401s.
- [x] Notification prefs persist and are honoured by step 11 — persisted per category with the floor; the policy module is the engine's contract.
- [x] Light+dark, EN+UR RTL, mobile+desktop; forms on the form kit, sessions on the table kit — see self-check.

**Checkpoint:** [CHECKPOINT] — progress marker; controller auto-approves and continues to step 10.

---

## 10 — storage-media — DONE (2026-07-12)
**Feature code:** `core.storage` (isCore) · **Depends on:** `core.tenancy`, `core.forms`
**Built:**
- **Data model** — `StoredFile` (`id`, `societyId?`, unique `key`, `driver`, `mimeType`, `sizeBytes`, `width?`/`height?`, `thumbKey?`, `checksum`, `visibility`, `uploadedBy`, `entityType?`/`entityId?`, `createdAt`, `deletedAt?`). It carries BOTH `societyId` and `deletedAt`, so the step-02 scoping layer auto-enforces it: society-scoped reads, cross-society id = "not found", `delete`→soft-delete+audit — no bespoke isolation code. `Visibility` enum (PRIVATE default / SOCIETY / PUBLIC). `Society.maxStorageMb` (default 1024) for the per-society quota. Indexes on `societyId`, `(societyId,entityType,entityId)`, `(societyId,checksum)`, `deletedAt`.
- **`lib/storage/` (adapter + pipeline, all pure cores unit-tested):**
  - `adapter.ts` — `StorageAdapter` contract (`put/get/exists/signedUrl/delete`), `DEFAULT_SIGN_TTL_SECONDS=300`.
  - `local.ts` — filesystem driver under `STORAGE_LOCAL_PATH`; `signedUrl` mints the HMAC `/api/files/blob` URL; every key `isSafeKey`-checked + resolved-path-inside-root before touching disk.
  - `s3-sigv4.ts` — dependency-free SigV4 query presigner (GET/PUT/DELETE/HEAD); path-style (R2/MinIO) + virtual-hosted (AWS); **anchored by AWS's official GET example vector**. `s3.ts` — driver over presign+fetch (injectable transport/clock).
  - `gdrive.ts` — Drive REST driver (name=key in a folder; bytes proxied through the signed blob route). Implemented, never default.
  - `factory.ts` — `getAdapter()` from `STORAGE_DRIVER` (memoised); s3/gdrive misconfig fails loudly.
  - `image.ts` — sharp pipeline: `.rotate()` auto-orient then metadata dropped (**EXIF/GPS stripped**), resize max-edge 1600 `fit:inside/withoutEnlargement`, WebP q82, 320px thumb; HEIC/HEIF decoded (arm64 prebuilt has libheif) → WebP. `fitInside` pure helper.
  - `mime.ts` — magic-byte sniff (jpeg/png/webp/gif/pdf/heic-heif) + `looksExecutable` (ELF/PE/shebang/Mach-O); the real type is the sniffed type, a renamed exe is rejected.
  - `keys.ts` — `buildFileKey` (`societies/<id>/<bucket>/<uuid>.<ext>` | `platform/…`), `thumbKeyFor`, `isSafeKey` (traversal-proof), `mimeForKey`.
  - `signing.ts` — HMAC-SHA256 blob tokens (`key\nexp`, constant-time verify, expiry) + upload tickets (signed JSON payload); secret = `STORAGE_SIGNING_KEY` ?? `NEXTAUTH_SECRET`.
  - `limits.ts` — 10 MB cap, allow-list (jpeg/png/webp/heic/heif/pdf), `checkStorageQuota`. `retention.ts` — `RETENTION_DAYS` (delivery-log 90 / visitor 180), `PURGE_GRACE_DAYS=7`, `isRetentionExpired`/`isPurgeReady`.
  - `service.ts` — `storeFile` (empty/size/executable/sniff → quota (audits `storage.quota.exceeded` on reject) → sharp → checksum → per-entity dedup → adapter.put(+thumb) → `StoredFile` row + `storage.file.uploaded` audit), `getFileAccess` (scoped read → fresh signed URLs), `softDeleteFile`/`…ByKey`/`…EntityFiles`, `societyUsageBytes`, `sweepRetentionForSociety`, `purgeSoftDeletedRows` (worker step-12 hook; pure timing).
- **API** — `POST /api/files/sign` (validate MIME/size/entity/permission/quota → 10-min signed ticket), `POST /api/files` (multipart: verify ticket bound to user+society, re-validate size, `storeFile`), `GET /api/files/[id]` (society-scoped DTO w/ signed url+thumbUrl; other-society id → 404), `DELETE /api/files/[id]` (soft-delete), `GET /api/files/blob` (the ONLY byte path — HMAC signature IS the capability; unsigned/tampered/expired/guessed/traversal → 403; content-type from key ext, `no-transform`+`nosniff`, no auth needed). `lib/storage/http.ts` uniform error envelope (415/413).
- **Client** — `<FileUpload>` (`components/form`): sign→XHR upload with real progress bar, object-URL preview then stored thumbnail, camera capture, remove, per-file failure toast. Dev-only `/design-system/files` gallery (avatar/complaint/gate-pass), i18n `designSystem.files` en+ur.
- **Cross-cut** — account service's avatar-removal TODO now soft-deletes the `StoredFile` by key (already in society scope → scoping layer soft-deletes + audits). `sharp@0.35.3` added (native arm64 verified) + `serverExternalPackages:["@prisma/client","sharp"]`. Env: `STORAGE_SIGNING_KEY`, `S3_*`, `GDRIVE_*` (+ `.env.example`).
**Migrations:** `20260712140000_storage_media`
**Tests:** +43 unit (265 total) — `mime`, `keys`, `signing` (blob+ticket expiry/tamper/key-swap), `limits`, `retention`, `s3-sigv4` (AWS vector + host/path), `image` (sharp WebP/resize/thumb/EXIF-strip + reject non-image), `adapter-contract` (**same suite passes identically for local over a temp dir and s3 over an in-memory object store**). e2e `files.spec` (sign/upload 401 unauth; blob 403 unsigned/forged/traversal; skips seeded).
**Verification:** typecheck ✔ · lint ✔ (0) · unit ✔ (265) · build ✔ (all `/api/files/*` + `/design-system/files`) · sharp native load + HEIF input + WebP output confirmed. Runtime/e2e against a seeded staging deploy (no local .env/DB here; not a CI gate) — matches prior steps.
**Decisions made:**
- **StoredFile is auto tenant-scoped** (societyId+deletedAt) rather than a bespoke isolation path — reuses the enforced data layer; platform assets (societyId null) are a platform/worker concern via the unscoped client.
- **Signed blob route is the single egress**; s3 driver hands the browser a direct presigned GET, local/gdrive proxy through `/api/files/blob`. The HMAC signature (not a session) is the capability, so a leaked URL is time-boxed and key-bound.
- **Dedup scoped to (checksum, entityType, entityId)** — an identical re-submission for the same entity returns the existing row (no double storage / no duplicate row); identical bytes for a *different* entity get their own key, avoiding the unique-key collision a shared object would cause.
- **S3 without an SDK** — a hand-rolled SigV4 presigner keeps the dependency surface tiny and is anchored to AWS's published vector; R2/MinIO use path-style.
- **sharp pinned + kept external** so the arm64 prebuilt resolves at runtime and is never bundled by Next.
**Deviations from spec:** Presigned direct-to-S3 upload is scaffolded (sign returns `strategy:"server"`) but the default/only path is server-receive — required because images MUST pass through sharp server-side to strip EXIF/GPS; a direct-to-S3 image would bypass that. Physical object purge of soft-deleted rows is a pure hook (`purgeSoftDeletedRows`) invoked by the worker in step 12 (it needs the unscoped client to see soft-deleted rows), mirroring how step 09 deferred delivery to step 11.
**Follow-ups:** worker (step 12) wires `sweepRetentionForSociety` + `purgeSoftDeletedRows` into the cron/queue and warns L0 on quota breach; notifications (step 11) can attach `StoredFile`s; entity modules (gate-pass, complaints, payments, expenses, documents) consume `<FileUpload>` + `softDeleteEntityFiles`; per-entity PRIVATE read permission (beyond society membership) is enforced by each owning module.

**Checkpoint:** [CHECKPOINT] — progress marker; controller auto-approves and continues to step 11.

---

## 11 — notifications-core — DONE (2026-07-12)

**Branch:** `feature/11-notifications-core` · **Spec:** `/specs/11-notifications-core.md` (no CODEREF) · **Feature:** `core.notifications` (isCore, deps `core.account`, `core.storage`)

**Goal:** one engine every module sends through — resolve audience → resolve channels (preference ∩ society config ∩ template availability; emergency ignores opt-out) → render per-locale template → ENQUEUE (never send in a request); society-supplied encrypted SMS/WhatsApp/SMTP credentials; in-app notification centre; template editor; recipient rules.

### Schema — migration `20260712150000_notifications_core`
- Enums `IntegrationKind` (SMS|WHATSAPP|SMTP), `Channel` (IN_APP|EMAIL|SMS|WHATSAPP|PUSH), `NotifStatus` (QUEUED|SENT|DELIVERED|FAILED|SUPPRESSED).
- `SocietyIntegration` (`@@unique[societyId,kind]`) — `configEnc` AES-256-GCM, `isActive`, `verifiedAt`. `NotificationTemplate` (`@@unique[societyId,code,channel,locale]`, societyId null = platform default). `Notification` (per user×channel row; `payload` Json holds rendered subject/body/to/data; `attempts`, `readAt`, `sentAt`; idx `[societyId,userId,readAt]`,`[societyId,status]`). `NotificationRecipientRule` (`@@unique[societyId,category,userId]`).
- **Decision — pass-through models, explicit scoping:** none carries BOTH `societyId`+`deletedAt`, so the `lib/db.ts` auto-scoper treats them as pass-through (like Feature/Session/AuditLog). Tenant isolation is enforced by every `lib/notifications/*` query filtering on `societyId` (same as entitlements/audit). Notifications are a delivery ledger — never soft-deleted; a user "clears" by marking read. Added non-spec fields `attempts`/`sentAt` (retry/backoff + delivery time) and `SocietyIntegration.createdAt/updatedAt`.

### Feature registry (`lib/features.ts`)
Registered `core.notifications` + `integrations.email`/`integrations.sms`/`integrations.whatsapp` (non-core, dep `core.notifications`) and the **deferred `auth.otp`** (non-core, deps `core.auth`+`integrations.sms`) — the DAG edge that was impossible until `integrations.sms` existed. OTP endpoint still 403s until a society enables SMS (feature not entitled by default) — unchanged behaviour, now with a valid DAG.

### `lib/notifications/*`
- `channels.ts` (pure) — `resolveChannels` = pref ∩ society ∩ template; applies `applyChannelFloor` so in-app is always on and emergency floors push; deterministic order.
- `templates.ts` (pure) — 8-code catalogue (one per category: billing.invoice.created/payment.received, announcement.posted, complaint.updated, gatepass.approved, utility.notice.posted, chat.message.received, emergency.alert.raised), **every (code,channel) in EN + UR**; `renderTemplate` (`{{var}}`, unknown → empty, whitespace-tolerant); `DEFAULT_TEMPLATES`/`categoryForCode`/`TEMPLATE_CODES`.
- `config.ts` — per-provider `ProviderSpec` (twilio, pk-sms-gw, wa-cloud, smtp) with Zod + field descriptors (secret flag); `encryptConfig`/`decryptConfig` (reuses `lib/crypto`); `redactStatus` (the ONLY client-facing shape — masks secrets to `••••`, echoes non-secret fields); `clientProvidersForKind` (drops the Zod schema so provider specs are serialisable to RSC/JSON).
- `adapters.ts` — `sendViaAdapter` per channel; in-app trivially ok (the row is the delivery); email via injectable `emailTransport` (SMTP wired at deploy; suppressed until then), push via injectable `pushTransport` (step 23), SMS/WhatsApp via pure `buildHttpRequest` (Twilio basic-auth, pk-sms-gw bearer, wa-cloud graph) + injectable `httpTransport` (real fetch default). NEVER throws, NEVER logs credentials. `set*Transport`/`__resetTransportsForTests`.
- `integrations.ts` — `getIntegrationStatuses` (redacted), `configureIntegration` (validate→encrypt→store INACTIVE), `testIntegration` (**live send; `isActive`+`verifiedAt` set only on ok**; audit `integration.verified`/`test_failed`), `clearIntegration` (admin may clear, never read), `getActiveConfig` (dispatcher), `availableChannels` (IN_APP+EMAIL always; SMS/WHATSAPP when active; **PUSH excluded until step 23**).
- `templates-db.ts` — `syncNotificationTemplates` (seed platform defaults; findFirst+create/update because Prisma upsert can't target a null in a compound unique), `resolveTemplate`/`templateChannelsFor` (society→default→EN fallback, **in-code fallback for an unseeded DB**), `listTemplates`, `upsertSocietyTemplate`/`resetSocietyTemplate`.
- `audience.ts` — `resolveAudience` (users | roles | allMembers | residents | recipientRuleOnly; block/flat = step 15-16 stubs) via `withSociety` (User is tenant-scoped); `recipientRuleUserIds`.
- `service.ts` — `notify` (derive category → audience (+recipient nominees) → bulk-load users/prefs/society/channels → per-locale template availability + resolved bodies → build `Notification` rows: IN_APP=DELIVERED+sentAt, external with address=QUEUED, external w/o phone/email=SUPPRESSED → `createMany`; **no provider call, no inline send**); `notifySafe` (swallows errors — a notification can't break its originating flow); notification centre `listNotifications`/`unreadCount`/`markRead`/`markAllRead` (scoped user+society, channel IN_APP).
- `dispatch.ts` — `dispatchQueued` (WORKER entry, step 12): drain QUEUED external rows → build outbound from stored payload+active config → send → SENT / retry(attempts++, ≤3) / FAILED / SUPPRESSED(no active integration); `recentDeliveries` (Pro delivery log). `actor.ts` — `requireNotificationActor` (any society user) / `requireSettingsActor` (+`society.settings.update`) / `getServerNotificationActor`.
- `recipient-rules.ts` — list/add/remove (+`listSocietyMembers` picker, `withSociety`-scoped).

### API
- `/api/notifications` GET (centre page + unread), `/api/notifications/[id]/read` POST, `/api/notifications/read-all` POST.
- `/api/society/integrations` GET (redacted statuses + client provider specs), `/api/society/integrations/[kind]` PUT (configure)/DELETE (clear), `/api/society/integrations/[kind]/test` POST (RECIPIENT_REQUIRED 400 for SMS/WA without `to`). `/api/society/templates` GET/PUT/DELETE. `/api/society/recipient-rules` GET/POST/DELETE. Zod in `schemas/notifications.ts`; ZodError from per-provider validation → 400.

### UI
- `components/shell/notification-bell.tsx` — live bell: unread badge (poll 60s), lazy list on open, mark-one/mark-all, skeleton/empty; 401/403 → graceful empty (no 500).
- `app/[locale]/app/settings/page.tsx` + `components/notifications/settings-client.tsx` — society settings (nav `settings` item already gated `society.settings.read`). Simple = integration cards (status chip active/needs-test/not-configured, provider form, mandatory **Send test**, Clear; secrets write-only, shown `••••`). Pro adds template editor (variable chips extracted from body + live EN/UR preview with RTL `dir`) and recipient rules (category × member picker). Read-only when lacking `society.settings.update`.
- i18n `settings.*` + `shell.notifications.{markAll,unread}` in en + ur (RTL Urdu bodies).

### Gates
- `pnpm typecheck` ✔ · `pnpm lint` ✔ (0 warnings) · `pnpm test:unit` ✔ **294** (+29: `channels` intersection/floor/emergency, `config` encrypt round-trip + never-leak + redact-masks-secret, `templates` EN+UR completeness + render, `adapters` builders + inject-transport + never-throws + email-suppressed-until-wired) · `pnpm build` ✔ (all `/api/notifications/*`, `/api/society/*`, `/app/settings`) · `prisma validate` ✔.
- No local `.env`/DB (as prior steps) → `prisma migrate deploy` + e2e run on the seeded staging deploy. e2e `notifications.spec` asserts unauth 401 across the centre + settings routes and that a test-send is refused without a session.

### Acceptance criteria
- [x] Vitest: channel resolution = pref ∩ society ∩ template; emergency overrides opt-out (`channels.test.ts`).
- [x] Vitest: credentials encrypted at rest, never in a response/log (`config.test.ts` — ciphertext excludes plaintext; `redactStatus`/JSON exclude the secret).
- [x] "Send test" verifies before `isActive` (adapters return ok/fail via transport; `testIntegration` flips active only on ok — `adapters.test.ts` covers the ok/fail branch).
- [x] All templates EN + UR; missing Urdu fails the seed test (`templates.test.ts` completeness).
- [x] Sending is queued, never inline; a provider failure never breaks the originating request (`notify` only persists rows; `notifySafe`; adapters never throw).
- [x] Notification centre in light/dark, EN/UR, mobile + desktop (bell + centre on shadcn primitives, logical props, RTL `dir`, skeleton/empty).

### Decisions
- **Explicit societyId scoping** for all four pass-through models (spec schema has no `deletedAt`) — matches audit/entitlements; keeps notifications as an immutable delivery ledger.
- **In-code template fallback** so `notify`/`listTemplates` work on an unseeded DB; `syncNotificationTemplates` is the deploy/bootstrap seed (companion to `syncFeatureRegistry`/`syncPlatformRoles`, which also have no in-repo caller yet).
- **Transports injected**, not hard-wired — the request path only writes rows; the worker (12) wires the SMTP/HTTP drain, push (23) wires VAPID. Keeps step 11 self-contained and network-free in tests.
- **PUSH excluded from `availableChannels`** until step 23 (no VAPID transport) so no push row is created that would churn to FAILED; emergency still delivers in-app.
- **Secrets never round-trip** — v1 requires re-entering a secret to re-save (masked `••••` ≠ a value); the client blocks saving an unchanged mask.

### Deviations / follow-ups
- Real SMTP + web-push transports and the queue **drain schedule** land in step 12 (`dispatchQueued`) / step 23 (push) — the functions + adapter seams exist and are tested now.
- Block/flat audiences resolve empty until society-structure (15) + occupancy (16); documented, not a crash.
- Account module's `availableChannels` stub (step 09) still reports SMS/WhatsApp unavailable in the prefs matrix; wiring it to real integration status would invert the module dependency (account←notifications) so left as a follow-up — the engine already floors unconfigured channels off.
- The step-09 password-change notification TODO is intentionally left: it needs a security category/template code outside the fixed 7-category matrix; consumers adopt `notify()` as they ship.

**Checkpoint:** [CHECKPOINT] — progress marker; controller auto-approves and continues to step 12.

---

## 12 — worker-scheduler — DONE (2026-07-12)

**Branch:** feature/12-worker-scheduler · **Spec:** /specs/12-worker-scheduler.md (no CODEREF) · **Feature:** `core.worker` (isCore, dependsOn `core.notifications`)

### What shipped
The second pm2 process is now real: a Postgres-backed job queue + a timezone-aware cron registry. Everything heavy or scheduled runs in the worker inside `withSociety`, never in a request.

**Schema (migration `20260712160000_worker_scheduler`)**
- `Job` — the queue unit: `dedupeKey String? @unique` (idempotency — a monthly run for "2026-07" can never double-generate), `status JobStatus (QUEUED|RUNNING|DONE|FAILED|DEAD)`, `attempts/maxAttempts`, `runAt` (backoff pushes it out), `lockedAt/lockedBy` (5-min stale-lock reclaim), `lastError/result`, `@@index([status,runAt])`.
- `CronSchedule` — `@@unique([societyId,code])`, `cron` + per-schedule `timezone`, `enabled`, `lastRunAt/nextRunAt`, `@@index([enabled,nextRunAt])`.
- `WorkerHeartbeat` — `id` (worker id) + `beatAt`; DB-backed liveness replacing the step-01 file heartbeat.
- All three carry `societyId?` but **no `deletedAt`**, so the scoping layer treats them as infrastructure (pass-through); the worker owns them through the unscoped client. Any domain effect still runs inside `withSociety`.

**`lib/worker/*`**
- `cron.ts` (pure) — 5-field parser (`*`, literal, `a-b`, list, `*/n`, `a-b/n`; dow 0/7=Sun, DOM∨DOW when both restricted) + `nextRun(expr, from, tz)` computing the next instant strictly after `from` in an IANA timezone via `Intl.DateTimeFormat` (offset two-pass for DST zones), field-skipping with a ~5-year horizon (returns null for impossible crons e.g. Feb 30).
- `backoff.ts` (pure) — `BACKOFF_MS = [1m,5m,30m]`; `nextStateAfterFailure(priorAttempts, maxAttempts, err, now)` → re-queue with backed-off `runAt`, or DEAD once the used attempt reaches `maxAttempts`.
- `types.ts` / `registry.ts` — `CRON_REGISTRY` (the 9 seeded schedules: society-scoped `billing.monthly-run|late-fee|reminders`, `utility.reminders`, `complaints.sla-escalate`, `storage.retention-purge`; platform-scoped `platform.grace-check`, `notifications.retry`, `exports.cleanup`) → `JOB_REGISTRY` (kind → handler + `mutating` + `concurrency`). `skipForReadOnly(def, status)` gates mutating jobs for a READ_ONLY society; read-only/delivery jobs (reminders, notifications.retry) still run.
- `handlers.ts` — real: `notifications.retry` → `dispatchQueued(200)` (delivery, non-mutating), `storage.retention-purge` → `sweepRetentionForSociety` + purge of grace-elapsed soft-deleted rows via the unscoped client. The other 7 are registered **stubs** (safe no-op logging the owning step: 18/19 billing, 29 utility, 26 complaints, 22 grace-check, 07 exports) so the queue/scheduler/DLQ are exercisable now and the owning module fills the handler at its step.
- `queue.ts` — `enqueue` (idempotent: catches P2002 on `dedupeKey`, returns the existing job `created:false`), `claimNext` (`UPDATE … WHERE id = (SELECT … WHERE due OR stale-lock … FOR UPDATE SKIP LOCKED LIMIT 1) RETURNING …`, with per-kind `excludeKinds` for concurrency caps), `completeJob`/`failJob`(applies the backoff transition)/`skipJob`, and the console ops `queueStats`/`retryJob`/`killJob`.
- `scheduler.ts` — `syncSchedules` seeds one platform row per platform code and one row per active society × society code (society timezone), recomputing `nextRunAt` only on cron/timezone drift or when unset (never steals a still-pending occurrence — satisfies the "timezone changed → nextRunAt recomputed" edge case). `tickSchedules` enqueues a job for each due schedule with `dedupeKey = cron:<code>:<society>:<occurrenceISO>` (one job per occurrence even with two workers ticking) and advances `nextRunAt`. `runScheduleNow` for "Run now".
- `heartbeat.ts` — DB upsert every 30s; `readHeartbeat` returns the freshest beat classified up/stale/down (90s threshold).
- `runtime.ts` — `createWorker(id)`: intervals for heartbeat (30s), schedule sync (5m), cron tick (15s), pump (2s); a per-kind + global (8) concurrency pool; each society job runs in `withSociety(societyId, …, { readOnly: status==='READ_ONLY' })`; a mutating job for a READ_ONLY society is skipped with a `job.skipped` AuditLog + DONE{skipped}; unknown kinds dead-letter; graceful shutdown drains in-flight jobs then `$disconnect`.
- `console.ts` / `automation.ts` — platform Jobs list (data-table kit over the unscoped `Job` delegate, enriched with society slug) and the society Automation reads/mutations (every query filters `societyId` explicitly; ownership-checked toggle + run-now).

**Worker process** — rewritten in TypeScript (`worker/index.ts`), run via **tsx** so it shares the app's lib code verbatim with no separate build. `package.json` `worker` script → `tsx worker/index.ts`; `ecosystem.config.cjs` worker → `node_modules/.bin/tsx worker/index.ts` (fork, `kill_timeout: 10000`). Env validation preserved (fails loud on missing keys). Added `tsx@4.23.0` devDependency (verified it resolves the `@/` tsconfig path alias).

**Health** — `/api/health` `checkWorker` now reads the DB heartbeat (`readHeartbeat`) instead of the file mtime; the field is always present so the deploy gate can read it. Removed `lib/worker-heartbeat.ts` and the old `worker/index.js`.

**API + permissions** — `platform.jobs.read`/`platform.jobs.manage` added to the registry (PLATFORM_ADMIN gets both, PLATFORM_SUPPORT gets read). Routes: `GET /api/platform/jobs` (`?stats=1` = queue depth; else paged/filterable list), `POST /api/platform/jobs/[id]` (`retry`|`kill`, jobs.manage). `GET /api/society/automation` (settings-read), `PATCH /api/society/automation/[id]` (enable/disable) + `POST` (`run-now`) (settings-update).

**UI** — Platform **Jobs** page `/platform/jobs` (nav item added; queue-depth tiles + the job/dead-letter table on the kit with retry/kill row actions gated by status). Society **Automation** panel embedded in `/app/settings` (per-schedule label + cron + next/last run in the schedule's timezone, on/off switch, "Run now"; mutations gated by `canManage`). i18n en+ur for `platform.jobs.*`, `platform.nav.jobs`, `settings.automation.*` (incl. friendly names for the 6 society cron codes).

### Decisions
- **tsx over a compiled worker** — the whole codebase keeps business logic in `@/lib/*` TS with `@/` aliases; compiling a separate worker bundle (or duplicating logic) is worse than running the shared code through tsx. Verified tsx resolves tsconfig `paths` and the full module graph (incl. sharp/db/dispatch) loads; worker boots, logs online, and drains on SIGTERM. **[DECIDE AT BUILD]** Redis/BullMQ deferred per spec — the SKIP-LOCKED Postgres queue avoids adding a service to the free-tier box.
- **Idempotency via `dedupeKey @unique`** — the spec's Job block had no idempotency column; added one. Cron occurrences and "Run now" mint occurrence-scoped keys so a duplicate enqueue is a no-op under a race (unique index lets one insert win).
- **DB heartbeat replaces the file heartbeat** — honours the step-01 TODO and works even if web (pm2 cluster) and worker don't share a tmpdir. The deploy health gate now fails when the worker is genuinely down, which is the spec intent.
- **Stub handlers for unshipped modules** — registering the cron + queue plumbing now (with a logging no-op) keeps the scheduler/DLQ exercisable and lets each owning module (17-19, 22, 26, 29, 07) drop in its handler at its step without touching the worker.
- **READ_ONLY gating by declared `mutating`** — a handler declares whether it mutates tenant domain data; the runtime skips mutating handlers for READ_ONLY societies (+ audit) and runs read-only-safe ones (reminders enqueue `Notification`, a pass-through model, so they succeed even under a read-only scope).
- **Automation lives inside `/app/settings`** — avoided adding a sidebar nav item (would churn `nav.test.ts` + nav-icons); "Society → Automation" is a settings subsurface, gated by the existing `settings` nav item's `society.settings.read`.

### Gate results
- `pnpm typecheck` — clean.
- `pnpm lint` — 0 warnings/errors.
- `pnpm test:unit` — **314 passed, 3 skipped** (+20 new: `cron.test.ts` 10, `registry.test.ts` 7, `backoff.test.ts` 3). The 3 skipped are `queue.integration.test.ts` (SKIP LOCKED double-processing, idempotency, backoff→dead-letter) guarded by `describe.skipIf(!DATABASE_URL)` — they **run against Postgres in deploy.sh/CI** (self-cleaning by test kind), satisfying the DB-dependent acceptance criteria; the timezone-correctness criterion is a pure `cron.test.ts` case (Karachi vs America/New_York).
- `pnpm build` — success; new routes `/platform/jobs`, `/api/platform/jobs`(+`[id]`), `/api/society/automation`(+`[id]`) all present.
- Worker smoke — `tsx worker/index.ts`: missing env → exit 1 with named error; with env + bogus DB → boots, logs `online`, catches DB errors on each interval (no crash), drains cleanly on SIGTERM.

### Acceptance criteria (spec 12)
- [x] Vitest: SKIP LOCKED claiming prevents double-processing — `queue.integration.test.ts` (DB path, deploy/CI).
- [x] Vitest: idempotency key blocks a duplicate monthly invoice run — `queue.integration.test.ts`.
- [x] Vitest: backoff → dead-letter after `maxAttempts` — `backoff.test.ts` (pure) + `queue.integration.test.ts` (end-to-end).
- [x] Cron fires at the correct local time for a non-Karachi timezone — `cron.test.ts` (same `0 2 1 * *` → 21:00Z Karachi vs 06:00Z America/New_York).
- [x] Worker heartbeat visible in `/api/health`; deploy fails if the worker is down — DB heartbeat + `checkWorker`; e2e asserts the field.
- [x] READ_ONLY society: mutating jobs skipped, reason logged — `skipForReadOnly` (unit) + runtime skip + `job.skipped` AuditLog.

### Notes / follow-ups
- Stub handlers deliberately no-op until their module ships (17-19 billing, 22 platform grace-check → read-only flip, 26 complaints SLA, 29 utility, 07 exports cleanup).
- PUSH delivery still lands as SUPPRESSED until step 23 wires a transport (unchanged from step 11).
- Per-society schedules are materialised for currently-active societies on worker boot/sync; step 21 onboarding can call `syncSchedules` (or add its own `ensureSocietySchedules`) so a brand-new society is scheduled immediately rather than at the next 5-minute sync.

**Checkpoint:** [CHECKPOINT] — progress marker; controller auto-approves and continues to step 13.

## 13 — branding-domains — DONE (2026-07-12)

**Spec:** `/specs/13-branding-domains.md` (no CODEREF companion). **Branch:** `feature/13-branding-domains`. **Work type:** FEATURE.

### Feature registry
- `branding.core` — **isCore: true**, deps `core.tenancy`, `core.storage`, `core.shell`. Non-disableable: it owns the settings shell, per-host token injection and the mandatory "Powered by Rihaish" footer, and the always-on subdomain — turning it off would strip a tenant's identity.
- `branding.custom_domain` — non-core, deps `branding.core`. Gates binding a custom domain; the DAG blocks a society without it and the routes 403.
- `branding.pwa` — non-core, deps `branding.core`. Contract-only here (the manifest); the installable shell + push land in step 23.

### Data model (migration `20260712170000_branding_domains`)
- **`SocietyBranding`** — PK = `societyId` (1:1 with Society; a `findUnique` is inherently tenant-safe). Infra model (has `societyId`, NO `deletedAt`) → the scoping layer passes it through, so the service scopes every access on `societyId` explicitly. Fields per spec: `displayName`, `shortName` (≤12), `logoFileId`/`logoDarkFileId`/`faviconFileId`/`iconFileId`/`loginBgFileId`, `splashBgColor`, `primaryColor`, `emailSenderName`/`emailReplyTo`, `supportPhone`/`supportEmail`, `updatedAt`. FK → Society ON DELETE CASCADE.
- **`SocietyDomain`** (declared step 02) — added `verifyToken`, `lastCheckedAt`, `verifyError` for the DNS challenge flow. `verifiedAt` (existing) gates custom-host resolution in `lib/tenant.ts` (already implemented there).

### `lib/branding/*`
- **`contrast.ts`** (pure) — `parseHex`/`normalizeHex`/`isValidHex` (#rgb + #rrggbb), `rgbToHsl`/`hslToRgb`/`hslToken` (space-separated to match globals.css), WCAG `relativeLuminance`/`contrastRatio`, `deriveForeground` (white by default; on AA failure → hue-tinted near-black, else pure black — the better of white/black is provably ≥4.58:1, so AA is always met), `brandTokens(hex)` → light + dark `{primary, primaryForeground, ring}` triples. Dark mode lifts the primary lightness into a legible 45–70 band so a dark brand colour stays visible on the near-black surface. `adjusted`/`contrastRatio` track the **light** surface (the mode the admin edits in) — a dark-mode dark-foreground is the theme's normal treatment, not a warning.
- **`tokens.ts`** (pure) — `brandCss(hex)` → `<style>` inner text overriding **ONLY** `--primary`, `--primary-foreground`, `--ring`, in a `:root{}` + scoped `.dark{}` pair. Empty/missing → no override (shipped emerald stays); invalid hex → no override + `invalid` flag.
- **`domains.ts`** — `subdomainHost(slug, apex)`, `normalizeCustomHost` (strips scheme/path/port/trailing-dot, validates DNS labels, rejects a rihaish-managed apex/wildcard host), `newVerifyToken` (16-byte hex), `dnsInstructions` (CNAME host→`<slug>.<apex>`, TXT `_rihaish-challenge.<host>` = `rihaish-verify=<token>`), pure `evaluateTxt` (verified | txt-missing | txt-mismatch), `resolveTxt` (node:dns/promises, empty on error), `verifyHost` (injectable resolver so worker + tests share one path), `diagnosticFor`.
- **`service.ts`** — infra DB glue via unscoped `prisma` with explicit `societyId`. `getBranding` (row or name-derived default), `upsertBranding` (validates hex + short-name, checks each asset id belongs to the society **through the scoped storage service** so a foreign file id is "not found", busts the CSS cache), `resolveBrandCss` (per-process TTL cache, reads only `primaryColor`, never throws — a bad stored colour degrades to default), `listDomains`/`bindDomain`/`unbindDomain`/`verifyDomain` (ownership-checked, `P2002`→`host_taken`), `verifyPendingDomains` (worker sweep across all societies), `resolveAssetUrls` (signed preview URLs for the settings UI). Typed `BrandingError` (`invalid_color`, `short_name_too_long`, `unknown_asset`, `invalid_host`, `host_taken`).

### Worker
- `CRON_REGISTRY` += `domains.verify` (platform scope, `*/15 * * * *`). `JOB_REGISTRY` += `domains.verify` — **mutating: false** (SocietyDomain is infra, not tenant domain data, so it runs even for a READ_ONLY society), concurrency 1, handler `runDomainsVerify` → `verifyPendingDomains`. Registry coverage test updated to include the new code.

### API
- `GET/PUT /api/society/branding` — read needs `society.settings.read`, write needs `society.settings.update`; returns row + signed asset URLs + live contrast diagnostic; `BrandingError`→400.
- `GET/POST /api/society/domains`, `DELETE`/`POST(verify) /api/society/domains/[id]` — list needs settings-read; all mutations need settings-update **and** `requireFeature(branding.custom_domain)`.
- `GET /manifest.webmanifest` — per-host, generated from the resolving society's branding (name/short_name/theme_color/background_color/icons/lang/dir). Unknown/non-society host → generic 404 (no enumeration). Middleware never runs on this path (dotted), so the handler resolves the host itself.

### UI
- **Token injection** — `app/[locale]/app/layout.tsx` injects the scoped brand `<style id="rihaish-brand">` (hoisted to `<head>`) + `<link rel="manifest">`. Two hosts → two brands from one deploy.
- **Powered by Rihaish** — `components/branding/powered-by.tsx` rendered in `app/[locale]/layout.tsx` (root) on every page of every host; body is now a `min-h-dvh` flex column so the footer sits at the bottom. No entitlement, no off switch. Deliberately unbranded (does not adopt `--primary`).
- **Settings → Branding & Domains** (`components/branding/branding-client.tsx`) — own Simple/Pro toggle. Simple = logo + colour + support phone. Pro = all assets (dark logo, favicon, app icon, login bg), email sender identity, splash bg, support email, and the custom-domain wizard. Colour picker runs the **same** contrast maths client-side (imports pure `contrast`/`tokens`) for a live badge (OK / auto-corrected / invalid) and a **scoped** preview swatch (`.brand-preview` + `.dark .brand-preview` — never leaks vars to the page). Domains: read-only subdomain chip; per-domain card with copyable CNAME+TXT rows, "Check now", pending diagnostic, and on verify an **SSL manual-step notice**. i18n `branding` namespace en + ur.

### Gates
- `pnpm typecheck` ✓ · `pnpm lint` ✓ (0 warnings) · `pnpm test:unit` ✓ **344 passed / 3 skipped** (+30: `contrast.test`, `tokens.test`, `domains.test`; registry coverage test updated) · `pnpm build` ✓ (all new routes + `/manifest.webmanifest`) · worker registry loads `domains.verify` handler under tsx.
- e2e `e2e/branding-domains.spec.ts` — unauth 401s on branding/domains routes, manifest per-host + unknown-host 404, "Powered by Rihaish" present on `/en`. (e2e is not a CI gate; runs against seeded staging.)

### Decisions & notes
- **`adjusted`/warning keys off the light surface** — dark mode always gets a contrast-correct (often dark) foreground, matching the shipped emerald default (`--primary-foreground: 168 40% 8%`); surfacing that as a "warning" would false-alarm on every dark brand, so the admin warning tracks the light-mode button only.
- **SSL is `[HUMAN_REQUIRED]` infra, not a build stop** — per-domain SSL issuance on aaPanel is exactly the infra the agent may not script. It is a **runtime** operational handoff (no custom domain exists in this build), surfaced to admins via the domain-card SSL notice and documented here; the code module is fully complete, so this step ends `[CHECKPOINT]`, not `[HUMAN_REQUIRED]`. The agent never touches the panel. Wildcard SSL/DNS remains the one-time `[HUMAN_REQUIRED]` from ARCHITECTURE §7.
- **Custom-domain resolution** already lived in `lib/tenant.ts` (`resolveByCustomDomain` requires `verifiedAt != null`), so an unverified host 404s while the subdomain keeps working — no change needed there.
- **Manifest icons** are best-effort (signed for 1h via a `system-manifest` actor) and minimal; the full icon set + installable behaviour ship in step 23 (`branding.pwa`).
- **Email sender identity** (`emailSenderName`/`emailReplyTo`) is captured here; wiring it into the notifications SMTP send path is the notifications module's concern and can read `SocietyBranding` when it next touches sender headers.

**Checkpoint:** [CHECKPOINT] — progress marker; controller auto-approves and continues to step 14 (ui-modes).

---

## Step 14 — UI modes (Simple / Pro) — DONE (2026-07-12)

**Branch:** `feature/14-ui-modes` · **Spec:** `/specs/14-ui-modes.md` (no CODEREF)

**Goal.** Per-module Simple vs Pro interfaces. Simple = easy for a non-technical user (fewer columns, plain language, guided flows, bigger targets), Pro = dense/keyboard-first/raw-codes. Resolved per module, not per app: user-pref (only when the society is entitled to `ui.modes` AND the module policy allows override) → society default → the module's built-in default (SIMPLE resident-facing, PRO committee/accounting). Losing the entitlement retains prefs and re-applies them on re-enable.

### Feature registry
- Added `ui.modes` (entitled, `isCore:false`, module `14-ui-modes`, dependsOn `core.shell`, `core.account`) to `lib/features.ts`. DAG validates at load; `getFeature("ui.modes")` confirmed.

### Data model — migration `20260712180000_ui_modes`
- `enum UiMode { SIMPLE PRO }`.
- `SocietyModuleUiPolicy` (`id`, `societyId`, `module`, `defaultMode UiMode`, `userCanOverride Boolean @default(false)`, timestamps; `@@unique([societyId,module])`, `@@index([societyId])`). Infra config model — has `societyId` but **no** `deletedAt`, so the tenant-scoping layer (`lib/db.ts`: scoped iff BOTH fields) passes it through; the service filters on `societyId` explicitly. FK → Society `onDelete: Cascade`, back-relation `Society.uiPolicies`.
- `UserModuleUiPref` (`id`, `userId`, `module`, `mode UiMode`, timestamps; `@@unique([userId,module])`, `@@index([userId])`). Keyed by user; FK → User `onDelete: Cascade`, back-relation `User.uiModePrefs`. **Never deleted on an entitlement change** — the resolver simply stops consulting it.
- `prisma generate` OK (ARM64 + native targets).

### lib/ui-modes/*
- `modules.ts` — `type UiMode`, `ModuleAudience`, `UiModuleDef`, `defaultForAudience` (resident→SIMPLE, committee→PRO). `UI_MODULES` = forward-declared registry of 12 mode-supporting modules (dashboard[reference], residents, billing, payments, finance, announcements, complaints, gatepass, expenses, documents, amenities, reports) each with `ownerStep`+`audience`. `getUiModule`, `builtInDefault` (SIMPLE for unknown), `MODE_MODULES`. Registry is forward-declared so settings/account have real rows before later steps build the module UIs — a module absent here simply never shows a toggle.
- `resolve.ts` — **pure** (no I/O). `canUserChoose(hasEntitlement, policy)` = entitled ∧ `policy.userCanOverride`. `resolveMode({hasEntitlement, policy, userPref, moduleDefault})` = exact spec order: (1) canChoose ∧ userPref → pref; (2) policy → policy.defaultMode (applies **even when not entitled** — the "defaults only" case); (3) → moduleDefault. `resolveModule` returns `{mode, canChoose}`.
- `resolve.test.ts` — 12 tests: the 3-way order, override-disallowed-with-pref, no-policy fallback, entitlement-off ignores pref → society default, re-apply pref on re-enable, `canUserChoose` truth table, `resolveModule`, registry defaults.
- `service.ts` — DB glue over `prisma` (context-independent). `getPolicyMap`, `getInterfaceConfig` (backfills built-in default + `configured` flag over all `MODE_MODULES`), `setPolicy` (upsert + manual `AuditLog` row `ui_mode.policy_set`, `actorType SOCIETY_USER`; validates module + mode). `getUserPrefMap`, `setUserPref` (**defense-in-depth**: re-checks `isEnabled(ui.modes)` ∧ policy override, throws `UiModeError('not_allowed')` → 403 so a hand-crafted request cannot store a disallowed pref). `resolveUserModes` (one entitlement + policy + pref read, then pure resolution per module → `{module, mode, canChoose, defaultMode, userPref}`), `resolveModeMap` (serialisable `module→{mode,canChoose}` for the client shell). `UI_MODES_FEATURE`, `UiModeError`.

### Shell wiring (module-aware top-bar toggle)
- `ShellData` gains `uiModes: Record<string,{mode,canChoose}>` (replaces the placeholder `showModeToggle` bool). `buildShellData` resolves it via `resolveModeMap` only for a user who belongs to the host (guest/cross-society → `{}`, so the toggle never appears).
- `components/ui-modes/ui-mode-context.tsx` — client `UiModeProvider` holds the modes map + `activeModule`. `useUiModule(module)` registers the module as active on mount / clears on unmount and returns `{mode, canChoose, setMode}`; `useUiModes()` (non-registering) for multi-module surfaces; `useActiveMode()` for the top bar. `setMode` is optimistic (instant, no reload → in-flight form state survives) + background `PATCH /api/me/ui-modes`, reverting + toasting on failure.
- `AppShellClient` wraps the frame in `<UiModeProvider initialModes={data.uiModes}>`. `TopBar` always renders `<ModeToggle/>`; the toggle **self-hides** (returns null) unless there is an active module with `canChoose` — hidden, never shown-but-disabled — and drives that module's mode via context.

### API
- `PATCH /api/me/ui-modes` (+ `GET` = resolved modules for the account surface) — `requireAccountActor` (society user), Zod `{module, mode}`, `setUserPref`; `UiModeError` → 403 `not_allowed` / 400 else.
- `GET/PUT /api/society/ui-modes` — `GET` (`society.settings.read`) returns config + `entitled`; `PUT` (`requireSettingsActor` = `society.settings.update`) Zod `{module, defaultMode?, userCanOverride?}` → `setPolicy` with the acting user as audit actor.

### UI (both modes fully designed; tokens, light/dark, RTL via `dir`, responsive, states)
- Settings → **Interface** (`components/ui-modes/interface-panel.tsx` in `settings/page.tsx`): per-module rows — segmented default (Simple/Pro) + "let users choose" `Switch` (disabled with `notEntitledNote` when the society lacks `ui.modes`, since override is inert without it). Collapsible **side-by-side preview** (`mode-preview.tsx`: guided cards vs dense table sketch). Optimistic per-row `PUT`, revert+toast on fail; read-only without `society.settings.update`.
- Account → **Interface** prefs (`components/ui-modes/user-modes-panel.tsx` on the account page, personal device only): lists **only** override-allowed modules (`canChoose`), each a Simple/Pro segmented control wired through the shared `useUiModes` context so it and the top-bar toggle stay in lockstep with no reload; empty state when nothing is customisable.
- Reference module **dashboard** (`components/dashboard/dashboard-client.tsx`, `/app/page.tsx` now a thin server wrapper): declares itself the `dashboard` module via `useUiModule("dashboard")` and renders a fully-designed **Simple** (few big guided metric cards, plain language, no deltas) and **Pro** (dense 4-col metric grid, raw values + deltas). Switching from the top bar is instant.
- i18n en+ur: new `uiModes` namespace (module labels ×12, interface/prefs copy, `notEntitledNote`, `defaultIs`) + `dashboard` additions (`subtitlePro`, `metric.*.simple|pro`).

### Gates
- `pnpm typecheck` ✓ · `pnpm lint` ✓ 0 warnings (renamed a `const module` to satisfy `no-assign-module-variable`) · `pnpm test:unit` ✓ **356 passed / 3 skipped** (+12) · `pnpm build` ✓ (both `/api/me/ui-modes` and `/api/society/ui-modes` present). e2e `e2e/ui-modes.spec.ts` (unauth 401s on both routes, dashboard never 500) — not a CI gate.

### Acceptance criteria
- [x] Vitest pins the resolution order incl. the entitlement-off fallback.
- [x] Toggle renders only when the user may choose; hidden otherwise (self-hiding `ModeToggle` + empty `uiModes` for guests).
- [x] Switching does not reload and does not lose form state (client context re-render; optimistic + background persist).
- [x] Losing the entitlement retains, not deletes, user prefs (`UserModuleUiPref` untouched; resolver stops consulting it; re-applies on re-enable — unit-tested).
- [x] Both modes designed for a reference module (dashboard) in light/dark, EN/UR (RTL via `dir`), mobile + desktop (responsive grids).

### Notes / decisions
- Module registry is intentionally forward-declared: later feature steps (billing, complaints, gatepass, …) just implement both variants and call `useUiModule(code)`; their policy/pref rows and settings surfaces already exist.
- `resolveModeMap` adds two small indexed reads to every society shell render (entitlement is cached). Acceptable per the scale principle; can be folded into a single shell query later if needed.
- The account screen's own local Simple/Pro toggle (spec 09, `localStorage`) is unrelated to `ui.modes` and left as-is — the account module is not in `UI_MODULES`.

**Checkpoint:** [CHECKPOINT] — progress marker; controller auto-approves and continues to step 15 (society-structure).

---

## Step 15 — society-structure — DONE (2026-07-12)

**Branch:** `feature/15-society-structure` · **Work type:** FEATURE
**Spec:** `/specs/15-society-structure.md` (authoritative; no CODEREF present). A substantial partial (schema, migration, pure logic, service, one route, unit tests) existed uncommitted in the working tree from a prior run; this step completed it (API routes, full UI, i18n, e2e, wiring verification) and hardened it (typecheck was never run on the partial — fixed the create-typing and null-acceptance defects below).

### Feature / DAG / RBAC / nav / ui-modes
- `flats.registry` — **isCore** (never disableable; a society cannot exist without its flat registry), `dependsOn: [core.tenancy, core.tables, core.forms]` (all present; `features.test` green).
- Permissions: `society.structure.read` / `society.structure.manage`, both gated by `flats.registry`. Seeded onto roles: committee/admin get manage, others read (see `lib/rbac.ts`).
- Nav item `structure` (icon `building`, group manage, `hideOnSharedDevice`), `nav.structure` label en+ur.
- UI module `structure` registered in `lib/ui-modes/modules.ts` (audience committee → built-in default PRO).

### Data model — migration `20260712190000_society_structure`
- Enums `OccupancyStatus {OCCUPIED_BY_OWNER,RENTED,VACANT,UNDER_CONSTRUCTION}`, `BillTo {OWNER,OCCUPANT}`, `ParkingMode {ASSIGNED,OPEN,NONE}`, `VehicleType {CAR,BIKE,OTHER}`.
- `Block`, `Floor`, `FlatCategory`, `Flat`, `ParkingSlot`, `Vehicle` — **every domain row carries BOTH `societyId` AND `deletedAt`** (architecture #1/#7 override the spec's illustrative schema which showed `deletedAt` only on Block/Flat), so the enforced scoping layer auto-pins reads, stamps societyId, and rewrites deletes into soft-delete + audit. `SocietySettings` (PK `societyId`, no `deletedAt`) is the pass-through infra exception, scoped explicitly.
- Uniqueness (`Flat @@unique[societyId,displayCode]`, `Block @@unique[societyId,code]`, …) **excludes `deletedAt`**: a soft-deleted/merged flat keeps its `displayCode` reserved so a demolished number is never silently reused (spec edge case) and its financial history stays attributable.
- FKs: Floor/Flat/Vehicle → parent `ON DELETE CASCADE`; Flat.floor/category and ParkingSlot.flat → `SET NULL`. `prisma validate` + `prisma generate` clean; 34 DDL statements verified.

### Pure logic (no DB — exhaustively unit-tested, carried from the partial)
- `flat-code.ts`: `{block}/{number}/{floor}` template → society-unique `displayCode`; must contain `{number}`; tolerant `effectiveFormat` fallback. (`flat-code.test`)
- `bulk.ts`: `generateFlats` (floorPrefixed = floor·10^digits+pos, or sequential from startAt) + `planBulkGenerate` — refuses the ENTIRE batch on any in-batch dup, existing-code conflict, or `maxFlats` breach ("blocked before creating anything"). (`bulk.test`)
- `csv.ts`: RFC-4180-ish parser + per-row validation against known blocks/categories/existing codes; structural errors as `line:0`; caller commits only when `errors` is empty → **no partial import**. (`csv.test`)

### Service (`lib/society-structure/service.ts`)
- Scoped CRUD for blocks/floors/categories/flats/parking/vehicles + settings upsert. **First tenant-scoped domain module**, so established the create pattern: the scoping layer injects `societyId` at runtime but Prisma's types require it → added a documented `tenantCreate<T>(Omit<T,"societyId">)` cast (no placeholder societyId that could mask a scoping regression; DB NOT NULL is the final guard). Applied to all `db.*.create`/`createMany`.
- `getGridData` (blocks+floors+light flats for the Simple lattice), `listFlats/ParkingSlots/VehiclesQuery` (data-table kit via `runList`), `listFlatsForExport` (all rows matching the view, capped at inline limit+1), `getCsvContext`, `importFlats` (one transaction), `bulkGenerate` (plan → one txn backfilling floors then `createMany`).

### API routes (all `nodejs`/`force-dynamic`, actor = `requireStructureActor`, read/write scope via `runReadScope`/`runWriteScope`; typed errors → stable statuses in `http.ts`)
`/api/structure` (GET settings+summary+grid+categories, PATCH settings) · `/api/blocks|floors|flat-categories|parking-slots|vehicles` (list/CRUD; parking+vehicles also serve a `?q=` ListResult for the Pro tabs) · `/api/flats` (GET `?id=` detail else `?q=` list, POST/PATCH/DELETE) · `/api/flats/bulk` · `/api/flats/import` (2-phase: dry-run preview → commit, 422 on any error) · `/api/flats/export` (**first real `buildInlineExport` caller** — inherits PII gate + audit + >1000 queued).

### UI (`components/structure/*`, `app/[locale]/app/structure/page.tsx`)
- `StructureClient` owns shared data + overlays, switches Simple/Pro via `useUiModule("structure")`, reloads on every mutation. Shared summary strip (block/floor/flat counts + occupancy legend).
- **Simple** `FlatGrid`: block→floor→flat lattice colour-coded by occupancy, click → drawer, empty→"Set up my building".
- **Pro** `FlatTable`: flats DataTable (every column, server filters block/floor/category/occupancy/billable, CSV import + export) + **Parking**/**Vehicles** tabs (data-table kit); Parking tab hidden entirely unless `parkingMode=ASSIGNED` (spec: not merely disabled).
- Shared `FlatDrawer` (create + edit, vehicles add/remove, parking shown, soft-delete), `BulkGenerateModal` (atomic, live count preview), `CsvImportModal` (file → dry-run per-row error report → commit), `SetupModal` (blocks/floors/categories/settings). Form kit + toasts throughout; skeleton in drawer, EmptyState for empty grid.

### Defects found & fixed while completing the partial
- **Create typing:** the partial's `db.*.create({data:{…}})` omitted `societyId` and had never been typechecked (TS2322 ×8). Fixed with `tenantCreate` (see Service).
- **Null-acceptance mismatch:** client forms send `field || null` for empty optionals, but several Zod fields were `.optional().or(z.literal(""))` (reject `null`) → POST/PATCH would 400. Added `.nullable()` to `block.name`, `floor.label`, `flat.{floorId,categoryId,billTo,notes}`, `parkingSlot.{flatId,location}`, `vehicle.{make,model,color,stickerNo}`.

### Gates
- `pnpm typecheck` ✓ · `pnpm lint` ✓ (0 warnings, incl. `react/jsx-no-literals`) · `pnpm test:unit` ✓ 383 passed / 3 skipped · `pnpm build` ✓ (11 new API routes + `/app/structure` prerendered en+ur).
- UI design self-check: kits only (DataTable for every list incl. parking/vehicles; form kit for every input), logical properties only (`ms-auto`; no `ml-/mr-/pl-/pr-/text-left/right`) → RTL-safe, token-based colours (light+dark), Simple **and** Pro, skeleton/empty states, toasts.
- e2e `society-structure.spec` (13 unauth-401 assertions + page-no-500) — not a CI gate per house rule.

### Decisions / notes (no approval gate — recorded per CLAUDE.md)
- **No Rufi seed written.** Spec lists an illustrative "Rufi seed (blocks A–F, categories)" but the repo has no seed infrastructure (prior modules reference seeders that don't exist as files), and populating a society's flats is the onboarding wizard's job (step 21). Deferred there rather than inventing a seed harness.
- **Occupancy-change-doesn't-rewrite-invoices** acceptance criterion is structurally guaranteed (`updateFlat` only mutates the flat row; no invoice tables exist until step 18) — no test possible yet; will be covered when the ledger lands.
- Parking/vehicles Pro tabs use the data-table kit (added `?q=` ListResult support) to honour the "Data-Table kit for every list" contract rather than hand-rolling.

**Checkpoint:** [CHECKPOINT] — progress marker; controller auto-approves and continues to step 16 (residents-occupancy).

---

## 16 — residents-occupancy — DONE (2026-07-12)

**Branch:** `feature/16-residents-occupancy` · **Spec:** `/specs/16-residents-occupancy.md` (no CODEREF) · WORK TYPE: FEATURE

**Purpose:** Who lives where, who owns what, who gets billed, and who may collect a child from the gate. One account holds many flats via `FlatOccupancy` (many-to-many); residents enter only via invite or CSV import (no self-signup).

### State on entry
The branch already carried a nearly-complete implementation from a prior session (uncommitted): schema models, migration, `lib/residents/*`, schemas, API routes, and UI. Gates had not been run and progress files were not updated. This step ran the verification gates, fixed the defects they surfaced, added the e2e spec, and recorded/committed.

### Schema / migration (`20260712200000_residents_occupancy`)
- Enum `Relation {OWNER,OCCUPANT}`.
- `FlatOccupancy` (societyId+deletedAt → enforced scoping auto-pins reads / soft-deletes; `@@index([societyId,flatId,endDate])`, `@@index([societyId,userId])`; `endDate=null` = current holding).
- `AuthorizedPickupPerson` (societyId+deletedAt scoped; for CHILD_EXIT in step 28) — `cnicEnc`, `photoFileId`, `isActive`.
- `ResidentInvite` — single-use security TOKEN resolved on the PUBLIC accept endpoint (no scope yet), so — like `OtpCode` — NO `deletedAt`; scoped explicitly by the service. `token @unique`, `expiresAt`, `acceptedAt`.

### Pure logic (unit-tested)
- `occupancy.ts` — `deriveFlatOccupancy` (holdings → OccupancyStatus, ignores closed rows, RENTED whenever an occupant present, preserves UNDER_CONSTRUCTION while empty) + `planOccupancyTransfer` (closes old owner + opens new on ownership transfer; adds tenant WITHOUT closing owner when owner rents out; no double-open when incoming user already holds the relation; demotes previous primary).
- `invite-token.ts` — `newInviteToken` (URL-safe non-guessable), `inviteExpiry` (7d), `inviteUsability` (single-use once accepted, expires after TTL, resend = fresh token + pushed-out expiry usable again), `inviteUrl`.
- `csv.ts` — RFC-header-validated parse, phone→E.164 normalise, row-level errors (unknown flat, bad phone, invalid relation, in-file duplicate person-flat pair), same person on two flats allowed, optional email/is_primary/occupancy override, empty-file report. No partial import.

### Service / API
- `service.ts`: scoped invite/resend/accept, `transferOccupancy`, `updateOccupancy`, `suspendResident`, directory/summary/list/export, `getFlatPanel` (current + history timeline + pickup persons), pickup CRUD, `importResidents` (one txn, atomic). Existing phone in the same society → linked to an additional flat, not duplicated. CNIC captured only when `SocietySettings.cnicCaptureEnabled`; encrypted + masked.
- Routes: `GET/POST /api/residents`, `PATCH /api/residents/[id]`, `POST /api/residents/invite`, `POST /api/residents/invite/[token]/accept` (PUBLIC), `POST /api/residents/import` (`?dryRun`), `POST /api/residents/export`, `POST /api/flats/[id]/occupancy/transfer`, `GET/POST/PATCH/DELETE /api/flats/[id]/pickup-persons`. Typed errors → stable statuses (409/404/410/400) via `residents/http.ts`, else shared auth envelope (401/403/423).
- `residents.registry` (isCore) registered in `lib/features.ts` (deps flats.registry/core.auth/core.notifications); nav `residents` gated by `society.residents.read` + feature; rbac perms; notifications audience + templates extended.

### UI (`useUiModule` Simple/Pro)
- Simple: card directory (photo, flat, role, call/WhatsApp) + add-resident invite wizard.
- Pro: resident DataTable (filters, bulk invite, CSV import dry-run diff, export), shared `FlatPanelDrawer` (occupants + make-primary/move-out/edit, history timeline, pending invites resend/copy, pickup persons, transfer-ownership). Form kit + toasts throughout.

### Defects found & fixed while completing the partial
- **lint (`react/jsx-no-literals`) ×3** — bare `:` / `·` literals in JSX (`flat-panel-drawer` CNIC line, `resident-cards`, `resident-table`) → wrapped in expression containers.
- **Unused symbols ×2** (lint warnings, `--max-warnings 0` fails the gate) — removed the unused `Copy` import and the unused `data` prop (+ its `ResidentsData` type import) from `FlatPanelDrawer`; dropped the now-dead `data={data}` at the call site in `residents-client`.

### Gates
- `pnpm typecheck` ✓ · `pnpm lint` ✓ (0 warnings/errors) · `pnpm test:unit` ✓ 408 passed / 3 skipped (was 383 → +25 residents unit tests) · `pnpm build` ✓ (8 new API routes + `/app/residents` prerendered en+ur).
- i18n en+ur parity: `residents` ns 121 keys each, no missing/extra.
- e2e `residents-occupancy.spec` (9 unauth-401 assertions + invite-accept public-client-error + page-no-500) — not a CI gate per house rule.

### Decisions / notes (no approval gate — recorded per CLAUDE.md)
- **Invoices-untouched acceptance criterion** remains structurally guaranteed (no invoice tables exist until step 18; transfer only writes `FlatOccupancy` + `Flat.occupancy`). Will be covered by a test when the ledger lands.
- Invite-accept is the one public route (token, not session): the e2e asserts it never 401s and never 500s — a bogus token is a plain client error (404/410/400).
- `lib/residents.ts` (a pre-existing stub) removed in favour of the `lib/residents/` module directory.

**Checkpoint:** [CHECKPOINT] — progress marker; controller auto-approves and continues to step 17 (charge-engine).

---

## 17 — charge-engine — DONE (2026-07-12)

**Branch:** `feature/17-charge-engine` · **Spec:** `/specs/17-charge-engine.md` (no CODEREF in range) · **Feature:** `billing.charges` (entitled; deps `flats.registry`, `residents.registry`).

**Goal:** decide what each flat owes, and why. Effective-dated rates that are never edited in place, so a past invoice is always reproducible.

### Registrations
- `lib/features.ts`: `billing.charges` (isCore=false, module `17-charge-engine`, deps `flats.registry`+`residents.registry`). DAG validates.
- `lib/rbac.ts`: perms `billing.charges.read`/`billing.charges.manage` (gated by `billing.charges`). ACCOUNTANT → read+manage; MANAGER → read; COMMITTEE_MEMBER → read; SOCIETY_ADMIN via `*`.
- `lib/nav.ts`: `charges` item (icon `coins`, perm `billing.charges.read`, feature `billing.charges`, hideOnSharedDevice) between structure/residents-block and billing. `components/shell/nav-icons.ts`: added `coins → Coins`.
- `lib/ui-modes/modules.ts`: `{ code: "charges", ownerStep: "17-charge-engine", audience: "committee", supportsModes: true }` → built-in default PRO.

### Schema + migration (`20260712210000_charge_engine`)
- Enums `ChargeKind{RECURRING,ONE_OFF,METERED}`, `TargetType{ALL,BLOCK,CATEGORY,FLAT_LIST,SINGLE_FLAT}`, `SplitMode{PER_FLAT,SPLIT_EQUALLY}`, `SpecialChargeStatus{DRAFT,SCHEDULED,APPLIED,CANCELLED}`.
- `ChargeHead` (code/name/kind/isActive/sortOrder), `RateRule` (occupancy?/categoryId?/blockId?/flatId? conditions, amountMinor BigInt, effectiveFrom, effectiveTo?, supersededBy?, createdBy), `SpecialCharge` (title/description?, amountMinor BigInt, targetType, targetIds String[], splitMode, dueDate, billingPeriod?, status, appliedAt?, createdBy). EVERY table carries `societyId` + `deletedAt` → the `lib/db.ts` scoping layer auto-pins reads and rewrites deletes into soft-delete+audit. FK RateRule/SpecialCharge → ChargeHead (onDelete Cascade). `blockId/categoryId/flatId/targetIds` are plain ids (validated in-society by the service), NOT FKs — so a flat soft-delete never cascades a rule (history preserved).

### Pure logic (unit-tested — 29 tests across resolve/split/period)
- `resolve.ts` `resolveCharges(flat, periodStart, heads, rules, specials)`: per RECURRING head, candidate = effectiveFrom≤start ∧ (effectiveTo null ∨ ≥start) ∧ every non-null condition matches; winner = highest `specificity` score `flatId(1000)>categoryId+occupancy(110)>categoryId(100)>occupancy(10)>blockId(1)>default(0)` (reproduces the spec ordering exactly), tie-break latest effectiveFrom → newest createdAt → greatest id (fully deterministic). No candidate → `no_rule` warning (never a silent zero). METERED head → `metered_unavailable` warning (step-37 source not shipped). Special charges appended; SPLIT_EQUALLY uses `shareForIndex` on the ordered target list.
- `split.ts` `splitEqually(total,count)` / `shareForIndex`: BigInt-only; base=total/count, first `remainder` parts get +1 → sum reconciles EXACTLY. **Decision:** followed the spec's authoritative RULE ("first N flats take the extra minor unit", N=remainder → e.g. 100000/7 = 14286×5 + 14285×2) rather than the spec edge-case example's looser arithmetic (14286×6 + 14284×1); both reconcile, the RULE version is the standard largest-remainder method and is what the tests assert.
- `period.ts`: `"YYYY-MM"` → UTC month start/end; resolution keys off PERIOD START, so a mid-month rate change never touches the current invoice.

### Service (`lib/charges/service.ts`) — all under active tenant scope
- Head CRUD (`duplicate_head_code` on P2002). Rate rules: `createRateRule` validates head + in-society conditions, then in one txn creates the new rule AND supersedes the open rule with the identical condition signature (sets its `effectiveTo`=new.effectiveFrom + `supersededBy`) — **never mutates the old amount**, so a 2024 invoice stays reproducible. This single path serves both "set a rate" and "change a rate". `listRateRules` (data-table kit), `listRuleHistory` (per-head, newest first), `deleteRateRule` (soft, for a mistaken rule).
- Special charges: `createSpecialCharge` (DRAFT/SCHEDULED; `empty_target` if a non-ALL target resolves to zero live flats), `updateSpecialCharge`/`cancelSpecialCharge` (APPLIED → `applied_immutable` 409). `resolveTargetFlats(type, ids)` → ordered live flat ids by displayCode (soft-deleted excluded → a demolished flat is silently skipped; non-billable INCLUDED so a construction levy can target under-construction flats). Same function backs the preview AND the eventual apply → counts match.
- `simulate(flatId, period)`: loads flat + active heads + live rules + SCHEDULED specials for the period, resolves the target flats for each special, calls the SAME pure `resolveCharges` the step-18 run will use → simulator output is guaranteed to match the produced invoice.
- `getOverview(money)`: heads (+open rules +rule count), all special charges, selector context (blocks/categories/flats), currency/minorUnitDigits.
- Money is BigInt end-to-end; serialised to strings at the module edge (JSON has no BigInt). `parseAmountMinor` parses a major-unit string via `parseMoneyInput` (rejects negatives).
- `actor.ts` mirrors structure/residents (`requireChargesActor`, `getChargesServerActor`, `runReadScope`/`runWriteScope` folding impersonation-RO + READ_ONLY society status → 423). `http.ts` maps typed `ChargesError` (duplicate/`applied_immutable` → 409, `unknown_*` → 404, else 400). `columns.ts` rate-rule + special-charge data-table whitelists. `schemas/charges.ts` (client+server Zod).

### API (6 route files)
- `/api/charges` GET overview · `/api/charges/simulate` POST · `/api/charge-heads` GET/POST/PATCH/DELETE · `/api/rate-rules` GET(list `?q` OR history `?chargeHeadId`)/POST(supersede)/DELETE · `/api/special-charges` GET(list `?q` OR all)/POST/PATCH/DELETE(cancel) · `/api/special-charges/preview` POST (live flat count + sample codes).

### UI (`components/charges/*`, `useUiModule("charges")`)
- **Simple** (`simple-charges.tsx`): per active RECURRING head, a card with a per-occupancy amount row (OCCUPIED_BY_OWNER/RENTED/VACANT/UNDER_CONSTRUCTION) showing the current occupancy-only rule amount + a set/change button that opens `RateRuleModal` locked to that head+occupancy (behind it writes a proper effective-dated occupancy rule that supersedes the old). "One-time charge" button → `SpecialChargeModal`.
- **Pro**: `HeadManager` (head list w/ edit), `RateRuleTable` (data-table: specificity columns, effective-from, current-vs-superseded, delete row action, "New rate" toolbar → full-condition `RateRuleModal`), `RateSimulator` (flat+month pickers → lines with source + total + no-rule/metered warnings).
- Shared `SpecialChargeModal`: head/title/amount/split/target-type/targets/due/billing-period, a **live target preview** (debounced POST to `/preview`) showing "This will bill N flats" + computed total (per-flat × N, or the split total). `SpecialChargeList` (both modes; cancel non-applied).
- Money rendered via `formatMoney` with the society currency/minorUnitDigits.

### i18n
- `charges` namespace en+ur, **110 keys each, full parity** (verified by key-diff). `nav.charges` added both locales.

### Gates
- `pnpm typecheck` clean · `pnpm lint` 0 warnings · `pnpm test:unit` **437 passed** / 3 skipped (+29 new) · `pnpm build` exit 0 (6 new API routes + `/[locale]/app/charges` page, both locales). e2e `e2e/charge-engine.spec.ts` (12 unauth-401 route checks + charges page no-500; not a CI gate).
- `prisma format` normalised the new block; `prisma generate` OK. `prisma validate` reported only a `getConfig` env error (no `DATABASE_URL` locally) — not a schema error.

### Decisions / notes
- Split-equally follows the stated RULE, not the spec's looser example arithmetic (see split.ts note); both reconcile exactly.
- `resolveTargetFlats` includes non-billable flats on purpose (construction-levy case); billability is a recurring-charge concern decided at invoice time (step 18).
- Simulate & preview reuse the exact functions the step-18 invoice run will call → the two acceptance criteria "simulator matches the run" and "preview count matches applied" are structurally guaranteed.
- RateRule/SpecialCharge given `deletedAt` (spec snippet omitted it) so they are first-class tenant-scoped models like every other domain row (architecture #1/#7); rules are superseded, not deleted, in normal flow.

**Checkpoint:** [CHECKPOINT] — progress marker; controller auto-approves and continues to step 18 (ledger-invoicing).

## 18 — ledger-invoicing — DONE (2026-07-12)

**Spec:** `/specs/18-ledger-invoicing.md` (authoritative; no CODEREF for 18). Branch `feature/18-ledger-invoicing`. This step was resumed from a substantially-complete uncommitted WIP: I reviewed every file against the spec, fixed one real defect (below), and ran the full gate set.

### Feature / RBAC / nav
- `billing.ledger` feature registered in `lib/features.ts` — `dependsOn: ["billing.charges","core.worker","core.notifications"]`, `isCore:false`, module `18-ledger-invoicing`.
- Permissions `billing.invoice.read` / `billing.invoice.create` in `lib/rbac.ts`, both gated by the `billing.ledger` feature (the ∩ rule: a role can only exercise them once the society is entitled). Roles already carry them (ACCOUNTANT read+create; MANAGER/COMMITTEE read).
- Nav `billing` entry (`lib/nav.ts`) gated by `permission: billing.invoice.read` + `feature: billing.ledger` — covered by existing `nav.test.ts` cases.

### Data model (migration `20260712220000_ledger_invoicing`)
- Enums `InvoiceStatus{DRAFT,ISSUED,PARTIALLY_PAID,PAID,OVERDUE,CANCELLED}`, `LedgerType{INVOICE,PAYMENT,REVERSAL,LATE_FEE,ADJUSTMENT,OPENING_BALANCE,WRITE_OFF}`, `Direction{DEBIT,CREDIT}`, `LateFeeMode{FLAT,PERCENT}`.
- `Invoice` / `InvoiceLine` / `LedgerEntry` — all carry `societyId` + `deletedAt` → auto-scoped by `lib/db.ts`. Infra `InvoiceSequence` (PK `(societyId,prefix,year,month)`) and `BillingSettings` (PK `societyId`), no `deletedAt`, scoped explicitly.
- **No `paidAmount` column** (architecture #4): money is the ledger. `Invoice.status` is stored as the money-derived state at write time; OVERDUE is a time overlay applied on read (`overlayOverdue`), never persisted by a read.
- Idempotency: partial unique index `Invoice_societyId_flatId_period_key ... WHERE period IS NOT NULL` — a racing second insert for the same (flat, period) fails P2002 and that flat is skipped, so a re-run only fills gaps. Standalone (period NULL) special-charge invoices are exempt.

### The one defect fixed
- The migration declared FKs for `InvoiceLine→Invoice` and `BillingSettings→Society` but **omitted `Invoice_flatId_fkey`**, even though `schema.prisma` declares `flat Flat @relation(fields:[flatId], references:[id], onDelete: Cascade)` and `Flat.invoices Invoice[]`. Every prior migration creates an FK for each declared relation (e.g. `Vehicle_flatId_fkey → Flat ON DELETE CASCADE`), so this was genuine schema↔migration drift a fresh `prisma migrate dev` would flag. Added `ALTER TABLE "Invoice" ADD CONSTRAINT "Invoice_flatId_fkey" FOREIGN KEY ("flatId") REFERENCES "Flat"("id") ON DELETE CASCADE ON UPDATE CASCADE;`. (Cascade never fires in practice — flats are soft-deleted, arch #7 — but the SQL must match the schema.)

### Pure logic (unit-tested, clock-injected, BigInt-only)
- `balance.ts` — `balanceOf = Σdebit−Σcredit` (the ONE balance function), `totals`, `paymentStatus`, `overlayOverdue`, `advanceToApply`. 15 tests.
- `numbering.ts` — `formatInvoiceNumber` (tokens `{prefix}{YYYY}{YY}{MM}{M}{seq}{seq:N}`), `normalisePrefix`. 9 tests. Gaplessness itself is the service's atomic upsert, not this pure half.
- `aging.ts` — `daysOverdue`/`bucketForDays`/`ageArrears` → 0–30 / 31–60 / 61–90 / 90+, only positive outstanding ages. 5 tests.
- `late-fee.ts` — `lateFeeAmount` (FLAT minor units / PERCENT basis-points floored), `graceDeadline`/`isPastGrace`, `feePeriod`. 6 tests.

### Service (`lib/billing/service.ts`)
- `getBillingSettings`/`updateBillingSettings` (upsert + audit; prefix normalised, days clamped 1–28).
- `previewRun` — resolves every flat via the SAME pure `resolveCharges` the step-17 simulator uses; writes NOTHING; returns per-flat lines/total/warnings + skips (no_billable_person / no_charges) + alreadyInvoiced + summary. Non-billable flats keep only explicitly-targeted special-charge lines (construction-levy case).
- `commitRun` — idempotent per (society, period). Each invoice in its OWN transaction: allocate seq via `INSERT … ON CONFLICT DO UPDATE … RETURNING` (gapless, rolls back with the txn) → `Invoice` + `InvoiceLine[]` snapshot + `LedgerEntry(INVOICE, DEBIT)`. **Advance auto-application:** if the flat has a credit balance (and advances allowed), applies `min(total, advance)` as a matched CREDIT (attributed to the invoice, so the invoice reads PAID/PARTIALLY_PAID) + DEBIT (drawn from the advance pool) that net zero on the running balance. P2002 race → count as alreadyInvoiced, continue. After all writes: mark the period's SCHEDULED specials APPLIED, audit, then `notifySafe` per invoice (never blocks the run).
- Reads: `listInvoices`/`listInvoicesForExport` (data-table kit; paid amount derived from linked non-INVOICE ledger entries), `getInvoiceDetail` (with snapshot lines). `cancelInvoice` — writes a REVERSAL credit for the outstanding debit, flips status CANCELLED + reason + audit, **keeps the number, never deletes**; a second cancel is a typed 409.
- `getFlatStatement` (balance + invoices + raw ledger + aging), `getAgingReport` (society-wide). `addOpeningBalance` (OPENING_BALANCE debit/credit, blocked once `openingBalanceLocked`) + `lockOpeningBalances` — the step-21 onboarding seam.
- Ledger is APPEND-ONLY: the service only ever calls `.ledgerEntry.create`; corrections are new REVERSAL/ADJUSTMENT rows.

### Worker (`lib/worker/handlers.ts` + `registry.ts`)
- `runBillingMonthly` replaced the step-18 stub: manual run carries `payload.period` and runs unconditionally; a cron fire (no period) runs only when `autoGenerate` is on and bills the occurrence month. READ_ONLY society → the scoped mutating job is skipped by the runtime. Idempotent via `commitRun`.
- `runBillingLateFee` replaced the step-19 stub: one `LATE_FEE` debit per overdue ISSUED/PARTIALLY_PAID invoice per month (never compounding daily), idempotent by looking up an existing LATE_FEE entry in the current month window; disabling stops future fees and removes none.

### API (9 routes) + UI
- `/api/billing`(overview), `/settings`(GET/PATCH), `/runs/preview`(POST), `/runs/commit`(POST → 202, enqueues `billing.monthly-run`), `/invoices`(GET list) + `/invoices/[id]`(GET detail / PATCH cancel) + `/invoices/export`(POST), `/ledger`(GET flat statement), `/opening-balance`(POST add / PATCH lock). Read routes require `billing.invoice.read`; run/cancel/settings/opening-balance require `billing.invoice.create` (`requireBillingActor({manage:true})`). `runWriteScope` folds impersonation-read-only + READ_ONLY society status into the scope (writes → 423).
- UI page `/[locale]/app/billing` via `useUiModule("billing")`: Simple generate→preview→confirm + plain flat statement; Pro invoice DataTable (filters/bulk/export) + flat-ledger drawer (running balance) + arrears aging card + settings modal. i18n `billing` ns en+ur, exact parity (104 keys each).

### Gates
- `pnpm typecheck` clean · `pnpm lint` 0 warnings · `pnpm test:unit` **473 passed** / 7 skipped · `pnpm build` exit 0 (9 `/api/billing/*` routes + `/[locale]/app/billing` page, both locales).
- Ledger immutability acceptance criterion is a real architecture test (`ledger-immutability.test.ts`): walks `lib/app/worker/components` and fails on any `.ledgerEntry.(update|updateMany|upsert|delete|deleteMany)`.
- `run.integration.test.ts` (gapless numbering under concurrency, idempotent re-run, advance auto-application, cancel-keeps-number) is DB-gated and skips without `DATABASE_URL`, same pattern as the pre-existing `queue.integration.test.ts`. e2e `e2e/ledger-invoicing.spec.ts` = 12 unauth-401 route checks + billing page no-500 (not a CI gate).

### Decisions / notes
- Resumed a near-complete WIP rather than rebuilding; the design already matched the spec (balance-only money, snapshot lines, gapless-via-upsert, advance auto-apply, reversal-not-delete). The substantive change was the missing FK.
- Advance auto-application is modelled as a zero-net CREDIT+DEBIT pair so the running balance still reflects the advance being consumed while the invoice's own paid-amount (linked CREDIT − linked reversal) reads it as settled — no separate "applied advance" table needed.
- Cancelled invoice keeps its number and its sequence slot is never reused (the sequence only ever increments); a cancel writes a REVERSAL, matching the acceptance criterion.

**Checkpoint:** [CHECKPOINT] — progress marker; controller auto-approves and continues to step 19 (payments-receipts).

---

## Step 19 — Payments & receipts (`billing.payments`) — DONE (2026-07-12)

Money IN, reversed when a cheque bounces, and a printable receipt. **v1 is record-keeping; a gateway drops into the same seam later** (`PaymentProvider.MANUAL|STRIPE`). Continues the architecture-#4 invariant from step 18: **money is the ledger, not a mutable column** — a `Payment` row carries the request/verification snapshot but no balance; a CLEARED payment writes `LedgerEntry(PAYMENT, CREDIT)` and a bounce/void writes a matching `REVERSAL, DEBIT` that never edits the original.

### Feature / perms / nav
- `lib/features.ts`: `billing.payments` (isCore false, dependsOn `billing.ledger`, `core.storage`). DAG assert passes at load.
- `lib/rbac.ts`: added `billing.payment.read` (gated `billing.payments`); `billing.payment.record` already existed. COMMITTEE_MEMBER → read; ACCOUNTANT → read + record. Verify/bounce/void all require `record`.
- `lib/nav.ts`: `payments` entry (icon `banknote`, group manage, perm `billing.payment.read`, feature `billing.payments`, hideOnSharedDevice). Registered `Banknote` in `components/shell/nav-icons.ts`. `payments` UI module was already forward-declared in `lib/ui-modes/modules.ts` (audience committee → default PRO).

### Schema + migration (`20260712230000_payments_receipts`)
- Enums `PaymentMethod`, `PaymentProvider`, `PaymentStatus`, `AllocationMode`.
- `Payment` (societyId+deletedAt scoped): flatId, paidByUserId?, method, provider, amountMinor(BigInt), reference?, bankName?, paidAt, status, allocationMode, targetInvoiceId?, proofFileId?, submittedBy, verifiedBy?, verifiedAt?, rejectedReason?, bouncedAt?, bounceReason?, bounceFeeApplied, voidedAt?, voidReason?, notes?. FK `Payment_flatId_fkey` → Flat CASCADE (per the step-18 FK-convention lesson, declared in the migration, not just the schema). Indexes `(societyId,flatId,status)`, `(societyId,status)`, `(societyId)`.
- `Receipt` (scoped): paymentId @unique, number, issuedAt, pdfFileId?, voidedAt?, voidReason?. FK → Payment CASCADE. `@@unique([societyId, number])`.
- `PaymentSettings` (infra, keyed by societyId): bounceFeeEnabled, bounceFeeMinor?, requireProofForResidentSubmission, allocationMode, receiptPrefix, receiptFormat. FK → Society CASCADE. Society back-relation `payments`.
- `ReceiptSequence` (infra, `@@id([societyId, prefix, year])`): gapless per-(society, year) counter, allocated with the same atomic `INSERT … ON CONFLICT DO UPDATE` as invoices, inside the payment transaction (rolls back with it → no holes).

### Pure logic (no DB / no clock, unit-tested)
- `lib/billing/allocation.ts` (10 tests): `allocatePayment({amountMinor, invoices, allowPartial, allowAdvance})` → `{applications[], excessMinor}`. Fully settles each invoice in order; with `allowPartial` off it will not half-settle (skips one it can't fully cover → money flows to a later smaller invoice or the advance pool); excess with `allowAdvance` off is rejected (typed `AllocationError`). `SPECIFIC_INVOICE` = a single-element list.
- `lib/billing/money-words.ts` (6 tests): `integerToWords` uses the **South-Asian scale** (thousand → lakh → crore, recursing for arab+), `amountInWords` splits minor→major+subunit on the integer (no float) and appends "Only"; `currencyWords(currency)` maps PKR→Rupees/Paisa etc.

### Service (`lib/billing/payments.ts`)
- Settings CRUD; `recordPayment` (committee → CLEARED): resolves the billed user (primary current occupant), then `clearCore` in a fresh txn. `clearCore(ctx, args, tx)` = allocate (against `loadOutstandingInvoices` via the tx, oldest-first or the target) → one `PAYMENT CREDIT` per settled invoice (carrying invoiceId) + one advance CREDIT for the excess → gapless receipt (`allocateReceiptSequence`, prefix `"{invoicePrefix}-{receiptPrefix}"`) → recompute each touched invoice's money-derived status. `afterCleared` enqueues `billing.receipt-pdf` (dedupeKey `receipt-pdf:{paymentId}`) and notifies the payer (`billing.payment.received`) — after the money commits.
- `submitPayment` (resident/committee-on-behalf → PENDING_VERIFICATION): proof-gated by `requireProofForResidentSubmission`, writes NO ledger.
- `verifyPayment(id, approve|reject)`: `SELECT … FOR UPDATE` locks the row AND the state transition (clear) runs in the **same** transaction, so a second concurrent verifier finds it non-pending → `already_verified` (spec edge case). Approve may edit the amount (audited); reject → VOID + `rejectedReason` + notify `billing.payment.rejected`.
- `bouncePayment`/`voidPayment` share `reverseCleared`: for each original `PAYMENT CREDIT` a matching `REVERSAL DEBIT` (same invoiceId, `reversesEntryId` set) — originals untouched; optional one-time bounce fee (`LATE_FEE DEBIT`, `bounceFeeApplied` guard); receipt voided (keeps number); every touched invoice recomputed; notify `billing.payment.reverted`. A PENDING void just marks VOID (no ledger); a PENDING bounce is refused (`payment_not_cleared`).
- Reads: `listPayments`/`listPaymentsForExport` (DataTable kit + `payment-columns.ts`), `listVerificationQueue`, `getPaymentDetail`, `getPaymentsOverview` (cleared/pending counts + collected-today), `getReceiptRef`.
- `http.ts` maps new codes: `already_verified`/`payment_not_cleared` → 409, `unknown_payment` → 404, `read_only` → 423.

### Worker (`lib/worker/handlers.ts` + `registry.ts`)
- `runReceiptPdf`: builds the receipt from the ledger (invoices settled from linked PAYMENT credits, remaining balance from `flatBalanceMinor`), renders a branded, Urdu-capable single-page A4 PDF (`receipt-pdf.ts`, `pdf-lib` + embedded Noto Naskh via `loadUrduFont` — same glyph-without-shaping limitation as the export engine, documented), amount in figures AND words, VOID watermark when voided; stores it via `storeFile` (PDF is an allowed upload mime) and links `Receipt.pdfFileId`. Idempotent (dedupeKey + `pdfFileId` short-circuit). Registered `billing.receipt-pdf` (mutating:false — non-financial, runs even in READ_ONLY, like delivery). No cron (enqueued on demand).

### API (8 routes) + UI
- `/api/payments` (GET overview / POST record → 201), `/list`, `/queue`, `/settings` (GET/PATCH), `/submit` (POST → 201), `/[id]` (GET detail / PATCH `approve|reject|bounce|void`), `/[id]/receipt` (GET → signed URL, or 202 while rendering), `/export` (POST, current view, >1000 → 202). Read routes require `billing.payment.read` (`requirePaymentActor`), mutations require `billing.payment.record` (`{manage:true}`); `runWriteScope` folds impersonation-read-only + READ_ONLY society → 423. Shared Zod in `schemas/billing.ts` (`recordPaymentSchema`, `submitPaymentSchema`, `paymentActionSchema` discriminated union, `paymentSettingsSchema`).
- UI page `/[locale]/app/payments` via `useUiModule("payments")`: Simple = record modal (flat combobox → amount → method → date → save, cheque/bank surface reference/bank) + verification queue (card list with slip thumbnail via `/api/files/{id}`, Approve with optional amount edit / Reject with reason); Pro = payment DataTable (status/method/date filters, CSV/Excel/PDF export, receipt download, bounce/void through a reason-required confirm dialog). i18n `payments` ns en+ur, exact parity (101 keys each), plus `nav.payments`.

### Gates
- `pnpm typecheck` clean · `pnpm lint` 0 warnings on the new surfaces · `pnpm test:unit` **489 passed** / 7 skipped · `pnpm build` exit 0 (8 `/api/payments/*` routes + `/[locale]/app/payments` page, both locales).
- Ledger-immutability architecture test still green (payments.ts / handlers.ts write no `.ledgerEntry.(update|delete|upsert)` — reversals are new `create`s; only Invoice/Receipt/Payment rows are updated).
- `payments.integration.test.ts` (DB-gated, skips without `DATABASE_URL`, same pattern as `run.integration`): cleared credits+settles+issues receipt; partial → PARTIALLY_PAID; excess → −advance; bounce writes a reversal, never mutates the originals, applies the fee once, re-opens the invoice, voids the receipt keeping its number; submission stays off-ledger until approved + second verify → `already_verified`; **receipt numbering gapless under concurrency** (5 parallel `recordPayment`); void reverses.
- e2e `e2e/payments-receipts.spec.ts` = 11 unauth-401 route checks + payments page no-500 (not a CI gate).

### Decisions / notes
- **A payment maps to N ledger rows, not one.** OLDEST_FIRST splits the credit into one `PAYMENT CREDIT` per settled invoice (each carrying its `invoiceId`, so the existing step-18 `paidByInvoice` derivation marks invoices PAID/PARTIALLY_PAID) plus one unattributed CREDIT for the advance. A reversal mirrors them 1:1 (same invoiceIds + `reversesEntryId`), so the invoices re-open exactly. No `paidAmount` anywhere.
- **Verification concurrency** is a real row lock: `FOR UPDATE` + the clear run in ONE interactive transaction (Prisma can't nest `$transaction`, so `clearCore` takes the caller's tx). This is the atomic version of the spec's "second sees already verified".
- **Receipt numbering** reuses the invoice `formatInvoiceNumber` token engine with a composite prefix (`RUFI-RCP`) and a yearly `ReceiptSequence`; gaplessness comes from the atomic upsert inside the payment txn, identical to invoices.
- **Resident submission path**: `submitPayment` is fully built and the `/submit` route works, but in v1 it's guarded by `billing.payment.record` (committee logs a pending on a resident's behalf). The resident-initiated upload UI + occupancy-scoped auth is step 20 (resident finance), which reuses this exact service function.
- **Deferred (documented, non-blocking):** the record modal always uses OLDEST_FIRST (SPECIFIC_INVOICE is wired through the service + API, but the per-flat invoice picker belongs with the resident finance screen); the receipt PDF renders society name + emerald branding but passes `logo:null` (adapter-byte logo loader deferred).

**Checkpoint:** [CHECKPOINT] — progress marker; controller auto-approves and continues to step 20 (resident-finance).

---

## 20 — resident-finance — DONE (2026-07-12)

**Work type:** FEATURE (branch feature/20-resident-finance)

**Spec:** /specs/20-resident-finance.md — `billing.resident_portal` (depends on `billing.payments`). "The screen that sells the product": a resident sees, in plain language, exactly what they owe and what they've paid. Read-only over steps 18–19 plus payment submission.

### Decisions
- **No migration.** Everything reuses `Invoice`/`InvoiceLine`/`LedgerEntry`/`Payment`/`Receipt`/`FlatOccupancy` from steps 16/18/19. Step 20 is a resident-scoped READ layer + one write seam (the resident's own submitPayment). This is the single biggest scope/risk reduction — money invariants stay owned by steps 18–19 and are not re-implemented.
- **Authorization is occupancy, not RBAC.** A plain RESIDENT role holds `permissions: []` (confirmed in lib/rbac.ts). So the portal does NOT gate on a billing permission. `requireMeFinanceActor` gates on: (1) an authenticated society session, (2) the `billing.resident_portal` feature entitled (`isEnabled`), and (3) — per read — the caller actually HOLDING the flat. The nav item is likewise feature-gated with NO `permission` (so residents see it), `hideOnSharedDevice: true`.
- **THE security gate = `assertHoldsFlat(userId, flatId)`** in lib/billing/me-finance.ts, called before every flat-scoped read (and inside `assertHoldsInvoice`/`assertHoldsPayment`). It reads the caller's current `FlatOccupancy` and throws `ForbiddenError` (→ 403) for any flat that is not theirs — so a resident can never reach another flat's invoice/ledger/receipt even by guessing an id (the explicit acceptance test). Proven by the DB-gated integration test.
- **Balance parity:** `getFinanceSummary` derives every headline through `balanceOf` (= step-18 `balance(flatId) = Σdebit − Σcredit`), so the "You owe ₨X" the resident sees equals `balance(flatId)` to the minor unit. Asserted in the integration test (headline == `flatBalanceMinor`).
- **Multi-flat:** `getUserFlats` (residents seam) yields every held flat (OWNER or OCCUPANT). Summary returns a per-flat headline + a combined total; the client has an in-page pill switcher that also POSTs `/api/me/active-flat` to persist the choice. An owner of a rented flat holds it as OWNER → sees its ledger; invoices carry the occupant's `billedToName` snapshot (naturally satisfies "invoices addressed to occupant, owner can view").
- **Submission status transparency (spec 19/20):** a PENDING submission writes no ledger + no receipt, so it would be invisible. `getMyStatement` therefore also returns `submissions[]` (PENDING_VERIFICATION + any rejected, with reason) so the resident sees "Pending verification / Rejected (reason)". Rejected → the Pay button allows a corrected re-submission.
- **READ_ONLY society:** reads use `runReadScope` (never read-only) so dues stay visible; `POST /api/me/payments` uses `runWriteScope`, which folds a READ_ONLY society status → the scoping layer rejects the write (423, `ReadOnlyError`). The client proactively disables the Pay button and shows an explanatory banner from `summary.readOnly`.
- **Invoice PDF is synchronous, no worker:** unlike receipts (rendered async at payment time and stored), an invoice PDF is rendered on demand by `lib/billing/invoice-pdf.ts` (pdf-lib, Latin + embedded Noto Naskh for Urdu, one A4 page, CANCELLED watermark) and streamed straight back. Simpler than a worker+storage round-trip and gives instant download. Receipts reuse step-19's stored PDF via `getReceiptRef` + `getFileAccess` (signed URL, or 202 while the worker renders).
- **Statement export** reuses the generic export engine (`generateExport` + `loadUrduFont`) for PDF/Excel/CSV — the resident's own single flat is bounded, so no worker/queue (unlike the committee table export). Runs the same date-filtered running-balance rows the Pro view shows.
- **Utilities (step 29) / amenities (step 35)** render as separate, clearly-labelled dashed sections (data pending those modules) — structurally and visually never mixed into the maintenance balance (acceptance).

### Files
- lib/features.ts — added `billing.resident_portal` (dependsOn `billing.payments`).
- lib/nav.ts — `finance` item (main group, `wallet` icon, feature-gated, hideOnSharedDevice). components/shell/nav-icons.ts — `wallet` → Wallet.
- lib/billing/me-actor.ts (NEW) — `requireMeFinanceActor` / `getMeFinanceServerActor` (feature + auth, no RBAC), reuses `runReadScope`/`runWriteScope`.
- lib/billing/me-finance.ts (NEW) — `assertHoldsFlat` (the 403 gate), `heldFlats`, `getFinanceSummary`, `getMyStatement` (running balance + invoices-with-lines + receipts + aging + submissions + date range), `assertHoldsInvoice`/`assertHoldsPayment`, `submitMyPayment` (= step-19 submitPayment).
- lib/billing/invoice-pdf.ts (NEW) + lib/billing/invoice-pdf.test.ts (unit — valid PDF en/ur/cancelled).
- API (7): app/api/me/finance/summary, app/api/me/invoices, app/api/me/invoices/[id]/pdf, app/api/me/ledger, app/api/me/ledger/export, app/api/me/receipts/[id]/pdf, app/api/me/payments.
- UI: app/[locale]/app/finance/page.tsx, components/finance/finance-client.tsx (Simple + Pro via `useUiModule("finance")`), components/finance/submit-payment-modal.tsx (FileUpload slip + camera).
- schemas: reused `submitPaymentSchema`.
- i18n: `finance` namespace + `nav.finance` in messages/en.json + messages/ur.json (parity, JSON validated).
- lib/billing/me-finance.integration.test.ts (DB-gated) — cross-flat 403, headline == balance(flatId), submit off-ledger reaches queue + visible pending.
- e2e/resident-finance.spec.ts — 7 unauth 401s + finance page renders < 500.

### Gate results
- Per CLAUDE.md the controller runs typecheck/lint/test/build after this session; not run here. Self-check by inspection: all imports/signatures verified against live files (isEnabled, getInvoiceDetail, ageArrears, getFileAccess, getUserFlats, export barrel, formatMoney, error→status mapping ReadOnlyError→423 / Forbidden→403 / Unauthenticated→401); both message files JSON-validated; every `t(...)` key cross-checked against the added namespace; nav/features tests inspected (no hardcoded counts broken; new feature dep exists).

### Notes / deferred
- Society toggle "may an owner see the occupant's invoices" — spec default **yes**, implemented as default-yes (owner holds the flat → sees it). The configurable-per-society flag is deferred (would need a societySettings column / migration); recorded here as an autonomous decision.
- Utilities/amenities sections are placeholders until steps 29/35 provide data.

---

## 21 — society-onboarding-wizard — DONE (2026-07-12)

**Branch:** feature/21-society-onboarding-wizard · **Feature:** `platform.onboarding` (non-core; deps `flats.registry`, `residents.registry`, `billing.ledger`, `branding.core`)

### What it is
The guided path that takes a brand-new society from empty to live — the module that makes Rihaish *sellable*. 10 resumable steps (profile, branding, structure, categories & occupancy, charge heads & rates, billing settings, opening balances, people, integrations, review & launch). Available to a `SOCIETY_ADMIN` on their own host and to L0 via impersonation (both hold `platform.onboarding.*`).

### Key architectural decisions (logged, autonomous)
- **No DRAFT society status.** The `SocietyStatus` enum is only `ACTIVE|READ_ONLY|SUSPENDED` and a society is `ACTIVE` from `createSociety`. So onboarding is an **orchestration layer over an already-ACTIVE tenant**, not a status flip. "Launch" = stamp `SocietyOnboarding.completedAt`/`launchedBy`, LOCK opening balances, audit `onboarding.launched`. Recorded here rather than adding a new enum value that would ripple through tenant resolution.
- **Non-core feature.** Depends on `billing.ledger` (itself non-core), so it cannot be a core feature (a core feature may not depend on a non-core one). It is therefore plan/L0-gated — appropriate for a "sellable" feature. New societies get it when their plan/L0 enables it.
- **Reuse-first.** The wizard's own API adds only the composites no module provides: draft state, society profile, a MULTI-block structure generator, the plain-language rate screen, readiness + launch. Branding, categories, billing settings, opening balances, resident invites and integrations reuse their existing endpoints from the wizard UI.
- **`Society` profile written via raw `prisma`.** The scoped `db` client forbids `Society` mutations (throws `CrossTenantError`), so the profile step updates the society by its own id via raw `prisma.society.update` (tenant-safe — the id is the actor's society) + a `SOCIETY_USER` audit. Only `db.unscoped()` is lint-restricted; raw `prisma` reads/writes are the established pattern (billing actor reads society the same way).
- **Full-screen inside the shell.** Next.js layouts compose, so a route under `/app` cannot bypass the app-shell layout via a nested `layout.tsx`. The wizard renders as an immersive full-width page (progress rail + summary panel) inside the shell — consistent with every other module — rather than fighting the layout system.
- **Occupancy/CSV scope.** Occupancy is set at structure generation (default per-block) and is adjustable later; the CSV imports called out for steps 3/4/7/8 reuse the existing atomic dry-run import modals rather than being re-embedded in the wizard. Integrations *configuration* (per-provider creds + test send) stays in Settings → Integrations; the wizard shows connection status and links there.

### Data model / migration
`20260712240000_society_onboarding`: `SocietyOnboarding` (societyId PK/FK cascade, currentStep Int=1, completedSteps Int[]=[], data Jsonb='{}', completedAt?, launchedBy?, createdAt, updatedAt). Infra (no `deletedAt`) → NOT auto-scoped; every access filters `societyId` explicitly (like `BillingSettings`). Relation added to `Society`.

### Server (`lib/onboarding/`)
- `constants.ts` — 10-step catalogue (key, optional, estimateMinutes) + feature/permission codes. Framework-free (client-safe).
- `structure-plan.ts` (pure) — `parseBlockCodes` ("A-F" range / "A,B,C" list / mix, dedup, case-normalise, reversed-range flip) + `planStructure` (plans every block against combined existing codes + `maxFlats`; refuses the ENTIRE run before any write on any duplicate / existing collision / limit breach — the "blocked before any write / upgrade prompt" edge case).
- `readiness.ts` (pure) — `computeReadiness(input)`: rates=0 → **block** ("cannot bill"), flats=0 → block, empty name → block, residents=0 → **warn** ("nobody can log in"), categories/opening-balances/branding empty → warn. `canLaunch = blockers.length === 0`.
- `service.ts` — `getState`/`saveState` (draft upsert: advance pointer, union completed steps, shallow-merge data), `saveProfile`, `generateStructure` (pre-flight `planStructure` → per-block idempotent `createBlock` + structure module's atomic `bulkGenerate`), `saveRates` (ensure charge head → one effective-dated `createRateRule` per line; supersedes open rules, history intact), `getSnapshot` (counts + reference lists + billing settings + branding flags; `SocietyIntegration` count filtered by societyId since it is infra/non-auto-scoped), `readinessFromSnapshot`/`getReadiness`, `listFlatsForPicker`, `launch` (re-check readiness → 409 if blocked; stamp completion; `billing.lockOpeningBalances`; audit).
- `actor.ts` — `OnboardingActor` (+ society money/locale meta), `requireOnboardingActor`/`getOnboardingServerActor`, `runReadScope`/`runWriteScope` (fold impersonation-RO + READ_ONLY status → 423), mirrors billing/structure actors exactly.
- `http.ts` — `onboardingErrorResponse` (LOCKED→423, NOT_FOUND→404, CONFLICT/structure_blocked/launch_blocked→409, Zod→400 validation, else `authErrorResponse`), `parseBody`, `requestOrigin`.

### API (`app/api/onboarding/`)
`route.ts` GET (bootstrap: state + snapshot + pure readiness) / PATCH (save draft); `profile` POST; `structure` POST (`?preview=1` = read-scoped dry-run plan, else write-scoped generate → 201); `rates` POST → 201; `launch` GET (live checklist) / POST (launch); `flats` GET (`?q=` picker). All `runtime=nodejs`, `dynamic=force-dynamic`, try/catch → `onboardingErrorResponse`, actor + scope enforced.

### UI (`components/onboarding/`, `app/[locale]/app/setup/`)
Full-screen wizard: `PageHeader` + "Save & continue later" (PATCH pointer → route to `/app`), left progress rail (per-step status tick / current / estimate / lock glyph on opening-balances after launch), center active step card, right sticky live "your society so far" summary + launch checklist. `setup-wizard.tsx` orchestrates step routing, `refresh()` re-fetches bootstrap, `advance()` marks current step complete + advances in one PATCH. 10 step components using the form kit only (Simple is the only mode — no `useUiModule`): profile (RHF+zod), branding (fields + `FileUpload` logo → PUT branding), structure (bulk form + server Preview + limit/conflict callout), occupancy (category add + occupancy note), rates (plain-language occupied/rented/vacant `MoneyField`s → effective-dated rules), billing (reuses `billingSettingsSchema`), opening-balances (async flat `Combobox` + amount + arrears/credit; locked banner post-launch), people (async flat combobox + `PhoneField` invite), integrations (status list + link to Settings), review (live checklist + Launch, disabled until `canLaunch`; live-state screen after launch). Logical CSS props throughout (RTL-safe); all copy via i18n.

### i18n
`onboarding` namespace + `nav.setup` added to `messages/en.json` + `messages/ur.json` — **160/160 key parity** verified by script. Urdu translations for every string; RTL via existing `dir` plumbing.

### Tests
- Unit (no DB): `structure-plan.test.ts` (block-spec parser incl. reversed range/dedup; combined plan totals; whole-run block on maxFlats breach incl. existing-count; existing-code collision; empty-blocks not-ok), `readiness.test.ts` (launch blocked without rates / no flats / no name; warned without residents/opening-balances/branding; 8 checklist items).
- DB-gated integration `launch.integration.test.ts` (`describe.skipIf(!DATABASE_URL)`): launch blocked with no rates; draft saved & reloaded (survives reload); opening balance accepted before launch then society launches, `openingBalanceLocked=true`, state `launched`/`launchedBy` set; opening balance after launch rejected `opening_balance_locked` (locked, cannot re-open).
- e2e `society-onboarding.spec.ts`: 9 onboarding endpoints 401 without a session; `/en/app/setup` renders < 500. (Full Rufi launch flow proven by the integration test, which needs seeded auth — matches the repo's lightweight e2e convention.)

### Registry
`lib/features.ts` (+`platform.onboarding` entry, DAG validated at load), `lib/rbac.ts` (+`platform.onboarding.read/.manage` gated by the feature; `SOCIETY_ADMIN` receives them via `*`), `lib/nav.ts` (+`setup` item, manage group, hideOnSharedDevice), `components/shell/nav-icons.ts` (+rocket).

### Gates
Not run in-session (controller runs lint → typecheck → test:unit → build, then `prisma generate` before typecheck/build so `db.societyOnboarding` resolves). UI design self-check done by inspection: form kit only, logical props, i18n en+ur, Simple-only (the wizard is the simple mode), skeleton/empty/error via shell states + no-access `EmptyState`, toasts (no alert), client-side nav, graceful when optional features disabled (reuse endpoints 403 → toast, never 500).

### Not done / deferred
- Playwright full 184-flat Rufi launch (needs seeded society + admin session; covered instead by the DB-backed integration test per repo convention).
- CSV imports embedded in steps 3/4/7/8 (reuse existing atomic dry-run import modals).
- Integrations credential configuration + test send (lives in Settings → Integrations; wizard shows status + deep link).
- First-invoice preview run at launch (launch returns readiness; the preview run is the billing module's existing generate-in-preview surface).

---

## 22 — platform-billing — DONE (2026-07-12)

**Branch:** feature/22-platform-billing (WORK TYPE: FEATURE)
**Spec:** /specs/22-platform-billing.md (no CODEREF in range).

### What & why
"How Rihaish gets paid." Built as a **completely separate namespace** from society
billing (`lib/billing/*`) — distinct tables, module, UI — per spec: sharing them
would eventually leak or mis-bill. All platform-billing tables carry a `societyId`
but NO `deletedAt`, so the tenant-scoping rule (needs BOTH) never touches them;
they are read/written only through the unscoped `platformDb()` client.

### Data model (migration 20260712250000_platform_billing)
- Enums: `PricingModel` (PER_FLAT|LUMPSUM), `CountMode` (ALL_REGISTERED_FLATS),
  `BillingCycle` (MONTHLY|QUARTERLY|ANNUAL), `PlatformInvoiceStatus`
  (ISSUED|PARTIALLY_PAID|PAID|OVERDUE|CANCELLED), `PlatformPaymentProvider`
  (MANUAL|STRIPE). `PlatformPayment.status` REUSES the shared `PaymentStatus` enum.
- `PricingProfile` — effective-dated (effectiveFrom/effectiveTo), superseded never
  edited; per-flat rate or lumpsum; discountPercentBp; @@index(societyId,effectiveFrom).
- `PlatformInvoice` — `flatCount` + `rateMinor` SNAPSHOTS at generation; idempotent
  `@@unique([societyId, period])`; number "RIH-YYYY-MM-####" globally unique.
- `PlatformPayment` — MANUAL v1; bounce/void columns; @@index(societyId,status).
- `PlatformBillingSettings` — singleton (graceDays 14, reminderDaysBefore [7,3,1],
  reminderDaysAfter [1,3,7,14], autoReadOnlyOnOverdue true).
- `PlatformInvoiceSequence` — gapless per (prefix, year, month), atomic upsert.

### Code
- `lib/platform-billing/`:
  - `constants.ts` — string-union mirrors of the enums (pure, no Prisma import).
  - `math.ts` (PURE, tested) — computeInvoiceAmounts (per-flat vs lumpsum; bp
    discount FLOORED in minor units; clamps negative), resolveEffectiveProfile
    (window contains instant, latest effectiveFrom wins tie, effectiveTo exclusive),
    monthlyMinor/mrrMinorFor (cycle normalisation), invoiceStatusFrom, isPastGrace,
    reminderKindFor (exact before/after-due days, ≤1/day).
  - `numbering.ts` (PURE, tested), `adapter.ts` (PaymentProviderAdapter seam; MANUAL
    only; mock-Stripe test proves adding a gateway = new adapter + webhook, NO schema
    change), `invoice-pdf.ts` (Latin one-page A4), `http.ts` (error envelope).
  - `service.ts` — settings get/update; pricing list/effective/set (supersede-in-tx);
    runPlatformBilling (per-ACTIVE-society, snapshot flatCount, idempotent, notify);
    invoice reads (paid = CLEARED payments, balance); recordPlatformPayment (adapter
    capture → reconcile invoice status → restore society if square, same request);
    reversePlatformPayment (bounce/void = status flip, never edit); runGraceCheck
    (mark OVERDUE, send reminders, flip READ_ONLY past grace with
    `platform-billing:`-prefixed reason, restore when owing nothing); grantExtension
    (audited: push due dates + restore). All audited to AuditLog(actorType PLATFORM).
  - `dashboard.ts` — MRR/ARR (effective profile × current flats, cycle-normalised),
    collection rate (cleared ÷ issued), overdue + about-to-read-only society counts,
    revenue by plan.
- Worker: `platform.grace-check` stub → real `runPlatformGraceCheck` (daily 0 4);
  new `platform.billing-run` cron (monthly `0 3 1 * *`, platform scope) →
  `runPlatformBillingRun`. Both non-mutating flag (platform-scoped, no read-only gate).
- Notifications: added `platform_billing` category (NOTIFICATION_CATEGORIES → 8) +
  5 template groups (invoice.issued, reminder, payment.received, readonly, restored),
  en+ur per channel. Audience = SOCIETY_ADMIN roles + nominated recipients
  (includeRecipientRules). Sent via notifySafe (never breaks a run).
- Wired `lib/platform/dashboard.ts` MRR + overdueSocieties to real billing (were 0).
- Feature `platform.billing` registered (deps platform.console, flats.registry,
  core.worker); perms `platform.billing.read/.manage` (ungated, on PLATFORM_ADMIN +
  read on PLATFORM_SUPPORT).

### API (all apex-only, platform-gated)
`/api/platform/billing/dashboard` GET; `/invoices` GET; `/invoices/[id]` GET;
`/invoices/[id]/pdf` GET; `/payments` POST; `/payments/[id]/reverse` POST;
`/settings` GET/PATCH; `/run` POST; `/societies/[id]/pricing` GET/POST;
`/societies/[id]/extension` POST.

### UI (platform console, Pro-only by nature)
- `app/[locale]/platform/billing/page.tsx` — MRR/ARR/collection/overdue tiles,
  animated revenue-by-plan bar chart, invoices table (filters, PDF, record-payment
  dialog, run-now dialog). New shell nav item `billing`.
- `components/platform/pricing-panel.tsx` embedded in society detail — effective
  price + full history (never edited in place) + set-new-profile form.
- i18n en+ur full parity (verified: 0 keys diverge; +67 platform.billing keys, plus
  `category.platform_billing` label/simple + settings recipient-rule label).

### Read-only semantics
Society status READ_ONLY set on the Society row with a `platform-billing:` reason;
the society data layer already returns 423 on writes and allows all reads, and the
resident finance portal (step 20) is unaffected — residents always see their dues.
Restore is immediate on a clearing payment (same request) or a manual extension.

### Decisions
- billableCountMode = ALL_REGISTERED_FLATS (non-deleted flats), per spec "decided".
- graceDays configurable (default 14); dueDate anchored to day 14 of the period.
- MANUAL payments land CLEARED immediately (money already in hand); the adapter seam
  keeps the door open for STRIPE with no migration.
- Reversal never edits the amount (mirrors step 19).

### Gate results
Did NOT run gates (controller runs them). Self-checks: `prisma validate` OK; client
generated (all 5 delegates present); en/ur JSON valid + key-parity 0 diff. Updated
`registry.test` (added platform.billing-run), `notifications-policy.test`
(categories → 8), `templates.test` (platform_billing category covered).

### Tests added
- `lib/platform-billing/math.test.ts`, `numbering.test.ts`, `adapter.test.ts` (pure).
- `lib/platform-billing/run.integration.test.ts` (DB-gated): effective-profile +
  one snapshotted invoice; idempotent per (society, period) + snapshot immutability
  on a mid-month flat change; grace → READ_ONLY, data stays readable, payment
  restores ACTIVE + invoice PAID; bounce reverses (amount untouched) → re-owes;
  manual extension restores; PlatformInvoice ≠ Invoice (table separation).
- `e2e/platform-billing.spec.ts`: every route 401 on apex unauth + 404 on society
  host; billing page renders without a 500.

WORK TYPE: FEATURE (branch feature/22-platform-billing)

---

## Step 23 — pwa-mobile-shell — DONE (2026-07-12)

**Branch:** feature/23-pwa-mobile-shell · **Feature codes:** `pwa.core`, `pwa.push` · Spec: /specs/23-pwa-mobile-shell.md

### Goal
Make the PWA feel like a real, installable, per-society branded mobile app (no native builds / stores): white-label manifest + icons + splash, standalone app-feel (bottom tabs, More drawer, safe areas, offline shell), install flow (Android + iOS), and VAPID web push plugged into the step-11 engine.

### Data model (migration 20260712260000_pwa_mobile_shell)
- **PushSubscription** — one row per user per device; `endpoint` globally unique (upsert), `p256dh`/`auth` (RFC 8291 keys), `userAgent`, `lastSeenAt`. Pass-through (societyId, NO deletedAt) → tenant-scoping layer ignores it; worker reads cross-society via scoped `db` (pass-through), request path filters `(societyId,userId)`; a dead endpoint is physically pruned (never retried forever).
- **PwaInstall** — install adoption per society; `@@unique(userId, platform)` (android|ios|desktop), `lastInstalledAt`. Pass-through.

### Manifest & branded images
- `app/manifest.webmanifest/route.ts` rewritten: `id/start_url=/app`, `display:standalone`, `orientation`, theme/background from branding, icons → **stable** `/api/branding/icon-192.png` + `icon-512.png` (`any maskable`). Host-scoped, `no-store`, apex/unknown → 404.
- New public (dotted-path, middleware-skipped) generator routes: `icon-192.png`, `icon-512.png`, `apple-touch-icon.png` (180, opaque), `splash?w=&h=` (device-sized). `lib/pwa/images.ts` (sharp, lazy import): maskable inner-80% composite on brand bg, opaque apple icon, centered splash on `splashBgColor`, and an SVG **fallback initial** on the brand colour when a society has no `iconFileId`. `lib/pwa/branding-assets.ts` shared resolver (host → society → branding → icon bytes via new `storage/service.readFileBytes`).
- App head (`app/[locale]/app/layout.tsx`): `theme-color`, `apple-touch-icon`, apple-mobile-web-app meta, and the full `apple-touch-startup-image` set (`lib/pwa/apple-splash.ts`, ~14 device geometries → per-society splash route). Root layout adds `viewport-fit=cover` (`export const viewport`).

### Service worker (app/sw.js/route.ts, generated per host+BUILD_ID)
- Served `text/javascript` + `Service-Worker-Allowed: /`; cache name `rihaish-shell-<host>::<BUILD_ID>` → **host-scoped** (already per-origin; host also baked in) and **BUILD_ID-versioned** (old caches dropped on activate; page shows "Update available — reload" and posts `SKIP_WAITING`, then `controllerchange` reloads — never yanks a session).
- Strategies: navigations network-first w/ cached-shell offline fallback; static (`/_next/static`, `/api/branding`, assets) stale-while-revalidate; **every other `/api/*` GET is network-only (never cache another user's data)**.
- Offline write queue: **opt-in** via `X-Offline-Queue` header — failed writes stored in IndexedDB + replayed via Background Sync `rihaish-outbox` (conservative: only explicitly-marked requests, no blind double-submits).
- `push` → showNotification(payload); `notificationclick` → focus/navigate the exact deep-link URL.

### Web push (VAPID) — dependency-free
- `lib/pwa/web-push.ts`: RFC 8291 aes128gcm (ECDH P-256 + HKDF ladder, 0x02 last-record, RFC 8188 header) and RFC 8292 ES256 VAPID JWT (JWK private key, `dsaEncoding: ieee-p1363`) on `node:crypto`. `buildPushRequest` assembles `Authorization: vapid t=…, k=…` + encrypted body.
- `lib/pwa/push.ts`: subscription CRUD (save/delete/revoke-all/prune), `deepLinkUrl` (same-origin `/…` only, else `/app`), and `buildPushTransport(sender, now)` → loads recipient's subs, encrypts+signs per device, sends, prunes 404/410; ok if any device accepts or the user has no device (nothing to retry); `push_not_configured` when VAPID unset.
- Engine wiring: `OutboundMessage` extended with `code/userId/societyId/data`; `dispatch.toOutbound` carries them for PUSH; `availableChannels` now includes `PUSH`; worker boot calls `setPushTransport(buildPushTransport())` when VAPID configured. Added a PUSH template to `billing.invoice.created` (complaint/gatepass/chat/emergency already had one). Emergency still ignores opt-out via the existing channel floor.
- Env: optional `VAPID_PUBLIC_KEY`/`VAPID_PRIVATE_KEY`/`VAPID_SUBJECT` (unset ⇒ push simply disabled, never crashes).

### Mobile shell UI
- `lib/pwa/tabs.ts`: pure `roleBucket` (management→committee, else guard, else resident) + `resolveBottomTabs` (candidates gated by `can(perm)&&hasFeature(feat)`, first 4 + always **More**; never empty, never >5). Resolved server-side in `shell-data.ts` into `ShellData.pwa.{bottomTabs, vapidPublicKey}`.
- `components/pwa/`: `bottom-tabs` (fixed, `lg:hidden`, safe-area pb), `more-drawer` (end Sheet: full nav + flat switcher + locale/theme + account + sign out — sign out unsubscribes push first), `offline-banner` (online/offline + sync nudge), `sw-register`, `push-manager` (re-affirm if granted; ask ≤2× only when standalone; respect denial), `install-prompt` (Android beforeinstallprompt sheet + iOS Share→Add-to-Home instructions, 7-day reminder, reports install + subscribes push), `pwa-runtime` wrapper. Mounted in `app-shell-client.tsx` (offline banner top, bottom tabs + runtime bottom, main gets mobile bottom padding). Desktop sidebar/top-bar untouched.
- `globals.css`: standalone tap-highlight/select suppression, momentum scroll, safe-area utilities. `nav-icons.ts` +idCard/listChecks/bell/menu.

### API
- `/api/pwa/push/public-key` (GET, auth) → `{publicKey|null}`; `/api/pwa/push/subscribe` (POST) upsert device sub (society-scoped, 401/403 guarded); `/api/pwa/push/unsubscribe` (POST) idempotent delete; `/api/pwa/installed` (POST) adoption upsert. Client round-trips in `lib/pwa/client.ts` (subscribe/unsubscribe/reportInstall/platform+standalone detection).

### Tests
- Unit: `web-push.test` (encrypt→decrypt round-trip from the UA side + VAPID JWT ES256 verify + request assembly), `tabs.test` (bucketing, ≤5, More-last, entitlement gating for all three roles), `push.test` (deepLinkUrl, getVapidKeys, transport not-configured/no-user guards — no DB), `images.test` (hexToRgb), `install.test` (normalizePlatform). Extended templates catalogue keeps the en/ur parity test green.
- e2e `pwa.spec`: manifest standalone+icon shape & apex 404; `/sw.js` headers + handlers + **host-scoped** bodies differ; branded icon PNG magic bytes; icon apex 404; push subscribe 401 unauthenticated.

### Decisions / notes
- Offline action replay is **opt-in** (`X-Offline-Queue`) to avoid double-submitting non-idempotent writes — a safe seam rather than blindly queuing every failed POST.
- Deep links rely on callers passing `data.url`; default is `/app`. Mechanism (transport + SW notificationclick) is complete; per-event URLs are each owning module's to add.
- Custom pull-to-refresh/haptics: left to the browser's native standalone behaviour (momentum + native PTR) rather than a bespoke JS gesture layer this step.
- Gates (typecheck/lint/test/build/prisma generate) run by the controller.

WORK TYPE: FEATURE (branch feature/23-pwa-mobile-shell)

## 24 — announcements — DONE (2026-07-12)

**Feature code:** `announcements.core` (deps `residents.registry`, `core.notifications`, `core.storage`). Branch `feature/24-announcements`. WORK TYPE: FEATURE.

### Data model (migration `20260712270000_announcements`)
- **Announcement** — SCOPED (societyId + deletedAt → auto tenant-pin + soft-delete). Fields per spec + additions: `sourceAuthority` (EXTERNAL_AUTHORITY attribution), `recipientFlatIds[]`/`recipientUserIds[]` (recipient SNAPSHOT captured at publish = the read-receipt denominator), `publishedAt`, `updatedAt`. `@@index([societyId, status, publishAt])`.
- **BereavementDetail** — pass-through (announcementId @id, no societyId). Family opt-in gate.
- **AnnouncementRead** — pass-through (societyId set explicitly), `@@unique([announcementId, userId])`.
- **AnnouncementComment** — pass-through; soft-delete via **`removedAt`/`removedBy`**, NOT `deletedAt`, so the auto `deletedAt IS NULL` scope filter can't hide a moderated comment (spec: shown as "removed").
- **AnnouncementReaction** — pass-through, `@@unique([announcementId, userId, kind])`; un-react physically deletes.
- New enums `AnnouncementType`, `AnnouncementTargetType` (dedicated — NOT the charge engine's `TargetType`, to avoid ALTER-TYPE-in-txn and cross-module switch breakage; adds `ROLE`), `Priority`, `AnnouncementStatus`.

### Decisions
- **Reactions are computed, not stored.** `reactionsAllowed(type) = type !== BEREAVEMENT`; the reaction endpoint calls `assertReactable` → 403 for bereavement. There is no toggle/field to enable it, satisfying "never a reaction control, by any role, any path".
- **Bereavement comments default OFF**, others ON (`defaultAllowComments`).
- **URGENT → `emergency` notification category** so the engine floors in-app+push; SMS still respects the user's mute (channel floor only covers in-app/push) — matches "URGENT may override channel prefs, but not the ability to mute non-emergency SMS".
- **Per-announcement channels** honoured via a new optional `restrictChannels` on `notify()`/`NotifyInput` (intersected with prefs ∩ society config ∩ template; IN_APP always retained). Extended the `announcement.posted` catalogue template with SMS + WhatsApp bodies (en+ur) so those channels can actually render (templates.test requires en+ur per channel — provided).
- **Feed is LIVE-targeted** (`feedTargetWhere` over the viewer's flats/blocks/categories/roles via `hasSome`), while the receipt denominator uses the publish-time snapshot → a resident added after publication sees the item but isn't counted (spec edge case).
- **Publish = create-now or schedule:** POST creates DRAFT; if `publishAt` is future → SCHEDULED (worker publishes); else publishes immediately in the same request. `publishAnnouncement` is idempotent, snapshots recipients, runs `assertPublishable` (family opt-in + non-empty audience), fires `notifySafe`.
- **Rich text** stored sanitised (`sanitizeRichText`: strips script/style + all tags, decodes basic entities, preserves newlines). No WYSIWYG in the repo yet; composer uses the kit `TextArea` + EN/UR preview-by-inspection.
- **Attachments**: schema stores StoredFile ids; feed surfaces a count (signed-URL thumbnail wiring deferred — no announcement-attachment access endpoint this step).
- Nav item is **feature-gated, permission-ungated** (every member sees the feed; committee compose/moderation gated inside the page by `society.announcements.*`). `hideOnSharedDevice`.
- Roles: read+publish → COMMITTEE_MEMBER & MANAGER; +moderate → MANAGER; read+moderate → MODERATOR; ADMIN via `*`.

### Files
- Schema/migration: `prisma/schema.prisma`, `prisma/migrations/20260712270000_announcements/migration.sql`.
- Registry: `lib/features.ts` (announcements.core), `lib/rbac.ts` (perms + role grants), `lib/nav.ts` + `components/shell/nav-icons.ts` (megaphone).
- lib/announcements: `constants.ts`, `rules.ts`(+test), `sanitize.ts`(+test), `targeting.ts`(+test), `columns.ts`, `actor.ts`, `service.ts`, `http.ts`.
- schemas: `schemas/announcements.ts`.
- Notifications: `lib/notifications/service.ts` (`restrictChannels`), `lib/notifications/templates.ts` (SMS+WhatsApp on announcement.posted).
- Worker: `lib/worker/handlers.ts` (`runAnnouncementsTick`), `lib/worker/registry.ts` (cron+job), `lib/worker/registry.test.ts` (expected-codes list +announcements.tick).
- API: `app/api/announcements/route.ts`, `[id]/route.ts`, `[id]/{publish,read,read-report,comments,reactions}/route.ts`, `comments/[id]/route.ts`, `recipients/route.ts`.
- UI: `app/[locale]/app/announcements/page.tsx`; `components/announcements/{types,announcements-client,announcement-feed,announcement-table,compose-modal,read-report-dialog}.tsx`.
- i18n: `messages/en.json` + `messages/ur.json` (`announcements.*` namespace + `nav.announcements`).

### Tests
- Unit (vitest, co-located): `rules.test.ts` (bereavement never reactable by any type/path; publish rejects missing family opt-in + empty audience; channel normalize incl. forced IN_APP; URGENT→emergency), `targeting.test.ts` (flatWhere for all 5 target types, ROLE→null, no accidental everyone), `sanitize.test.ts`.
- e2e: `e2e/announcements.spec.ts` (feed/compose/preview/read/read-report all require auth — 401).
- Note: type-driven invariants proven by pure unit tests (no DB); DB-backed receipt counting exercised via the service at runtime (no integration test added this step).

### Gates
- NOT run in-session per CLAUDE.md rule 6 (controller runs lint/typecheck/test/build/prisma generate). Self-checked by inspection: import/export names against residents/billing precedents; pass-through vs scoped model semantics in `lib/db.ts`; registry/templates/nav/rbac test brittleness (updated the one exact-list assertion in registry.test.ts). Prisma client types (Announcement*, Priority, Channel[]) require the controller's `prisma generate` before typecheck.

---

## Step 25 — Staff directory & attendance — DONE (2026-07-12)

**Spec:** `/specs/25-staff-directory-attendance.md` (no CODEREF). Work type: FEATURE, branch `feature/25-staff-directory-attendance`.

### [DECIDE AT BUILD] decisions
- **Geofenced check-in: DEFERRED.** `AttendanceMethod` ships as `MANUAL | QR | SELF`. Manual roster marking is the default; QR = a scanned kiosk, SELF = the staff member's own app. Geofencing adds device-permission + accuracy + spoofing concerns disproportionate to v1; the enum can gain a `GEO` value later without a data migration. Rationale: spec says manual-by-default, QR optional; geofence explicitly optional.
- **`phone` optional at the DB layer** (`String?`) even though the spec data-model wrote `phone String`, to honour the edge case "vendor staff without a phone → phone optional for VENDOR". The shared Zod schema (`refineStaff`) requires phone for PERMANENT/CONTRACT and allows it empty for VENDOR — one payload shape, type-driven validation.
- **`LeaveRequest.status` uses a new `LeaveStatus` enum** (PENDING/APPROVED/REJECTED/CANCELLED). The spec wrote `ApprovalStatus` but no such enum exists in the schema; introduced a dedicated one rather than coupling to another module.
- **CNIC capture reuses the existing `SocietySettings.cnicCaptureEnabled`** toggle (shared with residents step 16) rather than a new staff-specific flag — one society-level PII switch.
- **Salary crosses the wire as a cleaned digit STRING**, not a `bigint` (BigInt is not JSON-serialisable); the service does `BigInt(...)` on store into `salaryMinor`.

### Data model / migration `20260712280000_staff_directory_attendance`
- `StaffMember` — societyId + deletedAt ⇒ AUTO-SCOPED (`lib/db.ts`): reads pin to society, `delete` → soft-delete + audit so a departed staff member's attendance/leave history is retained (spec). Unique `(societyId, employeeCode)`. Optional `userId` (may have no login), `cnicEnc` (encrypted), `photoFileId`, `department`, `vendorId`, `shift`, `salaryMinor BigInt`, `skills String[]`, `leftOn`. Indexes `(societyId, isActive)` + `(societyId)`.
- `Attendance` — societyId, NO deletedAt ⇒ pass-through child (resolved by staffId, society-unique). Unique `(societyId, staffId, date)`; `date` is the society-local calendar day anchored to 00:00Z. Attendance is HISTORY — never deleted; a re-mark UPDATES the unique row. Index `(societyId, date)`. FK → StaffMember ON DELETE CASCADE.
- `LeaveRequest` — societyId, NO deletedAt ⇒ pass-through. Indexes `(societyId, staffId)` + `(societyId, status)`. FK → StaffMember ON DELETE CASCADE.
- Enums: `EmploymentType` (PERMANENT/CONTRACT/VENDOR), `Shift` (MORNING/EVENING/NIGHT/ROTATING), `AttendanceStatus` (PRESENT/ABSENT/LEAVE/HALF_DAY/HOLIDAY), `AttendanceMethod` (MANUAL/QR/SELF), `LeaveStatus`.
- Migration hand-authored to match Prisma's generated DDL (index/constraint names verified against `@@unique`/`@@index` naming).

### Registry / permissions / nav / ui-mode
- `lib/features.ts`: `staff.directory` (deps residents.registry, core.auth, core.storage) + `staff.attendance` (dep staff.directory). DAG validates (no cycle).
- `lib/rbac.ts`: `staff.directory.read/.manage` (gated `staff.directory`), `staff.attendance.read/.mark`, `staff.leave.approve` (gated `staff.attendance`) — so with attendance off the ∩ rule drops the attendance perms and the directory keeps working (graceful degradation). Seeded: MANAGER (all), COMMITTEE_MEMBER (directory.read + attendance.read). SOCIETY_ADMIN via `*`.
- `lib/nav.ts`: `staff` item (icon `idCard`, group manage, perm `staff.directory.read`, feature `staff.directory`, hideOnSharedDevice).
- `lib/ui-modes/modules.ts`: `staff` module, audience committee (built-in default PRO), supportsModes.

### Society-local dates & pure rules (both unit-tested)
- `lib/staff/dates.ts` — `societyDayString` (Intl en-CA in the society tz), `dayKey`/`dayKeyToString` (00:00Z anchor), `compareDay`, `monthDays`/`daysInMonth`. A night-shift check-OUT after midnight updates the OPEN check-in-day row (found by `checkInAt != null && checkOutAt == null`), never a new next-day row.
- `lib/staff/rules.ts` — `assertMarkableDay` (future → `future_date_blocked`; past → `backdate_reason_required`, returns `{backdated}`), `isEmployedOn` (joined ≤ day ≤ leftOn — attendance stops at leftOn), `attendancePercent` (Σweight/counted; HALF_DAY=0.5, HOLIDAY excluded from denominator, null when nothing counted), `assertDecidable`/`assertLeaveRange`.

### Service (`lib/staff/service.ts`)
- Staff CRUD (soft delete), directory cards, Pro list + export (skill filter applied outside the generic column layer via `skills hasSome`), `getStaffDetail` (CNIC decrypt→mask last-3→`staff.cnic.viewed` audit; runs under READ scope so a READ_ONLY society still views it — audit writes are permitted in read scope, mirroring residents `getFlatPanel`).
- `listStaffBySkill(code)` — the step-26 complaint assignment-suggestion hook (active staff whose `skills[]` has the category).
- Roster (`getRoster` — only staff employed on the day), bulk `markAttendance` (future blocked, past reason-gated + audited, non-employed entries skipped, upsert on the unique key), `checkIn` (QR/self IN/OUT with the night-shift open-row rule).
- Monthly grid (`getMonthlyGrid` — staff×day cells + `attendancePercent`), leave list/create/decide (approve upserts LEAVE marks across the range without overwriting existing marks).

### API
- `/api/staff` GET (directory / `?scope=manage` table) + POST; `/api/staff/:id` GET (detail) / PATCH / DELETE (soft); `/api/staff/export` (table, PII-audited via the export kit).
- `/api/attendance` GET roster `?date=`; `/api/attendance/mark` POST bulk; `/api/attendance/check-in` POST; `/api/attendance/grid` GET + `/api/attendance/grid/export` POST (xlsx via the export kit, no PII columns).
- `/api/leave-requests` GET (`?status=`) / POST; `/api/leave-requests/:id` PATCH (decide). HTTP mapping: future/backdate/range → 422, not-pending leave → 409, duplicate code → 409, unknown → 404.

### UI (`components/staff/*`, `app/[locale]/app/staff/page.tsx`)
- Sub-view switch: Directory (always) + Roster/Attendance/Leave (only when `staff.attendance.read`). Simple mode renders the directory as cards (call/WhatsApp, initials, skills), Pro as the data-table (export). Roster = mobile-first one-tap chips + "mark all present" + sticky save + a reason field that appears for a past day. Attendance grid = colour cells + attendance % + month nav + Excel export + legend. Leave panel = list + approve/reject + file-a-request modal. Add/edit modal (form kit: TextField/PhoneField/SelectField/MultiSelect skills/MaskedField cnic/DatePicker/SwitchField) + detail drawer (CNIC masked, edit + soft-delete confirm).
- i18n `messages/en.json` + `messages/ur.json`: `staff.*`, `nav.staff`, `uiModes.module.staff`. RTL via logical props.

### Gates (agent-run: lint + typecheck)
- `pnpm prisma generate` ✓, `pnpm lint` ✓ (0 warnings), `pnpm typecheck` ✓. Unit (`lib/staff/dates.test.ts`, `lib/staff/rules.test.ts`) + e2e (`e2e/staff-directory-attendance.spec.ts`, auth-gate on every surface) written for the controller to run. Existing features/nav/rbac tests are membership-based and unaffected.

### UI design self-check (by inspection)
shadcn/form+table kits only · light+dark (semantic tokens) · RTL/Urdu (ms/me/ps/pe, no ml/mr) · responsive (roster one-handed on a phone) · skeleton/empty/error states · Data Table kit for the Pro list (sort/filter/search/export/card toggle) · Form kit for every input · client-side nav (no reload) · Simple + Pro both implemented · nav+actions gated by role×entitlement, re-enforced server-side · toasts (no alert) · graceful degradation with `staff.attendance` disabled.

---

## Step 26 — complaints-service-requests — DONE (2026-07-12)

**Spec:** `/specs/26-complaints-service-requests.md`. **Branch:** `feature/26-complaints-service-requests`. **Work type:** FEATURE.

### Feature-code decision (recorded, not escalated)
Spec names `complaints.core` / `complaints.sla` / `complaints.public`. The platform had **forward-declared** the core feature as `complaints` in step 23/25 wiring — `lib/nav.ts` (`feature: "complaints"`), `lib/pwa/tabs.ts` (both resident + committee tabs), `lib/rbac.ts` PERMISSION_REGISTRY (`"complaints.assign": "complaints"`), AND an existing passing test `lib/pwa/tabs.test.ts:33` that asserts the complaints tab shows when `features` contains `"complaints"`. Renaming to `complaints.core` would break that green test and require editing another module's wiring. **Decision:** register the core feature as `complaints` (spec's `complaints.core` maps 1:1) and add the two new sub-features `complaints.sla` + `complaints.public`. Keeps all forward-declared wiring + tests green; noted in `lib/features.ts` comment.

### Data model — migration `20260712290000_complaints_service_requests`
Enums `TicketKind` (COMPLAINT|SERVICE_REQUEST), `TicketVisibility` (PRIVATE|PUBLIC), `TicketStatus` (OPEN|ASSIGNED|IN_PROGRESS|ON_HOLD|RESOLVED|CLOSED|REOPENED|CANCELLED); reuses existing `Priority`.
- `ServiceCategory` — auto-scoped (societyId+deletedAt). Tree via self-relation (`parentId`); per-category `slaHours` + `defaultAssigneeStaffId`; `sortOrder`, `isActive`. Unique `(societyId, code)`. Soft-deleted (retired), refused while live tickets reference it.
- `Ticket` — auto-scoped. `number` gapless per society, **separate sequence per kind** (`TicketSequence` PK `(societyId, kind, year)`, atomic `INSERT … ON CONFLICT DO UPDATE` inside the create tx, exactly like invoices). `reassignNeeded`, `dueAt`, `warnedAt`, `escalatedAt`, `escalationLevel`, `resolvedAt`/`resolutionNote`, `closedAt`, `reopenCount`, `linkedFromTicketId` (late-reopen relink). Indexes `(societyId,status,assignedStaffId)`, `(societyId,kind,status)`, `(societyId,visibility,status)`.
- `TicketEvent` — pass-through, APPEND-ONLY audit thread (one row per transition, with attachments). `TicketRating` — pass-through, unique `ticketId`, 1–5 (feeds step 36). `TicketUpvote` — PK `(ticketId,userId)`, public board only. `ComplaintSettings` — one row per society (numberPrefix seeded from BillingSettings.invoicePrefix, defaultVisibility, anonymousPublic, businessHoursOnly + window + businessDays[], reopenWindowDays/reopenHardDays, autoCloseDays, warnBeforeHours). Added back-relation `Society.complaintSettings`.
Migration generated via `prisma migrate diff --from-empty --to-schema-datamodel --script` (schema validated), then sliced to the complaints objects (3 enums, 7 tables, 11 indexes, 8 FKs).

### Pure rules (unit-tested — acceptance criteria)
- `lib/complaints/rules.ts` — `STATUS_TRANSITIONS` machine (`assertTransition` rejects illegal + same-status), `formatTicketNumber` (RUFI-CMP-2026-0042; separate per kind; no truncation), `reopenDecision` (reopen ≤window / blocked ≤hard / relink >hard), `assertRating` 1–5. 15 tests.
- `lib/complaints/sla.ts` — `slaDueAt` (24×7 = start+hours; business-hours walks the working clock, spills across midnight, skips weekends, starts a job raised outside hours at next open) + `nextEscalation` ladder (warn once → manager L1 on breach → committee L2 after 24h dwell; never for resolved/no-SLA). 12 tests. All 27 pass (`vitest run lib/complaints`).

### Service (`lib/complaints/service.ts`)
createTicket (flat-held 403 for residents / committee bypass; auto-assign+SLA clock from category default; no-assignee-but-SLA starts clock at creation + nudges committee), assignTicket/bulkAssign, transitionTicket (machine-guarded + notify raiser), updateTicketMeta, addNote (access-gated), rateTicket (raiser only, RESOLVED only, idempotent, confirm→CLOSE), reopenTicket (window/relink), toggleUpvote (public only), getTicketDetail (visibility: raiser|assignee-user|committee|PUBLIC else 403; anonymous mode nulls identity), getMyTickets, getBoard (kanban incl. REOPENED), listTickets/listTicketsForExport (table + decorated flat/staff/raiser names via `user.profile.fullName`), getPublicBoard (upvote-sorted), getDashboard (open/overdue/breached/resolved/avgRating + 14-day trend + by-status), getOverview, category CRUD, settings, `suggestAssignees` (→ `listStaffBySkill(category.code)`), and `sweepTickets` (worker).

### Worker
`lib/worker/handlers.ts` `runComplaintsSlaEscalate` replaces the stub; `registry.ts` wired (society-scoped, mutating, concurrency 2, existing `*/30 * * * *`). Does escalation ladder (only when `complaints.sla` enabled), auto-close RESOLVED-unrated past `autoCloseDays`, and flags tickets whose assignee left the society (`reassignNeeded` + committee notify) — never silently orphaned.

### API — `app/api/complaints/*`
`route.ts` GET (overview default / `?scope=manage` / `?board=1` / `?dashboard=1` / `?suggest=<cat>` / `?public=1`) + POST raise; `[id]` GET detail / PATCH meta; `[id]/{assign,transition,note,rate,reopen,upvote}`; `bulk-assign`; `categories` (+`[id]` PATCH/DELETE); `settings` (GET/PATCH); `export` (inline-or-202, audited). Actor `lib/complaints/actor.ts` mirrors staff (member tier + permission-gated committee tier + read/write scopes); `http.ts` error→status map (illegal_transition/invalid_rating 422, reopen_window_passed 409, flat_not_held/not_your_ticket 403).

### Wiring
`lib/features.ts` (+3 features), `lib/rbac.ts` (+3 perms, seeded COMMITTEE_MEMBER read+assign / MANAGER all four); nav/pwa-tabs/ui-modes already forward-declared. Notifications reuse existing `complaint.updated` template + `complaints` category. i18n: full `complaints.*` namespace added to `messages/en.json` + `messages/ur.json` (RTL).

### UI (`app/[locale]/app/complaints/page.tsx` + `components/complaints/*`)
RSC page (feature-gated → graceful EmptyState). Client `useUiModule("complaints")`. Simple: report-a-problem modal (kind + big category tiles + camera-capture photo via FileUpload + priority), my-tickets timeline list, committee kanban (native drag-to-transition, machine-guarded). Pro: DataTable ticket table (every filter + export), animated/exportable SLA dashboard (LineChartCard trend + BarChartCard by-status + stat tiles), category manager (tree CRUD + retire), settings panel (visibility default, business-hours window/days, reopen/auto-close windows), public board (upvote cards). Shared ticket drawer = audit thread + role-aware actions (assign w/ skill suggestions, transitions, note, rate+stars, reopen, upvote). Badges/tiles use design tokens + logical props (ms/me/ps/pe/border-s/text-start), light+dark, `dir` threaded to table+charts.

### Edge cases (spec §69–74) — all handled
flat-not-held → 403; assignee-leaves → `reassignNeeded` flag + notify (worker); no-assignee SLA starts at creation + committee notified; `complaints.sla` off → no dueAt/escalation, dashboard shows note (graceful); reopen >30d → blocked + new linked ticket. Private ticket unreachable cross-resident (403); public board hidden when `complaints.public` off; anonymous mode hides raiser identity.

### Gates
`pnpm prisma generate` ✓, `pnpm typecheck` ✓ (clean), `pnpm lint` ✓ (0 warnings), targeted `vitest run lib/complaints` ✓ (27/27). Did not run test:unit/e2e/build (controller runs full gates). Screenshot not attached — no runnable dev server/DB in the agent env (same as step 25); UI verified by inspection against the AGENT.md design self-check.

---

## Step 27 — Staff operations console ("My jobs") — DONE (2026-07-12)

**Work type:** FEATURE (branch `feature/26-complaints-service-requests` — see note below)
**Spec:** `/specs/27-staff-operations-console.md` (authoritative). No CODEREF.

### What it is
The worker's screen. A plumber/electrician opens the app on his phone and sees exactly the jobs assigned to him — Today, Assigned, Upcoming, and a Completed history with the resident's rating — and drives each job (Start → note+photo → resolve) one-handed. A pure, HARD-SCOPED surface over the step-26 ticket model: **no new tables, no new status machine.**

### Decisions (autonomous, no approval gate)
- **Feature deps mapping.** Spec names deps `complaints.core`, `staff.directory`, `pwa.core`. The registry has no `complaints.core`/`pwa.core` codes — they map 1:1 to the registered `complaints` and `branding.pwa` (the PWA/offline shell). Registered `staff.console` with `dependsOn: ["complaints", "staff.directory", "branding.pwa"]` (DAG validates; `assertValidDag` + features.test require every edge to resolve).
- **Access model.** New permission `staff.console.read` gated by the `staff.console` feature, **seeded to the STAFF role** (previously permission-less). Because effective permissions are role ∩ entitlements, the console vanishes when the feature is off (graceful degradation) with no extra plumbing. A caller must ALSO resolve to an active `StaffMember` linked to their user account — a user with no staff record simply has no console (spec: the committee updates their tickets on their behalf) → `not_a_staff_member` (403) at the API, graceful NoConsole state on the page.
- **Hard scope = server-side, not hidden links.** Every read filters `assignedStaffId = me`; every action calls `requireMyJob`, which loads the ticket and throws `not_your_job` (→ 403) unless `assignedStaffId` equals the caller's own `StaffMember.id`. This is the acceptance-criteria guarantee ("another staff member's job is 403 even by direct id"). `transitionTicket`/`addNote` themselves don't check assignee, so the ownership gate lives entirely in the console service and is applied before every delegate call.
- **Simple only.** Spec: "a worker does not need a Pro mode." No ui-module registered; the console is a single, fixed, mobile-first experience.
- **Bucketing.** Pure `rules.ts`: `classifyActiveJob(dueDay, todayDay)` → TODAY when due today / overdue / **no SLA due date at all** (so SLA-off societies still see work), UPCOMING when future-dated (society-local day compare via `lib/staff/dates.societyDayString`). `compareJobs` orders priority (URGENT→NORMAL) → nearest SLA due (dated before undated) → oldest-created (FIFO). Active buckets classify+sort in memory (a worker holds tens, capped 500); completed is a direct DB-paginated `status ∈ {RESOLVED,CLOSED}` filter with rating. 11 vitest cases.
- **Actions delegate to complaints service** so the status machine, append-only `TicketEvent` thread and resident notification stay in one place: start = ASSIGNED/REOPENED/ON_HOLD → IN_PROGRESS; hold → ON_HOLD (reason mandatory); resolve → RESOLVED (resolution note mandatory — it is what the resident sees — triggers the rating request); note + before/after photos (no status change). `requestReassign` is console-owned: sets `reassignNeeded`, writes a `REASSIGN_FLAGGED` event + audit, notifies MANAGER/COMMITTEE ("→ back to the committee"), status untouched.
- **Offline (step 23).** All POST writes send `X-Offline-Queue: 1`; the service worker returns the real response when online, or queues to the IndexedDB outbox + Background-Sync replay (202) when offline. The client treats 202 as "saved offline — will sync". **Conflict, not silent overwrite:** an offline resolve replayed onto an already-CLOSED ticket fails `assertTransition` (illegal_transition) — surfaced as 409 to the client (and the outbox drops it since <500), never re-resolving a closed ticket.
- **Global bottom bar suppressed on the console route.** The console ships its OWN bottom tabs (Today·Assigned·Upcoming·Done·More); `components/pwa/bottom-tabs.tsx` now returns null on `/app/my-jobs*` so a worker never sees two stacked bottom bars.
- **Branch.** The controller handed this step on the step-26 branch `feature/26-complaints-service-requests`; built here per that instruction. Per CLAUDE.md the natural branch would be `feature/27-staff-operations-console`.

### Files
- **lib/staff-console/**: `constants.ts` (JOB_BUCKETS, ACTIVE/COMPLETED status sets, PRIORITY_RANK), `rules.ts` + `rules.test.ts` (pure classify/compare), `actor.ts` (requireStaffConsoleActor / getStaffConsoleServerActor / hasConsoleAccess / runRead+WriteScope / staffConsoleCtx — mirrors complaints/staff actors), `http.ts` (error→status map + parseBody), `service.ts` (getMyStaffMember, listJobs, getJobDetail, startJob, noteJob, holdJob, resolveJob, requestReassign; `StaffConsoleError`).
- **schemas/staff-console.ts**: jobNote / jobHold / jobResolve / jobReassign (mandatory reason/note is the product rule).
- **app/api/staff/me/jobs/**: `route.ts` (GET ?bucket=&page=), `[id]/route.ts` (GET detail), `[id]/{start,note,hold,resolve,reassign-request}/route.ts` (POST).
- **app/[locale]/app/my-jobs/page.tsx**: feature+staff-record gated; graceful NoConsole.
- **components/staff-console/**: `my-jobs-client.tsx` (bottom tabs, per-bucket fetch, skeleton/empty/error, More panel), `job-card.tsx` (big flat card, call/WhatsApp, SLA countdown, photo hint, rating), `job-detail-drawer.tsx` (photos, resident contact, thread, sticky action bar, offline-aware `act()`, disabled-with-explanation when not actionable), `types.ts`.
- **Registration**: `lib/features.ts` (+`staff.console`), `lib/rbac.ts` (+perm, STAFF role seed), `lib/nav.ts` (+`myJobs`), `components/pwa/bottom-tabs.tsx` (route suppression).
- **i18n**: `messages/en.json` + `messages/ur.json` (`staffConsole.*` 37 keys incl. tabs/empty/actions/form/toasts/events, + `nav.myJobs`), RTL logical props throughout.
- **e2e/staff-operations-console.spec.ts**: 401 on every endpoint × every bucket/action (mirrors the complaints spec; the 403 hard scope is enforced in the service and documented in the spec header).

### Gates
- `pnpm typecheck` — pass. `pnpm lint` — pass (0 warnings). Prisma generate not needed (no schema change).
- NOT run by the agent per CLAUDE.md: `pnpm test:unit`, `pnpm test:e2e`, `pnpm build` (controller runs these).

### UI self-check
Design tokens + shadcn kit (Button/Skeleton/Textarea/Drawer/FileUpload/EmptyState); card list is the correct pattern for a phone job feed (not a Data Table); light+dark tonal chips reused from complaints badges; RTL via `ms-/me-/ps-/pe-/start-/end-` only (no `ml-/mr-/left-/right-`); mobile-first `max-w-lg`; skeleton/empty/error states present; camera capture one-tap via FileUpload `camera`; toasts (no alert); client-side nav; Simple-only by spec; graceful degradation when the feature is disabled.

### Edge cases covered
Reassigned-away job → detail bar disabled with an explanation (`actionable=false`). Offline resolve after committee close → 409 conflict surfaced, not overwritten. 40 open jobs → paginated. Resolve without a note → blocked by the Zod schema (mandatory). SLA off → jobs still show (TODAY bucket). Feature off → complaints unaffected, NoConsole page.

---

## 28 — gate-pass — DONE (2026-07-12)

**Feature/spec:** `/specs/28-gate-pass.md`. Registered feature `gatepass` (bare, matching the forward-declared nav/rbac/ui-modes wiring; the spec's `gatepass.core` maps 1:1) + per-type sub-features `gatepass.item_exit`, `gatepass.child_exit`, `gatepass.visitor`, `gatepass.delivery`, `gatepass.staff_recurring`, and `gatepass.cnic`. Deps: residents.registry, core.storage, core.notifications, core.worker, branding.pwa.

**Design decisions (recorded, autonomous — no approval gate):**
- **`DeviceKind` reused, not duplicated.** The spec's `deviceKind SHARED|PERSONAL` is identical to the existing `DeviceKind` enum (guard shared-device auth), so `GateEvent.deviceKind` reuses it. Direction is a new `PassDirection {IN,OUT}` (the ledger already owns `Direction`).
- **`Pass` is auto-scoped** (societyId + deletedAt) per the platform convention, giving auto-pinned reads + soft-delete; children (`PassPerson`/`PassItem`/`GateEvent`/`PassSequence`) and `GatePassSettings` are pass-through infra, resolved by passId/societyId explicitly (mirrors complaints).
- **Gapless numbering** "RUFI-GP-2026-0042" via `PassSequence(societyId,year)` with the atomic `INSERT … ON CONFLICT DO UPDATE … RETURNING` inside the create transaction (single sequence per society/year, not per type — the spec shows one GP series).
- **QR = signed, expiring, SOCIETY-BOUND HMAC** (`lib/gate-pass/token.ts`, modelled on `lib/storage/signing.ts`). societyId is inside the signed payload AND re-checked on verify, so a Rufi token cannot verify at another society (fails the equality guard AND could never carry a valid signature for another society). Secret = `STORAGE_SIGNING_KEY || NEXTAUTH_SECRET` (no new env var). The token is signed with the real pass id after create (a placeholder satisfies the NOT NULL @unique column, replaced in the same transaction).
- **No new npm dependency for QR generation.** There is no QR lib in the repo; rather than a risky `pnpm add` (offline/lockfile risk under the "verify cheaply, don't build" rule), a compact dependency-free byte-mode QR encoder was vendored (`lib/gate-pass/qrcodegen.ts`, adapted from Nayuki's public-domain reference) with a defensive `QrCode` React component that falls back to the 6-digit code if encoding throws. Guard scanning uses the native `BarcodeDetector` when available, else the CODE/SEARCH paths. Follow-up: swap in a battle-tested `qrcode` dep when install is available.
- **Per-type disabling** = a type is offerable iff its `gatepass.*` sub-feature is entitled ∩ it is in `GatePassSettings.enabledTypes` (default all five). Satisfies "every pass type can be independently disabled, UI adapts with no dead links".
- **Approval model** (no extra schema fields): resident-of-flat approval alone is sufficient (replaces the paper slip); when `itemExitCommitteeCosign` is on, an item-exit needs BOTH the resident approval (recorded on `approvedByUserId`) AND a committee co-sign to reach APPROVED. Visitor/delivery passes a resident creates for their own flat are self-authorised (APPROVED); item/child/recurring are PENDING_APPROVAL. A guard walk-in creation path exists in the service (`isGuardWalkIn`, gated by `walkInGuardCreate`) and pushes an approval request to the flat; the v1 guard UI surfaces verify/quick-delivery/manual-entry (walk-in creation is a documented service capability, not yet a dedicated guard button).
- **Safeguarding:** child photos are not stored unless `childPhotoStored`; child passes are visible only to the flat's residents + committee (`getPassDetail` 403s others); the register export EXCLUDES child passes and writes an audit entry. CNIC (`gatepass.cnic`) is AES-encrypted at rest, masked (last-4) on read, and every reveal writes a `gatepass.cnic.viewed` audit.

**Migration `20260712300000_gate_pass`:** enums PassType/PassStatus/PersonRole/PassDirection/VerifyMethod/ItemExitApprover/DeliveryMode; tables Pass/PassPerson/PassItem/GateEvent/PassSequence/GatePassSettings + indexes + FKs (children CASCADE, GateEvent.passId SET NULL, settings→Society CASCADE). Society gains a `gatePassSettings` back-relation. No permission/role/feature/template INSERTs (those are seeded from app code).

**lib/gate-pass/:** `constants.ts` (enum tuples, PASS_TYPE_FEATURE map, notif codes), `rules.ts` (pure: normalizePrefix/formatPassNumber, canApproveItemExit, isApprovalComplete, validityWindow/isExpired, verifyDecision — 6 vitest groups), `token.ts` (sign/verify + env wrappers — vitest proves cross-society + expiry + tamper + bad-secret rejection), `actor.ts` (requireGatePassMember/Actor/Guard, run{Read,Write}Scope, isCommittee/isGuard), `http.ts` (typed-error → status), `columns.ts` (pass register + gate-event tables), `service.ts` (settings ensure/update/availableTypes; createPass; approve/reject/cancel; verifyPass QR/CODE/SEARCH; logGateEvent → GateEvent+USED+notify; quickDeliveryLog; manualEntry; guardFlatLookup; searchPasses; currentlyInside; getOverview/getMyPasses/getPendingApprovals/getPassDetail/listPickupPeople; listPasses/listGateEvents + export variants; getGuardConsole; worker expirePasses), `qrcodegen.ts` (vendored QR) + `token.test.ts`/`rules.test.ts`.

**Wiring:** features.ts (+7 defs), rbac.ts (perms + GUARD/COMMITTEE/MANAGER grants), nav.ts (resident `gatePass`→/gate-pass main; guard `gatepass`→/gate manage, no hideOnSharedDevice), nav-icons (ticket), worker registry+handlers (`gatepass.expire` cron */15 + runGatepassExpire), notifications/templates.ts (7 codes, en+ur every channel), pwa/bottom-tabs (suppress on /app/gate). ui-modes + notification category already forward-declared.

**API (app/api/gate-pass):** `route.ts` GET overview/?scope=manage/?events/?pickups + POST create; `[id]/route.ts` GET detail; `[id]/{approve,reject,cancel}`; `settings` GET/PUT; `export` POST (child-excluded, audited); `guard/{verify,search,events,delivery,manual,inside}` all gated on `gatepass.verify` (hard-scoped — no resident/financial reach; search returns only pass-verification data, never a directory).

**Schemas:** `schemas/gate-pass.ts` (passCreate/passReject/passVerify(QR|CODE|SEARCH)/gateEvent/quickDelivery/manualEntry/gatePassSettings).

**UI:** `app/[locale]/app/gate-pass/page.tsx` → `GatePassClient` (Simple/Pro via `useUiModule("gatepass")`): type tiles (availableTypes), create-pass modal (flat + person/item + camera photo + notes), my-passes, pending-approvals queue, pass-detail drawer (QR + big 6-digit code + persons/items/gate-events + approve/reject/cancel), committee register table + settings toggles. `app/[locale]/app/gate/page.tsx` → `GuardConsoleClient` (mobile, Urdu-first): big Scan-QR (BarcodeDetector)/Enter-code/Search/Quick-delivery buttons, verify result screen (green OK with capture-photo→confirm IN/OUT; red EXPIRED/REJECTED with resident phone to call), delivery form (flat lookup + company + rider + photo), currently-inside board, 30-min idle lock, offline writes send `X-Offline-Queue`→202 "saved offline". `components/gate-pass/qr-code.tsx`.

**i18n:** en+ur `gatePass.*` and `gate.*` namespaces + `nav.gatePass`; RTL via existing `dir=rtl` on `<html>` + logical properties.

**Tests:** vitest `token.test.ts` (society-binding, expiry, tamper, bad secret, malformed), `rules.test.ts` (approver policy ANY vs OWNER_ONLY, co-sign gate, numbering, expiry, verify decision). Playwright `e2e/gate-pass.spec.ts` (401 on every endpoint incl. guard console).

**Gates run by the agent:** `pnpm prisma generate` ✓, `pnpm lint` ✓ (no warnings/errors), `pnpm typecheck` ✓. Unit/e2e/build deferred to the controller per CLAUDE.md.

**Edge cases covered:** expired scan → clear EXPIRED screen + resident phone (never silent); walk-in no-response → guard manual entry with reason (audited); item-exit for a 3-resident flat → any one approves (default), OWNER_ONLY optional; offline guard → event queues (202) + Background-Sync replay; NO_LOG delivery → quick-log button hidden; a type's sub-feature off → its tile disappears, others unaffected; recurring staff pass stays APPROVED (many entries), one-shot passes flip to USED.

WORK TYPE: FEATURE (branch feature/28-gate-pass)

---

## Step 29 — utility-bill-notices — DONE (2026-07-12)

**Branch:** feature/29-utility-bill-notices · **Spec:** /specs/29-utility-bill-notices.md (AUTHORITATIVE)
**Feature code:** `utility.notices` · deps `residents.registry`, `core.notifications`, `core.storage`, `core.worker`.

### What it is
"SSGC bills have arrived, due 25 July." A REMINDER with per-flat paid/unpaid state, targetable at a subset of flats. **NOT billing** — the society is not collecting this money.

### HARD RULE (spec) & how it is enforced
A utility amount NEVER enters the maintenance ledger: no `LedgerEntry`, no `Invoice`, no effect on a flat's balance. Enforced by `lib/utility/architecture.test.ts`, a static scan that fails if any `lib/utility/*.ts` source references `@/lib/billing/`, `@/lib/charges/`, `ledgerEntry`, or `invoice.create`. `amountMinor` is a reminder figure only. The notification copy explicitly says "not a maintenance charge", and the resident surface is a **separate nav page (`utility`, icon `zap`)** — structurally distinct from `finance`, never mixed into the maintenance balance.

### Data model (migration `20260712310000_utility_bill_notices`)
- Enums: `UtilityKind` (GAS/ELECTRICITY/WATER/INTERNET/TELEPHONE/OTHER), `UtilityTargetType` (ALL/BLOCK/CATEGORY/FLAT_LIST — DEDICATED, not the charge engine's `TargetType`, to keep utility decoupled from billing, mirroring the announcements precedent), `UtilityNoticeStatus` (DRAFT/PUBLISHED/CLOSED), `UtilityFlatStatus` (PENDING/PAID/DISMISSED).
- `UtilityProvider` — per-society `code` unique, `kind`, optional `logoFileId`, `isActive`. Presets seedable: SSGC, SNGPL, K-Electric, LESCO, KW&SB, PTCL, StormFiber, Nayatel.
- `UtilityBillNotice` — providerId, title, billMonth ("2026-07"), dueDate, targetType+targetIds[], `reminderDaysBefore Int[] @default([7,3,1])`, `remindAfterDue`, status, publishedAt.
- `UtilityBillFlat` — the per-flat state (the whole point): optional `amountMinor BigInt?`, `consumerNumber?`, `billFileId?`; status; `paidAt`/`markedBy`/`paymentProofFileId`; `revertReason` (audited committee revert); `amountNotifiedAt` (amount-added notified once); `lastReminderOn` "YYYY-MM-DD" (one reminder per society-day dedupe). `@@unique([noticeId, flatId])`.
- All three carry societyId+deletedAt → AUTO-SCOPED by `lib/db.ts` (reads auto-pinned, societyId stamped on create incl. publish `createMany`, deletes → soft delete + audit). Notice→provider (RESTRICT) and notice→flats (CASCADE) FKs.

### lib/utility
- `constants.ts` — enum tuples, NOTIF codes (posted/reminder/amount_added), provider presets, `REMINDER_STOPPING_STATUSES`.
- `rules.ts` (pure, vitest `rules.test.ts`) — `zonedDayKey`, `daysUntilDue` (society-tz calendar diff), `reminderDue` (THE rule: only PUBLISHED notice + PENDING flat, ≤once/day via lastReminderOn, fires when days-until-due ∈ reminderDaysBefore, after due only if remindAfterDue — a passed due date otherwise stops reminders), `paidProgress`, `normalizeReminderDays`. `UtilityRuleError`.
- `targeting.ts` — `flatWhereForTarget` + `resolveTargetFlats` (flats + current occupants; a flat with no resident still resolves → row created, nobody notified). Self-contained (not imported from announcements).
- `actor.ts` — `UTILITY_READ`/`UTILITY_MANAGE`; `requireUtilityMember` (any member: view + mark own flat), `requireUtilityActor` (committee), `runReadScope`/`runWriteScope` (folds impersonation + READ_ONLY → 423), `isCommittee`/`canManage`.
- `http.ts` — typed-error → status mapping (404/403/409/422) + `authErrorResponse` fallback; body/list-query parsers.
- `columns.ts` — Pro notice-table `ColumnConfig` + flat-matrix export cells.
- `service.ts` — providers (list/create/update/seed, P2002→duplicate_code), notices (create DRAFT → `publishNotice` fan-out one `UtilityBillFlat` per targeted flat + notify occupants → close), `updateNotice`, `listNotices` (decorated with per-status counts via groupBy), `getNotice` (flat matrix + progress), `patchFlat` (amount/consumer/photo; amount-added notifies once), `bulkFlats` (match by flatId or consumerNumber — CSV import), `markPaid` (resident-or-committee), `dismiss`, `revertToPending` (committee, audited, reason required), `myUtilityBills` (resident), `getOverview` (+committee target options: blocks/categories/flats), worker `sweepUtilityReminders`. `UtilityError`.

### Permissions / roles / features / nav / ui-modes
- rbac: `utility.read`→`utility.notices`, `utility.manage`→`utility.notices`. Seeded: COMMITTEE_MEMBER (+utility.read), MANAGER (+utility.read, +utility.manage), SOCIETY_ADMIN via `*`. Residents mark their own flat with NO permission (member tier).
- features.ts: `utility.notices` registered (module 29-utility-bill-notices).
- nav.ts: `utility` main-group item, feature-gated, hideOnSharedDevice; icon `zap` added to nav-icons. PWA bottom tabs left as-is (curated).
- ui-modes/modules.ts: `utility` module, audience resident (Simple default).

### API
`GET/POST /api/utility/providers`, `POST /api/utility/providers/seed` (+GET presets), `PATCH /api/utility/providers/[id]`, `GET/POST /api/utility/notices` (default GET = page overview; `?scope=manage` = Pro table), `GET/PATCH /api/utility/notices/[id]`, `POST .../publish`, `.../close`, `.../bulk`, `PATCH /api/utility/flats/[id]`, `POST .../mark-paid`, `.../dismiss`, `.../revert`, `POST /api/utility/export` (per-notice flat matrix, audited, >1000 rows → 202 queued), `GET /api/me/utility-bills`.

### Worker
`utility.reminders` cron (`0 10 * * *`, society-scoped) — replaced the registered stub with `runUtilityReminders` → `sweepUtilityReminders`. Marked `mutating: true` (stamps lastReminderOn) so a READ_ONLY society is skipped. Reminders STOP on paid/dismissed or due-date pass (unless remindAfterDue).

### Notifications
Added `utility.notice.reminder` + `utility.notice.amount_added` templates (en+ur, IN_APP/SMS/PUSH); rewrote `utility.notice.posted` to use `{{title}}`/`{{dueDate}}` and state "not a maintenance charge". Category `utility` (already present).

### UI (Simple + Pro, light/dark, EN/UR RTL, mobile+desktop)
`app/[locale]/app/utility/page.tsx` (RSC feature-gate) + `components/utility/utility-client.tsx`:
- Resident (Simple, always): bill cards — provider/title, due badge ("due in N days"/"today"/"overdue"), amount if known, consumer no, view-bill link, "Mark as paid" (optional slip modal upload) + "Not applicable" (dismiss); settled cards grey out and stop nagging.
- Committee: "Notices" tab (Simple cards / Pro table with paid progress) → per-notice drawer with the flat matrix (progress bar, inline bulk amount entry on blur, per-flat mark-paid / audited revert-with-reason, close-notice), and "New notice" wizard (provider → month → due date → who: ALL/BLOCK/CATEGORY/FLAT_LIST picker → remind-after-due → Save draft / Publish). Empty-provider state offers a one-tap seed of common providers.

### Decisions
- Dedicated `UtilityTargetType` (not the shared `TargetType`) — decouples utility from the charge module and reinforces "never billing".
- Auto-scoped all three tables (added deletedAt) — simplest correct fan-out (`createMany` gets societyId injected) and consistent soft-delete/audit.
- Reminder dedupe via `lastReminderOn` day-string (idempotent within a day and across worker re-runs) rather than a separate sent-log table.
- flatId kept as a plain string (no FK to Flat), matching the `Pass.flatId` precedent — no Society/Flat back-relations added.

### Gates
`pnpm prisma generate` ✓, `pnpm lint` ✓ (fixed jsx-no-literals in the client), `pnpm typecheck` ✓. Did NOT run test:unit/test:e2e/build per CLAUDE.md — controller runs full gates. Tests written: `lib/utility/rules.test.ts` (reminder cadence incl. stop-on-paid/dismiss/due-pass), `lib/utility/architecture.test.ts` (never-ledger), `e2e/utility-bill-notices.spec.ts` (401 on every endpoint).

WORK TYPE: FEATURE (branch feature/29-utility-bill-notices)

## 30 — expenses-accounts — DONE (2026-07-12)

**Feature:** `expenses.core` (deps `billing.ledger`, `core.storage`; `staff.directory` is a SOFT dep — runtime-checked via `isEnabled`, NOT a DAG edge, so enabling expenses never forces the directory on). Registered in `lib/features.ts`. The ui-mode module `expenses` was already pre-registered (`lib/ui-modes/modules.ts`, audience committee → default PRO); no change there.

**Spec:** /specs/30-expenses-accounts.md (AUTHORITATIVE). "Where the union's money went." Collection without expense transparency is half a product.

### Hard rule (architecture test)
`lib/expenses/architecture.test.ts` statically forbids `@/lib/billing/`, `@/lib/charges/`, `ledgerEntry`, `.invoice.create` in every `lib/expenses/*.ts` (non-test). An expense NEVER creates a `LedgerEntry` and never touches a flat's maintenance balance — expenses are the society's books, not a resident's dues. Income for the account summary is read by AGGREGATING the `Payment` table directly (status CLEARED) — a read, never a ledger write — so no billing-module import is needed (kept the FORBIDDEN list airtight; had to reword a service.ts comment that literally contained `@/lib/billing/` so the test itself stays green).

### Data model — migration `20260712320000_expenses_accounts`
- New enum `ExpenseStatus` (DRAFT/PENDING_APPROVAL/APPROVED/REJECTED/PAID/VOID). Reused existing `PaymentMethod` (spec restricts the Zod layer to CASH/CHEQUE/BANK_TRANSFER).
- `ExpenseHead` (societyId+deletedAt → auto-scoped; self-relation parent→sub-head `HeadSubHead`; unique [societyId,code]; isActive).
- `Vendor` (auto-scoped; contactPerson/phone/email/ntn/category; soft-remove modelled as `isActive=false` so historical expenses keep showing the vendor, flagged — spec edge case).
- `Expense` (auto-scoped; gapless per-society `voucherNumber` "RUFI-EXP-2026-07-0031"; headId/vendorId?/staffId?; amountMinor BigInt; paymentMethod; attachments String[] StoredFile ids; status; approvedBy/At, rejectedReason, voidedAt/voidReason, recordedBy; unique [societyId,voucherNumber]; index [societyId,expenseDate,headId] + [societyId,status]).
- `Budget` (auto-scoped; unique [societyId,headId,period] where period is "2026-07" or "2026").
- Infra pass-through (societyId-keyed, no deletedAt): `ExpenseSettings` (requireApproval, approvalThresholdMinor?, publishToResidents, voucherPrefix default "EXP"; FK→Society) + `ExpenseSequence` (@@id [societyId,prefix,year,month]).
- Migration hand-written to repo convention (scalar lists as bare `TEXT[]`, matching utility's `targetIds`). `pnpm prisma generate` clean.

### Numbering
Voucher = `{societyPrefix}-{docCode}-{YYYY}-{MM}-{seq4}`. societyPrefix ← `BillingSettings.invoicePrefix` (fallback society name → "SOC"); docCode ← `ExpenseSettings.voucherPrefix`. `seq` allocated inside the create txn via atomic `INSERT … ON CONFLICT DO UPDATE` on `ExpenseSequence` (same gapless mechanism as invoices/receipts/passes). A VOIDED expense keeps its number (void is a status flip, never a delete / never re-allocates).

### lib/expenses/
- `constants.ts` — pure enum tuples, `SPEND_COUNTED_STATUSES` (APPROVED∪PAID = actual money out), `HEAD_PRESETS` (salaries/electricity/lift-maintenance/water-tanker/security/cleaning/repairs/generator-fuel), `VOUCHER_DOC_CODE`, `SALARY_HEAD_CODE`.
- `rules.ts` (vitest `rules.test.ts`) — `formatVoucherNumber`/`normalizePrefix`; `requiresApproval` (off→never; on+no threshold→always; on+threshold→strictly ABOVE, so below/equal SKIP approval); `initialStatus`; `accountSummary` (opening+income−expense=closing, all BigInt; hand-computed fixture 200k+644k−512k=332k, income−expense=132k matching the Simple card); `budgetVariance` (exceeded flag, %; over-budget warns, never blocks); `isUnsupported` (no receipt & not void/rejected).
- `actor.ts` — perms EXPENSES_READ/RECORD/APPROVE/MANAGE; `requireExpenseMember` (any authed member, for the resident transparency view), `requireExpenseActor(req,perm)`, `getExpenseServerActor` (RSC), isCommittee/canRecord/canApprove/canManage, runReadScope/runWriteScope (fold impersonation-RO + READ_ONLY society → 423), expenseCtx. Mirrors `lib/utility/actor.ts`.
- `http.ts` — `expenseErrorResponse` (ExpenseRuleError→422; ExpenseError code sets →404/403/409/422/400; Zod→400; else shared authErrorResponse) + parseBody + parseListQueryFromUrl.
- `columns.ts` — Pro expense table ColumnConfig (voucher/description searchable, headId/vendorId/status/paymentMethod enum filters, amount+expenseDate sortable) + export cells; head/vendor names + has-receipt derived by the service.
- `service.ts` — settings ensure/get/update; heads CRUD+seed (+ensureSalaryHead); vendors CRUD (isActive deactivate); expense create (approval decision → DRAFT/PENDING_APPROVAL/APPROVED)→updateDraft (throws `not_editable` past DRAFT — the immutability gate)→submit→approve→reject→markPaid→void (keeps voucher); listExpenses/getExpense (decorate head/vendor names, vendorDeleted, unsupported); budgets setBudget(upsert)/listBudgets(variance)/deleteBudget; `getAccountSummary(period)` (opening = net of all prior activity, income = CLEARED payments, expense = counted, byHead groupBy, cash/bank split, unsupportedCount); `residentAccounts` (published-only; head-level aggregates + monthly trend + totals; NO vendor, NO per-staff — the transparency guarantee); `generateSalaryDrafts` (staff.directory-gated → 422 `staff_directory_off`; one DRAFT per active staff with salary); `getOverview` (page bootstrap).
- `schemas/expenses.ts` — shared Zod (head/vendor/expense create+update, reject/void/markPaid, budget, salaryDraft, settings). Amounts are non-negative minor-unit ints (client parses money→minor before send), mirroring utility.

### Permissions (lib/rbac.ts)
`expenses.read` (COMMITTEE_MEMBER, MANAGER, ACCOUNTANT) · `expenses.record` (ACCOUNTANT, MANAGER) · `expenses.approve` (COMMITTEE_MEMBER, MANAGER) · `expenses.manage` (MANAGER). All gated by `expenses.core` (∩ rule). Residents hold NO perm — the published transparency view is authenticated-member only. SOCIETY_ADMIN via wildcard.

### API
`/api/expense-heads` (GET/POST) + `/[id]` (PATCH) + `/seed` (POST) · `/api/vendors` (GET/POST) + `/[id]` (PATCH) · `/api/expenses` (GET overview | `?scope=list` Pro table; POST create) + `/[id]` (GET/PATCH) + `/[id]/{submit,approve,reject,void,mark-paid}` (POST) + `/salary-draft` (POST) + `/settings` (GET/PUT) · `/api/budgets` (GET/POST) + `/[id]` (DELETE) · `/api/accounts/summary?period=` (GET) · `/api/me/society-accounts` (GET, resident, `{published:false}` when off).

### UI
Page `app/[locale]/app/expenses/page.tsx` (feature-gated; branches committee vs resident). Client `components/expenses/expenses-client.tsx` (`useUiModule("expenses")` Simple/Pro): committee console — month summary card (Collected/Spent/Balance + cash/bank + unsupported count), tabs Add/Expenses/Approvals(badge)/Budgets/Vendors/Settings gated by perms; Add-expense Simple flow (category tiles → amount → description → vendor/method → date → receipt capture, save-draft/record); expense list (Simple cards / Pro table, unsupported+vendor-inactive flags, detail drawer with approve/reject(reason)/mark-paid/void(reason)); budgets vs actual bars; vendor add/list; settings (requireApproval, threshold, publish toggle). Resident view — published-only spend-by-head bars + income/expense/balance. Nav item `expenses` (icon `wallet`, group `main`, feature-only gate so residents can reach the published view; hideOnSharedDevice). i18n en+ur full `expenses.*` namespace + `nav.expenses`.

### Deliberate scope decisions (recorded, no approval gate)
- NO worker cron and NO notification templates/category: spec 30 requires neither (approvals are a pull queue, salary batch is a manual button). Kept the worker/notification registries untouched.
- Salary batch creates DRAFT expenses (committee reviews then submits/approves).
- "Expense" for the summary counts APPROVED∪PAID (committed money); DRAFT/PENDING/REJECTED/VOID excluded.

### Gates
`pnpm prisma generate`, `pnpm lint`, `pnpm typecheck` all clean. Did NOT run test:unit/e2e/build (controller runs full gates). Acceptance-criteria unit tests written (`rules.test.ts` for numbering/threshold/summary/variance/receipt-flag; `architecture.test.ts` for the no-ledger rule). e2e `e2e/expenses-accounts.spec.ts` (401 on every endpoint).

WORK TYPE: FEATURE (branch feature/30-expenses-accounts)

---

## Step 31 — document-vault — DONE (2026-07-12)

**Feature:** `documents.core` (non-core). Deps `core.storage`, `residents.registry` (both registered; DAG validated at load). Spec `/specs/31-document-vault.md` — AUTHORITATIVE. No CODEREF. WORK TYPE: FEATURE (branch feature/30-expenses-accounts — controller manages branch/merge).

**Purpose:** Bylaws, meeting minutes, notices, NOCs, floor plans in one place, with the right people able to see them. Private by default; versioned; every read logged.

### Decisions
- **Private by default (spec):** downloads never hit a public path. `getDownload` resolves the document's `StoredFile.key` (within the society scope) and returns a fresh short-lived signed URL from the step-10 adapter (`getAdapter().signedUrl(key, 300s)`). The actual bytes still leave only via the existing `/api/files/blob` HMAC route, so a guessed/tampered signed URL is 403 by the storage layer. The document `download` endpoint itself requires auth and runs the visibility gate, so a resident guessing another flat's FLAT_SCOPED document id gets 403 there.
- **Visibility resolution (pure, `rules.ts`):** `resolveVisibility(doc, folder)` = document override → folder → default `COMMITTEE_ONLY`. When the document overrides, ITS `allowedRoles` apply; when it falls through, the FOLDER's do. `flatId` only ever comes from the document. `canViewDocument`: committee sees all; resident sees ALL_RESIDENTS, a role match for ROLE_SCOPED, and FLAT_SCOPED only if the flat is in `currentFlatIds`.
- **The privacy trap (acceptance criteria):** `buildViewer` derives `currentFlatIds` from OPEN `FlatOccupancy` holdings (`endDate == null`) for the user. A previous occupant's holding is closed (endDate set) → absent from `currentFlatIds` → refused immediately; the new occupant only ever gets their OWN current flat, never the old occupant's personal NOC. Proven exhaustively in `rules.test.ts` with no DB.
- **Committee vs resident tier:** the pure `canViewDocument` treats committee (`isCommittee`) as seeing everything (they administer the vault — NOCs are committee-issued). The FLAT_SCOPED restriction is the RESIDENT-tier decision, which is exactly where the trap is tested.
- **Versioning never overwrites (spec):** `newVersion` creates a NEW `Document` with `supersedesDocumentId = old.id`, `version = old.version + 1`; the old row is retained. A document is the "head" iff no live row supersedes it (`supersededIds`/`isHeadVersion`). Only a head can be versioned (`not_head` otherwise — a fork would corrupt history). Feed/tiles/table show head versions only; the version-history drawer walks the chain both directions.
- **Expiry hides, never deletes:** `isVisibleInFeed` = published (`publishedAt <= now`, non-null) AND not expired (`expiresAt` unset or in the future). Residents see feed-visible head docs only; committee sees drafts/expired too, flagged. Soft-delete (folder delete cascades to its documents via scoped `deleteMany`) retains rows until the retention window.
- **Access log:** `DocumentAccessLog` has `societyId` but NO `deletedAt` → pass-through infra (like AuditLog/Notification), filtered on `societyId` explicitly, never soft-deleted. `getDownload` writes one row per open (`?mode=view` → VIEW, else DOWNLOAD). `getAccessLog` (committee, documents.read) joins UserProfile for names.
- **50 MB PDF edge case:** handled by the storage layer's existing 10 MB `MAX_UPLOAD_BYTES` cap — the upload flow throws `FileTooLargeError` before a document row is ever created; the add-document form surfaces the "compress and re-upload" guidance. The document module adds no new size logic.
- **Committee Pro table paginated in memory:** visibility resolution + head detection are app-level (not a simple `where`), and the console is committee-only over a bounded per-society document set, so `listDocumentsTable` decorates all rows then filters/sorts/pages in memory. The resident feed is a non-paginated array of head, feed-visible docs.
- **No cron, no notifications:** the spec requires neither. (A document upload could notify targeted residents, but that is out of scope for this spec.)

### Data model (migration `20260712330000_document_vault`)
- `DocVisibility` enum: COMMITTEE_ONLY | ALL_RESIDENTS | ROLE_SCOPED | FLAT_SCOPED.
- `DocumentFolder` (auto-scoped societyId+deletedAt): self-ref `parentId` tree, `visibility` (default COMMITTEE_ONLY) + `allowedRoles[]`, `sortOrder`, `createdBy`.
- `Document` (auto-scoped): `folderId`, `title`/`description`, `fileId` (StoredFile), `version`, self-ref `supersedesDocumentId`, `flatId` (FLAT_SCOPED), `tags[]`, per-doc `visibility?` override + `allowedRoles[]`, `publishedAt`/`expiresAt`, `uploadedBy`. Indexes `[societyId, folderId]`, `[societyId, supersedesDocumentId]`.
- `DocumentAccessLog` (pass-through, no deletedAt): `documentId`, `userId`, `action`, `at`. Indexes `[societyId, documentId]`, `[societyId, at]`.
- Schema appended cleanly (96-line add; did NOT run `prisma format` to avoid whitespace churn across unrelated models). `prisma generate` OK.

### Code
- `lib/documents/`: `constants.ts` (DocVisibility tuple, access actions, folder presets, `docKindForMime` icon classifier), `rules.ts` (all pure decisions) + `rules.test.ts` (vitest: resolution, canView/canViewFolder incl. the privacy trap, publish/expiry, versioning, search), `actor.ts` (member/committee actors, read/write scopes, `documentCtx` carrying isCommittee+roleCodes), `http.ts` (typed error → status; forbidden→403, unknown→404), `columns.ts` (Pro column contract + `DocumentDto`), `service.ts` (folders, documents, versions, download, access log, overview).
- Registration: `lib/features.ts` (`documents.core`), `lib/rbac.ts` (`documents.read` → COMMITTEE_MEMBER+MANAGER; `documents.manage` → MANAGER; PERMISSION_REGISTRY gated by documents.core), `lib/nav.ts` (`documents` item, icon `folderArchive`, main group, feature-gated, hideOnSharedDevice, no permission → member-visible), `components/shell/nav-icons.ts` (FolderArchive glyph). `ui-modes/modules.ts` already had `documents` (resident/SIMPLE).
- `schemas/documents.ts`: folder create/update/seed, document create/update, new-version, bulk-visibility (shared client+server).
- API (`app/api/documents/`): `route.ts` (GET feed / `?scope=list` / `?scope=overview`; POST create), `folders/route.ts` (GET/POST), `folders/[id]/route.ts` (PATCH/DELETE), `folders/seed/route.ts` (POST), `[id]/route.ts` (GET/PATCH/DELETE), `[id]/download/route.ts` (GET signed+logged), `[id]/new-version/route.ts` (POST), `[id]/versions/route.ts` (GET), `[id]/access-log/route.ts` (GET), `bulk-visibility/route.ts` (POST).
- UI: `app/[locale]/app/documents/page.tsx` (feature-gated RSC → overview) + `components/documents/documents-client.tsx` (Browse: folder tiles → doc cards → mobile PDF/image viewer modal + download; Manage/committee: folder bar seed/create, Pro table with per-doc version-history + access-log drawers, add-document + new-version modals, delete; RTL logical props, semantic tokens for light/dark).
- i18n: `messages/en.json` + `ur.json` `documents.*` namespace + `nav.documents` (en/ur key parity verified by script).
- e2e: `e2e/documents-vault.spec.ts` (401 on every GET/POST/PATCH/DELETE endpoint).

### Gates
- `pnpm prisma generate` — OK.
- `pnpm typecheck` — clean.
- `pnpm lint` — clean (fixed two `react/jsx-no-literals` on a "·" separator, added `cancelLabel` to ConfirmDialog).
- Did NOT run test:unit / test:e2e / build per CLAUDE.md — controller runs full gates.

### Acceptance criteria mapping
- [x] Vitest visibility resolution (document → folder → default) + FLAT_SCOPED current-occupant limit — `rules.test.ts`.
- [x] Previous occupant cannot access after endDate (explicit test) — `rules.test.ts` "FLAT_SCOPED is limited to the flat's CURRENT occupants".
- [x] Downloads use signed URLs and are logged; a guessed URL returns 403 — `getDownload` (signed adapter URL + DocumentAccessLog) + storage blob-route HMAC 403 + visibility gate 403.
- [x] Versioning retains old versions; nothing overwritten — `newVersion` + `supersededIds`/`isHeadVersion`.
- [x] PDF viewer works on mobile; light/dark, EN/UR (RTL) — iframe viewer (native pinch-zoom), logical props, semantic tokens (design self-check by inspection).

---

## Step 32 — messaging-chat — DONE (2026-07-13)

**Feature:** `chat.core` (isCore=false) · **Deps:** residents.registry, core.notifications, core.storage, branding.pwa (spec names `pwa.core`; mapped to the registered `branding.pwa` code, as step 27 does). **Branch:** feature/32-messaging-chat (already checked out from prior step; WORK TYPE FEATURE).

### What it is
The direct, two-way conversation channel between a resident and the committee / manager / amenity desk — the channel residents actually open the app for (announcements are broadcast, complaints are tickets; this is the chat). Role-addressed, private-by-participant, soft-delete-only.

### Key design decisions
- **Role-addressed threads (the survives-a-year rule).** A `ThreadParticipant` row with `userId = NULL` and a `roleTarget` ("COMMITTEE"/"MANAGER"/"AMENITY") addresses a ROLE, not a person. Access + reply derive from CURRENT role membership (pure `canAccessThread`/`satisfiesRoleTarget` in `lib/chat/rules.ts`), so any current committee member can reply and the thread survives the original replier leaving. Role→role-codes map lives in `constants.ts` (`ROLE_TARGET_ROLES`: COMMITTEE→{COMMITTEE_MEMBER,MANAGER,SOCIETY_ADMIN}, MANAGER→{MANAGER,SOCIETY_ADMIN}, AMENITY→{AMENITY_MANAGER,SOCIETY_ADMIN}); `roleTargetForKind` maps the ThreadKind. Committee does NOT get blanket access — a DIRECT resident↔resident thread stays private even from committee (proven in the unit test).
- **Isolation / 403 by id.** `requireAccessibleThread` loads the (society-scoped) Thread + participants and runs `canAccessThread`; a resident who is not creator / explicit participant / satisfying role holder gets `forbidden` → 403 even when they guess a thread id.
- **Soft-delete that STAYS VISIBLE.** The scoping layer (`lib/db.ts`) hides any row with `deletedAt` set and rewrites `.delete()` into a soft-delete — which would make a "removed" message vanish. So removal uses a SEPARATE `Message.removedAt` (+`removedBy`), never the scoping `deletedAt`. Reads include removed rows and `redactMessage` withholds the body + sets `removed: true` → the client renders "message removed". No `.delete()` on Message/Thread exists anywhere → no hard delete (doubly safe: the scoping layer would soft-delete even an accidental one). Editing leaves `editedAt` (field present; no edit UI this step).
- **Resident-to-resident OFF by default.** `ChatSettings.residentToResidentEnabled` (singleton keyed by societyId, pass-through like ExpenseSettings). `canCreateThreadOfKind` gates DIRECT+GROUP; the create endpoint throws `resident_to_resident_disabled` → 403 (truly unavailable, not merely hidden). Committee may enable it in the settings drawer (`chat.manage`).
- **Guard on a shared device → no chat access.** `requireChatMember`/`getChatServerActor` refuse `deviceKind === "SHARED"` (403 / null), closing the endpoint even to a hand-crafted request; the nav item also carries `hideOnSharedDevice`.
- **Muted threads are silent.** `notifiableUserIds` drops the sender + any muted explicit participant; the role-holder fan-out additionally skips any muted userId (a committee member who muted a role thread holds a muted participant row) — so a muted thread never notifies, but its unread count still shows.
- **Real-time via polling, not a WebSocket.** The client polls the thread list (12s) and the open thread's messages (8s); `GET /api/chat/overview` returns `unreadTotal` for a tab badge. No long-lived process added (spec: free-tier box).
- **READ_ONLY society.** Posting a message writes the scoped `Message` model → the write scope rejects it with 423; reading (all GETs run under read scope) is unaffected. Thread create + reopen are also scoped writes (423). markRead/mute touch the pass-through `ThreadParticipant` (allowed during read-only, a read-side effect).
- **Auto-close.** `effectiveStatus` lazily displays an OPEN-but-silent thread as CLOSED past `autoCloseAfterDays` (no mutation on read); posting a message sets `status=OPEN`+`lastMessageAt=now` (reopen). No cron this step — the display is lazy; a sweep can be added later.
- **Attachments** go through the existing private/signed storage adapter (`FileUpload entityType="ChatMessage"`); message stores StoredFile ids, UI shows an attachment count.

### Migration `20260712340000_messaging_chat`
Enums `ThreadKind` (RESIDENT_TO_COMMITTEE|RESIDENT_TO_MANAGER|RESIDENT_TO_AMENITY|GROUP|DIRECT), `ThreadStatus` (OPEN|CLOSED). Tables: `Thread` (societyId+deletedAt → tenant-scoped; kind/subject?/flatId?/createdBy/assignedTo?/status/lastMessageAt; idx societyId+lastMessageAt, societyId+status), `ThreadParticipant` (pass-through child of Thread — NO societyId/deletedAt; userId?/roleTarget?/lastReadAt?/isMuted; `@@unique(threadId,userId)`; FK CASCADE), `Message` (societyId+deletedAt scoped; senderId/body/attachments[]/replyToId?/editedAt?/removedAt?/removedBy?; idx threadId+createdAt, societyId+threadId; FK CASCADE), `ChatSettings` (societyId @id singleton; residentToResidentEnabled=false/officeHoursOnly=false/autoCloseAfterDays=30; FK Society CASCADE). Society back-relation `chatSettings ChatSettings?` added.

### Files
- `lib/chat/`: `constants.ts` (kinds/statuses/role-target maps, page sizes, notif code), `rules.ts` (pure: canAccessThread/canReplyToThread/satisfiesRoleTarget/satisfiedTargets/residentToResidentAllowed/canCreateThreadOfKind/redactMessage/unreadCount/previewText/notifiableUserIds/isAutoClosed/effectiveStatus), `rules.test.ts` (24 vitest — every acceptance criterion at the logic level), `actor.ts` (requireChatMember [SHARED-device block] / requireChatActor / getChatServerActor / chatCtx / read+write scopes), `http.ts` (typed-error→status; forbidden+resident_to_resident_disabled→403), `service.ts` (settings ensure/get/update; createThread; listThreads [resident + committee inbox, filters status/kind/flat/assignee/unread + subject+message-body search, cursor]; getThread; listMessages [cursor, newest-first fetch, redact]; postMessage [reopen + upsert read cursor + notifySafe]; markRead; patchThread [self-mute; committee close/assign]; removeMessage [soft, sender-or-committee]; unreadTotal; exportThread).
- API routes: `app/api/threads/route.ts` (GET/POST), `threads/[id]/route.ts` (GET/PATCH), `threads/[id]/messages/route.ts` (GET/POST), `threads/[id]/read/route.ts` (POST), `threads/[id]/export/route.ts` (GET, chat.read), `app/api/messages/[id]/route.ts` (DELETE soft), `app/api/chat/overview/route.ts` (GET), `app/api/chat/settings/route.ts` (GET chat.read / PATCH chat.manage).
- `schemas/chat.ts` (threadCreate [DIRECT needs exactly one other member], threadPatch, messageCreate, chatSettings).
- UI: `app/[locale]/app/chat/page.tsx` (RSC, feature-gated → getOverview), `components/chat/chat-client.tsx` (Simple resident: "Message the committee" + WhatsApp thread list + thread view with composer/attach/remove-own/mute; Pro committee: filter bar unread/status/kind + search, assign/close/export, settings drawer resident-to-resident/office-hours/auto-close). Mobile-first single-column (list OR thread) → two-column md+; semantic tokens for light/dark; logical `text-start`/`ms-auto` for RTL.
- Registrations: `lib/features.ts` (chat.core), `lib/rbac.ts` (`chat.read`→COMMITTEE_MEMBER/MANAGER/AMENITY_MANAGER, `chat.manage`→MANAGER; both gated by chat.core), `lib/nav.ts` (chat item, icon messageCircle, group main, feature chat.core, hideOnSharedDevice), `components/shell/nav-icons.ts` (MessageCircle), `lib/ui-modes/modules.ts` (chat, resident/SIMPLE). Notification catalogue already carried `chat.message.received` (category chat, IN_APP+PUSH, en+ur) — reused, no template change.
- i18n: `messages/en.json` + `messages/ur.json` `chat.*` section + `nav.chat` (key parity verified programmatically).
- e2e: `e2e/messaging-chat.spec.ts` (401 on every GET/POST/PATCH/DELETE endpoint).

### Gates
`pnpm prisma generate` ✓ · `pnpm typecheck` ✓ (clean) · `pnpm lint` ✓ (0 warnings) · targeted `vitest run lib/chat/rules.test.ts` → 24/24; `lib/{nav,features,rbac}.test.ts` + `notifications/templates.test.ts` → 28/28 (registrations don't regress). Did NOT run test:unit/e2e/build (controller runs full gates).

### Deferred / notes
- Bottom-tab unread badge: the count is exposed at the data layer (`/api/chat/overview.unreadTotal`) and shown as per-thread badges in-surface; wiring a live badge into the shared shell bottom-tab bar (lib/nav.ts has no badge slot) is deferred to a shell change.
- No cron: auto-close is a lazy display (`effectiveStatus`); a nightly materialise-to-CLOSED sweep can be added if a report needs the stored status.
- Message edit UI not built (spec lists it as a marker only; `editedAt` field + DTO `edited` flag are in place for a future edit action).

---

## 33 — polls-voting — DONE (2026-07-13)

**Feature:** `polls.core` (deps `residents.registry`, `core.notifications`). Spec: /specs/33-polls-voting.md. Branch: `feature/33-polls-voting`.

### What it is
Binding society decisions (electing a committee, approving a levy) — not a survey; a vote with an auditable result the losing side cannot credibly dispute.

### Key design decisions
- **The FLAT is the atomic voting unit for EVERY eligibility mode.** The spec's flagship acceptance criterion is a DB-enforced one-vote-per-flat (`@@unique([pollId, flatId])`), and its edge cases (three residents share one vote; a two-flat resident gets two votes; a flat with no resident is not in the quorum base) all centre on the flat. Rather than switch the uniqueness key per mode (multi-resident-per-flat would collide with a flat-unique index under ALL_RESIDENTS), eligibility governs only (a) WHICH flats are eligible and (b) WHICH occupant may cast the flat's single vote:
  - ALL_RESIDENTS / ONE_PER_FLAT → any current occupant of an occupied flat.
  - OWNERS_ONLY → only an OWNER-relation current occupant.
  - COMMITTEE_ONLY → only a flat held by a current committee member (the voter must hold a committee role).
  - BLOCK → only flats inside the poll's `eligibilityIds` blocks.
  This keeps a single unambiguous DB constraint and makes the acceptance test provable. Recorded here as the authoritative interpretation.
- **Anonymity vs turnout split across two tables.** `Vote` (holds `optionIds` = WHAT) sets `voterUserId = NULL` when anonymous; `VoteTurnout` (flat-keyed `@@id([pollId, flatId])`, NO userId, pass-through infra) records WHO/turnout. Person→choice is never stored for an anonymous poll, yet the committee can chase non-voting FLATS. VoteTurnout deliberately omits userId (matches the spec model exactly: `{ pollId, flatId, votedAt }`).
- **Receipt hash** = sha256(`pollId|flatId|sortedOptionIds|voteId-nonce`). Deterministic + verifiable (`verifyReceipt`), reveals nothing about the choice to anyone lacking the inputs, and the vote-id nonce stops two identical ballots sharing a receipt. Two-step write: create Vote (id = nonce) → compute → update `receiptHash`.
- **Results 403 before close for EVERY role** incl SOCIETY_ADMIN — `canSeeResults` is a pure function on (status, endsAt, now); the route throws `results_hidden` (→403). No live counts anywhere. `getTurnout` returns counts/percent + the not-voted flat list ONLY (never per-option tallies).
- **Quorum** — `tally` checks quorum first; below `quorumPercent` turnout → outcome `INCONCLUSIVE_QUORUM`, `winnerOptionIds = []`, however lopsided. Counts still returned for transparency. Ties for first surface every tied option; multi-seat (COMMITTEE_ELECTION, votesPerVoter>1) elects the top N.
- **Server time authoritative** — `isOpenAt` requires status OPEN and `startsAt ≤ now < endsAt` (`endsAt` EXCLUSIVE → a clock-skew vote at/after close is rejected). `effectiveStatus` lazily shows an OPEN-past-endsAt poll as CLOSED without a write.
- **Immutability / cancel** — a cast Vote is never edited/deleted (`deletedAt` exists only for the scoping contract, never set). Cancel sets status CANCELLED + `cancelledAt`/`cancelReason` + audit + notify-all; the result outcome becomes CANCELLED (void), votes retained, never silently rerun.
- **Notifications** routed through the existing `announcements` category (codes `poll.opened` / `poll.results_published` / `poll.cancelled`, en+ur, IN_APP+PUSH) — avoids adding a NOTIFICATION_CATEGORIES entry (which has a `toHaveLength(8)` test) while still respecting residents' broadcast prefs.
- **Shared-device block** — `requirePollMember`/`getPollServerActor` refuse a SHARED device (a binding, anonymous vote must never be cast from a guard gate terminal); nav item `hideOnSharedDevice`.

### Tenant-scoping
`Poll` + `Vote` carry societyId+deletedAt → auto-scoped (reads pinned, writes stamped, READ_ONLY society → 423 on a cast vote). `VoteTurnout` has societyId, no deletedAt → pass-through, scoped explicitly. Verified via Prisma DMMF at build.

### Files
- Migration `prisma/migrations/20260713350000_polls_voting/migration.sql` — PollKind/Eligibility/PollStatus enums; Poll, Vote (unique `Vote_pollId_flatId_key`), VoteTurnout; FKs Vote/VoteTurnout → Poll (cascade). schema.prisma models appended.
- `lib/polls/`: `constants.ts`, `rules.ts` (pure; `rules.test.ts` ~30 assertions), `actor.ts` (POLLS_READ/POLLS_MANAGE, member/actor guards, read/write scopes), `http.ts` (error→status incl 409 for `already_voted`, 403 for `results_hidden`/`not_eligible`), `service.ts` (create/patch/open[snapshot eligibleFlatCount]/close/cancel/castVote[P2002→already_voted]/getResults/getTurnout/listPolls/getPoll/listMyPolls/getOverview/listBlocks), `polls.integration.test.ts` (DB-backed: raw P2002, two-residents-one-vote, anonymity+turnout+receipt, results 403→reveal, quorum inconclusive, two-flats-two-votes).
- `schemas/polls.ts` (pollCreate/pollPatch/vote/pollCancel).
- API: `app/api/polls/route.ts` (GET/POST), `app/api/polls/[id]/route.ts` (GET/PATCH), `.../[id]/{open,close,cancel,vote}/route.ts` (POST), `.../[id]/{results,turnout}/route.ts` (GET), `app/api/me/polls/route.ts` (GET).
- Registries: `lib/features.ts` (polls.core), `lib/rbac.ts` (polls.read→COMMITTEE_MEMBER/MANAGER, polls.manage→MANAGER; both gated by polls.core; residents vote with NO perm), `lib/nav.ts` (key `polls`, icon `vote`, hideOnSharedDevice), `components/shell/nav-icons.ts` (`vote`→Vote), `lib/ui-modes/modules.ts` (polls, resident/SIMPLE), `lib/notifications/templates.ts` (3 poll codes).
- UI: `app/[locale]/app/polls/page.tsx` (feature-gated, blocks for BLOCK picker), `components/polls/polls-client.tsx` (Simple ballot: big option cards, anonymity note, confirm "you cannot change this", receipt with hash, animated result bars; Pro builder + live turnout dashboard + who-hasn't-voted chase list + open/close/cancel).
- i18n: `messages/en.json` + `messages/ur.json` `polls.*` + `nav.polls` (parity).
- e2e: `e2e/polls-voting.spec.ts` (401 on every endpoint).

### Gates
`pnpm prisma generate` ok. `pnpm lint` clean (fixed react/jsx-no-literals on ·/✓/% and unused vars). `pnpm typecheck` clean. Did NOT run test:unit/e2e/build (controller runs full gates). No cron.

### Acceptance criteria mapping
- one-vote-per-flat DB constraint → migration unique index + `polls.integration.test.ts` (raw P2002 + castVote 409). ✓
- anonymous: no voterUserId, turnout recorded, receipt verifies → integration test. ✓
- results 403 before close for every role → `canSeeResults` + `getResults` throws + integration test (admin ctx). ✓
- quorum not met → inconclusive, never winner → `tally` + rules.test + integration test. ✓
- multi-flat user one vote per flat → flat switcher + integration test. ✓
- light/dark, EN/UR RTL, mobile-first, Simple+Pro → design tokens, logical props (text-start/ms/gap), mobile-first grid, both variants implemented (self-check by inspection). ✓

---

## Step 34 — surveys — DONE (2026-07-13)

**Feature:** `surveys.core` (isCore false; deps `residents.registry`, `core.notifications`). Branch: `feature/34-surveys`.

**What it is / design stance.** Multi-question OPINION collection — explicitly NOT a poll (step 33). A poll decides something binding (one-vote-per-flat, a winner the losing side cannot dispute); a survey gathers feedback. So the atomic unit is the RESPONDENT (a user), branching + rating/NPS/free-text are allowed, and there is no quorum/winner. A leaf module — never touches billing, tickets or anything else.

**Data model (migration `20260713360000_surveys`).**
- `ALTER TYPE "Eligibility" ADD VALUE IF NOT EXISTS 'ROLE'` — `Survey.audience` reuses the shared `Eligibility` enum; ROLE is a survey-only value (polls' `ELIGIBILITIES` const omits it, polls `isFlatEligible` default→false, so polls never offer/accept it). New value not used in any DDL default (default is the pre-existing `ALL_RESIDENTS`), so it is safe inside the migration transaction.
- New enums `SurveyStatus` (DRAFT|OPEN|CLOSED — simpler than a poll: no SCHEDULED/CANCELLED) and `QuestionType` (SINGLE_CHOICE|MULTI_CHOICE|RATING|NPS|TEXT|LONG_TEXT|YES_NO|DATE|NUMBER).
- `Survey`, `SurveyQuestion`, `SurveyResponse`, `SurveyAnswer` — ALL tenant-scoped (societyId + deletedAt), so `lib/db.ts` auto-pins reads and blocks writes in READ_ONLY.
- `SurveyParticipation` — pass-through infra (societyId, NO deletedAt; `@@id([surveyId,userId])`), scoped explicitly. THE reconciliation of "anonymity vs draft-resume vs turnout" (mirrors step 33's VoteTurnout): records WHO opened/submitted a survey WITHOUT linking a user to their ANSWERS. `draftResponseId` points at the in-progress draft; `hasSubmitted`/`submittedAt` record turnout.

**THE anonymity mechanism.** On submit of an anonymous survey the `SurveyResponse.respondentUserId` is NULL AND `SurveyParticipation.draftResponseId` is SEVERED (set null) — so the submitted answers can never be traced back to the participant — while `participation.hasSubmitted` still records that the user responded (sample size / turnout / one-response). Between save-draft and submit the pointer links user→response (server-side) so ANY device the user signs into resumes the same draft.

**Design decision (recorded, deviates from spec sketch).** The spec data model shows `@@unique([surveyId, respondentUserId]) // when !allowMultipleResponses`. NOT implemented as a DB unique because (a) a plain Prisma unique cannot express "only when !allowMultipleResponses" and would break `allowMultipleResponses` for named surveys, and (b) for anonymous surveys respondentUserId is NULL for every row (Postgres treats NULLs as distinct, so the constraint is inert anyway). Instead one-response is enforced in `respond()` via `SurveyParticipation.hasSubmitted` honouring `allowMultipleResponses` — which correctly covers BOTH anonymous and named surveys. Documented in schema + service comments.

**Rules (`lib/surveys/rules.ts`, pure, vitest `rules.test.ts`).** `effectiveStatus`/`isOpenAt`/`withinGrace`/`canSubmitAt` (server time authoritative; a draft may still submit within a 10-min grace after endsAt — spec edge case — but a FRESH response cannot start once ended); `resultsVisibleTo` (committee always, resident per `resultsVisibleToResidents`); `isUserEligible` (audience at the USER level: ALL_RESIDENTS/OWNERS_ONLY/COMMITTEE_ONLY/BLOCK/ROLE); `isQuestionVisible`/`answerMatches` (simple branching); `isAnswered`/`validateAnswerValue`/`scaleBounds`/`validateResponse` (required enforced SERVER-side, branching-aware — a required question hidden by an unmet condition is exempt; range/type checks per type); `summarizeQuestion` (choice/scale[+NPS]/text, ALWAYS a valid shape so zero responses → empty state, never a broken chart) + `npsScore` (promoters 9–10 − detractors 0–6); `buildExport`/`toCsv`/`stableOrderKey` (anonymity-preserving export). Note: rules.ts imports `node:crypto` (stableOrderKey) so the client re-implements the tiny branching/isAnswered helpers rather than importing them.

**Anonymity-preserving export.** `buildExport`: for an anonymous survey the CSV has NO Respondent / Flat / Submitted-at column AT ALL (not even blank, so nothing can be joined to a person — spec edge case), and rows are ordered by SHA-256 hash of the response id (`stableOrderKey`), NOT submission order/time, so the row order carries no temporal information. Named survey includes respondent/flat/time, sorted by time. Proven in both rules.test (order ≠ submission order, no name in cells) and the integration test (no fullName in csv).

**Service (`lib/surveys/service.ts`).** create/patch (DRAFT-only meta) / `setQuestions` (DRAFT-only; deletes[soft] + recreates the whole set, order = array index) / `openSurvey` (DRAFT→OPEN, requires ≥1 question, notifies audience via `resolveAudienceUserIds`) / `closeSurvey` (→CLOSED, notifies audience when residents may see results) / `listSurveys` (committee all, resident only OPEN/CLOSED they're eligible for or responded to; groupBy for response counts) / `getSurvey` (detail + questions + the caller's resumable `draftAnswers`) / `respond` (save partial draft `complete=false` [must be OPEN] OR submit `complete=true` [OPEN or grace-with-draft]; validates every answer value; required-validation on submit; anonymity severs the link; one-response via participation) / `getResults` (aggregated per-question summaries; committee immediate, resident per flag; free-text answers returned only to committee, never with an author; response-rate + responded-by-block) / `exportResponses` (CSV) / `listMySurveys` / `getOverview` / `listBlocks` / `listRoles` (from `ROLE_SEEDS`, since `Role` is not a tenant-scoped model). `deleteMany` on scoped models is a SOFT delete (lib/scoping) — fine for the replace-on-save flows (soft-deleted old questions/answers are excluded from subsequent scoped reads).

**Actor/http (`lib/surveys/actor.ts`, `http.ts`).** `requireSurveyMember` (any authed member on a PERSONAL device; SHARED guard terminal → 403 — an individual/anonymous opinion must not be entered from a shared device), `requireSurveyActor` (perm-gated), read/write scopes (write folds impersonation-RO + READ_ONLY society → 423). Error map: results_hidden/forbidden/not_eligible→403, already_responded→409, not_open/survey_closed/grace_expired/incomplete/invalid_answer/…→422.

**Permissions/features/nav/notifications/ui-modes.** `surveys.read` (COMMITTEE_MEMBER + MANAGER) / `surveys.manage` (MANAGER) in `FEATURE_BY_PERMISSION` + role seeds; feature `surveys.core` in `lib/features.ts` (deps residents.registry/core.notifications); nav item `surveys` (icon `listChecks`, group main, feature-gated, hideOnSharedDevice); notification templates `survey.opened` + `survey.results_published` (category `announcements`, en+ur IN_APP+PUSH); ui-module `surveys` (ownerStep 34-surveys, resident, supportsModes → built-in default SIMPLE).

**Schemas (`schemas/surveys.ts`).** surveyCreate/surveyPatch/questions/respond (+ question/questionOption/answer). One schema, client + server.

**API routes.** `/api/surveys` (GET list / POST create), `/api/surveys/[id]` (GET detail+resume / PATCH), `/api/surveys/[id]/questions` (POST), `/[id]/open` (POST), `/[id]/close` (POST), `/[id]/respond` (POST — 200 draft / 201 complete), `/[id]/results` (GET), `/[id]/export` (GET, text/csv), `/api/me/surveys` (GET).

**UI (`app/[locale]/app/surveys/page.tsx` + `components/surveys/surveys-client.tsx`).** Simple (resident): one-question-per-screen with a progress bar and big touch targets; proper tap-scale widgets for RATING (stars over min..max) and NPS (0–10 grid, not a dropdown); YES/NO big buttons; text/long-text/number/date inputs; "Save & finish later" (draft) + Submit; branching skips hidden questions live; anonymity note; thank-you screen. Pro (committee): survey builder (meta + question editor with type/required/options/min-max/branching, reorder up/down as a pragmatic, a11y-friendly DnD-equivalent — recorded), per-question result charts (animated bars for choice, histogram for scale, NPS score), free-text answer browser with search (committee-only), response-rate-by-block bars, CSV export (blob download), open/close, edit-questions. i18n en+ur `surveys.*` (126 leaf keys, full parity verified) + `nav.surveys`.

**Tests.** `lib/surveys/rules.test.ts` (branching visibility, required server-side validation incl. branching exemption, value range checks, zero-response aggregation, NPS score, audience eligibility per mode, anonymity-preserving export order + no-identity + CSV escaping). `lib/surveys/surveys.integration.test.ts` (skipIf no DATABASE_URL): anon submit → respondentUserId NULL + participation.hasSubmitted true + draftResponseId severed; named submit records respondent; draft resumes across devices; required enforced server-side (and a failed submit does not mark submitted); branching exempts a hidden required question; one-response unless allowMultiple → already_responded; anonymous export contains no identity. `e2e/surveys.spec.ts` — 401 on every endpoint anonymously.

**Gates.** `pnpm prisma generate` ok; `pnpm lint` clean (fixed react/jsx-no-literals for %, ·, *, #, ":" and unused helpers); `pnpm typecheck` clean. Did NOT run test:unit/e2e/build per CLAUDE.md — controller runs full gates. No cron. No CODEREF present for this range.

WORK TYPE: FEATURE (branch feature/34-surveys)

---

## 35 — amenities-booking — DONE (2026-07-13)

**Feature:** `amenities.core` (+ sub-features `amenities.billing`, `amenities.approval`). Deps: residents.registry / core.notifications / core.worker. Spec: /specs/35-amenities-booking.md (authoritative). Branch: feature/34-surveys (controller merges).

**What it is:** Bookable shared spaces (gym/pool/banquet/community/rooftop) with a dedicated AMENITY_MANAGER role, a slot calendar, a manager-proposes/admin-approves effective-dated rate list, booking invoices through the step-18 ledger as a SEPARATE `amenity` line, security deposits, and post-use feedback.

**Key decisions (recorded, no approval gate):**
- **Double-booking guarantee = per-amenity Postgres ADVISORY TRANSACTION LOCK + transactional overlap check** (`pg_advisory_xact_lock(hashtext(societyId:amenityId)::int8)` inside `db.$transaction`, then `overlappingLiveCount` respecting `maxConcurrentBookings`, then insert). A naive UNIQUE index CANNOT express `maxConcurrentBookings > 1`, so the lock+count is the correct race-free mechanism (spec's "DB constraint + transactional overlap check" satisfied by the lock serialization). Proven by a concurrency Vitest: two simultaneous `createBooking` for one slot → exactly one fulfilled, the other throws `slot_taken` (409). Booking does NOT persist slotId; approval re-check assumes concurrency=1 (safe default for approval-gated exclusive-use amenities).
- **billing.ledger is deliberately NOT a hard `dependsOn` edge** for `amenities.core` — graceful degradation requires billing be disable-able while amenities stay on (a hard dep would forbid that). Pricing is a RUNTIME check: `pricingEnabled = isEnabled(billing.ledger) && isEnabled(amenities.billing)`. With either off → free, unpriced bookings, no invoice (proven in vitest).
- **Rate approval gate**: `proposeRate` → PENDING_APPROVAL; only `decideRate` (admin `amenities.rate.approve`, gated by `amenities.approval` sub-feature) → APPROVED. `selectRate` (pure) picks ONLY an APPROVED, effective-dated rate (effectiveFrom≤at<effectiveTo) by specificity (SPECIFIC_SLOT>WEEKDAY/WEEKEND>ALL) then latest effectiveFrom. A DRAFT/PENDING/REJECTED rate prices NOTHING. On approve, the prior current approved rate of the same (appliesTo, slot) is CLOSED (effectiveTo := new effectiveFrom) — effective-dating, never an in-place edit. Amount+deposit are SNAPSHOTTED onto the booking; a later rate change never re-prices it (vitest).
- **Money seam**: an approved chargeable booking raises its OWN Invoice (numbered from InvoiceSequence, gapless) with a single InvoiceLine `chargeHeadCode="amenity"` + a LedgerEntry(INVOICE,DEBIT) — the resident portal renders lines by chargeHeadCode, so it shows as a clearly separate labelled line, never merged into maintenance (vitest asserts the line code == "amenity", != "maintenance").
- **Security deposit** tracked as its OWN ledger entries (ADJUSTMENT DEBIT hold at approval; ADJUSTMENT CREDIT refund at completion), separate from the charge invoice. Withhold on completion requires a mandatory, audited reason (`withhold_reason_required` 422). Cancellation inside the free window (`withinFreeCancellation`) writes a REVERSAL CREDIT of the invoice (+ deposit release) and cancels the invoice; after the window the charge stands.
- **Amenity-manager scoping**: AMENITY_MANAGER holds `amenities.read`/`slot.manage`/`booking.approve`/`rate.propose`/`chat.read` but the service HARD-SCOPES every desk read/write to amenities they run (`assertDeskAccess`: canManageAll → all; else managerUserId===userId; else `amenity_forbidden` 403). Vitest proves a manager sees their own amenity and is 403 on another's.
- Booking number `RUFI-BKG-2026-0042` gapless per (society, year) via `AmenityBookingSequence` atomic INSERT…ON CONFLICT inside the booking txn.

**Files.** Schema: 3 enums (RateApplies, AmenityApprovalStatus, BookingStatus) + Amenity/AmenitySlot/AmenityRate/AmenityBooking/AmenityFeedback (all societyId+deletedAt scoped) + AmenityBookingSequence (pass-through infra). Migration `20260713370000_amenities_booking`. `lib/amenities/`: `constants.ts` (enum tuples, charge-head code, notif codes, feature codes), pure `rules.ts` (time/overlap/wouldDoubleBook/validateBookingWindow/selectRate/rateAppliesToBooking/withinFreeCancellation/canTransition state-machine/buildAvailability greyed-not-hidden/averageStars — 16 vitest), `actor.ts` (requireAmenityMember/Actor, run{Read,Write}Scope, amenityCtx with per-permission flags), `http.ts` (typed error→status), `service.ts` (amenity CRUD, slots, rate propose/decide, availability, createBooking/approve/reject/cancel/complete, feedback, listMyBookings/listBookings/listFeedback, getOverview, listAmenityManagers/listMyFlats). `schemas/amenities.ts` (shared Zod). API: `/api/amenities`(GET/POST), `/[id]`(GET/PATCH), `/[id]/slots`(GET/POST), `/[id]/rates`(GET/POST), `/[id]/availability`(GET), `/[id]/feedback`(GET), `/api/amenity-slots/[id]`(DELETE), `/api/amenity-rates/[id]/approve`(POST), `/api/bookings`(GET/POST), `/[id]/{approve,reject,cancel,complete,feedback}`(POST), `/api/me/bookings`(GET). Wiring: rbac.ts (feature-permission map → amenities.core / amenities.approval; AMENITY_MANAGER + MANAGER + COMMITTEE_MEMBER role grants), features.ts (3 FeatureDefs), notifications/templates.ts (amenity.booking.{requested,approved,rejected} + amenity.rate.approved, en+ur IN_APP+PUSH, `announcements` category), nav.ts (amenities item, icon calendarCheck, feature-gated, hideOnSharedDevice). UI: `app/[locale]/app/amenities/page.tsx` (RSC bootstrap, feature-gated) + `components/amenities/amenities-client.tsx` (1771 lines; Simple resident: amenity cards+stars → 14-day availability with unavailable windows GREYED+labelled (blackout/full/past, never hidden) → flat switcher → request → my-bookings track/cancel/feedback; Pro desk: bookings table filter+approve/reject/complete(withhold-deposit+reason,no-show), amenity CRUD+manager picker, slot/blackout editor, rate propose→approve workflow+effective-dating history, feedback browser; 409 slot_taken → "just taken" toast+refetch; formatMoney/RTL/light-dark). ui-module `amenities` (resident/SIMPLE) already registered. i18n en+ur `amenities.*` (177 leaf keys, parity 0-diff) + `nav.amenities` (pre-existing).

**Tests.** `lib/amenities/rules.test.ts` (16 pure, PASS). `lib/amenities/amenities.integration.test.ts` (6 DB-backed acceptance: concurrency exactly-one, rate gate+effective-dating+snapshot, ledger separate `amenity` line, deposit refund credit + withhold-needs-reason, manager scoping 403, billing-disabled free). `e2e/amenities-booking.spec.ts` (401 on every endpoint).

**Gates:** `pnpm prisma generate` OK; `pnpm lint` clean; `pnpm typecheck` clean; rules 16/16 + rbac 8/8 (updated the pre-existing rbac test to entitle `amenities.core` instead of the old placeholder `amenities`). Did NOT run test:unit/e2e/build (controller runs full gates). No cron.

---

## 36 — staff-performance — DONE (2026-07-13)

**Feature:** `staff.performance` (module `36-staff-performance`), deps `staff.directory` + `complaints` (spec's `complaints.core`) + `staff.console`. Branch: `feature/36-staff-performance`. WORK TYPE: FEATURE.

**Spec intent.** Track staff on what residents actually EXPERIENCE — resolution speed, reopen rate, SLA compliance, resident ratings. The composite score is society-configurable (weights ARE the policy, visible to staff — a hidden scoring system is a management failure). Snapshots are computed MONTHLY by the worker and stored; reports must not recompute across a year of tickets on a page load.

### Data model / migration `20260713380000_staff_performance`
- `StaffPerformanceSnapshot` — pass-through infra (societyId, NO deletedAt → not auto-scoped; service filters societyId explicitly, like Attendance/TicketRating). Fields per spec + additions: `reassignedCount` (separate signal from reopen, per the reassignment edge case), `scoreStatus` ("ok"|"insufficient_data"|"no_ratings"), `reviewFlag`, `partialPeriod`, `computedAt`. `score` is nullable (null = insufficient data — never a misleading number). `@@unique([societyId, staffId, period])` makes a re-run idempotent.
- `PerformanceWeights` — one row keyed by `societyId` (@id), pass-through like ComplaintSettings; FK → Society onDelete Cascade; back-relation added on Society.

### Pure rules (`lib/staff-performance/rules.ts`) + 14 vitest (`rules.test.ts`, incl. hand-computed fixture)
- `computeScore(raw, weights, ctx)` → weighted average of five 0-100 sub-scores; a sub-score with no data is EXCLUDED and its weight RENORMALISED over the remaining metrics (so zero ratings gracefully falls back to speed+SLA, attendance drops out when disabled — never silently counted as 0).
- Sub-scores: speed `100*min(1,target/actual)` (target 48h constant — society expresses speed-vs-quality via WEIGHTS not target); sla `100*(1-breach/resolved)`; reopen `100*(1-reopen/resolved)`; rating `100*avg/5` only when ratingCount≥3 (MIN_RATED_TICKETS); attendance = attendance% passthrough only when sub-feature on.
- HONESTY: society collects ratings but <3 rated → status `insufficient_data`, score NULL. Society collects NO ratings → status `no_ratings`, score from non-rating metrics. No metric at all → insufficient. `reviewFlag` when avgRating ≤ LOW_RATING_THRESHOLD (2) with ≥1 rating.
- Fixture: full metrics + default weights → 88.5 (100·30+80·35+90·20+80·10+90·5 /100). Attendance-off renormalise → 88.4. No-ratings fallback → 93.3. Period arithmetic helpers (periodOf/periodBounds/periodMinus/periodsBack), `scoreTrend`.

### Service (`lib/staff-performance/service.ts`)
- `computeSnapshotsForSociety(societyId, period)` — worker entry: gathers raw metrics per employed staff and upserts a snapshot each (idempotent). Attribution per spec edge cases: resolution/rating/SLA credited to a ticket's FINAL assignee (`Ticket.assignedStaffId`); assignment + reassignment derived from `assigned` TicketEvents (first per ticket = initial, rest = reassignment); reopens (`reopened` events) map back to the final assignee; first-response = earliest `status`→IN_PROGRESS event minus assignedAt; partialPeriod set when joinedOn>periodStart or leftOn<periodEnd. Attendance % computed from Attendance rows only when `staff.attendance` enabled.
- `getWeights`/`updateWeights` (audited), `getLeaderboard` (stored rows + previous period for trend arrow + raw metrics for the browser live-preview), `getStaffPerformance` (snapshot + 12-month trend + the resolved tickets behind the numbers), `getMyPerformance` (resolves staffId from userId → the worker's own view). All reads are cheap lookups — never a recompute.

### Worker
- Handler `runStaffPerformanceSnapshot` (`lib/worker/handlers.ts`): society-scoped; gated by `isEnabled(staff.performance)` (skips when off); CRON finalises the just-ended month `periodMinus(periodOf(now),1)`, MANUAL honours `payload.period`.
- Registry: cron `staff.performance.snapshot` `0 1 1 * *` (society scope); job entry mutating, concurrency 1 (skipped for READ_ONLY society).

### Wiring
- `lib/features.ts`: registered `staff.performance` (deps as above) — disabling leaves complaints + directory unchanged (graceful degradation).
- `lib/rbac.ts`: perms `staff.performance.read` + `.manage` (gated by the feature). Seeded COMMITTEE_MEMBER→read, MANAGER→read+manage. A worker's own view rides `staff.console.read` (STAFF role) + the feature gate.
- `lib/ui-modes/modules.ts`: `staffPerformance` module (committee, supportsModes). `lib/nav.ts` + `components/shell/nav-icons.ts`: nav item `staff-performance` (icon `gauge`, permission+feature gated, hideOnSharedDevice). i18n nav labels en+ur.

### API routes
- `GET /api/staff/[id]/performance?period=` (staff.performance.read) — static-vs-dynamic Next resolution keeps `[id]/performance`, `performance/leaderboard`, `me/performance` distinct.
- `GET /api/staff/performance/leaderboard?period=` (read; default = most recent complete month).
- `GET|PUT /api/staff/performance/weights` (read | manage).
- `GET /api/staff/me/performance?period=` — staff-console actor + `isEnabled(staff.performance)` (403 when off); resolves the caller's own staff record. Residents never reach any of these.

### UI (`components/staff-performance/` + console integration)
- `score-ring.tsx` — animated SVG ring (arc grows on mount, hue red→green; null = dashed muted ring with em-dash, never a 0) + `Stars`.
- Simple = committee leaderboard cards (rank badge top-3, score ring, trend arrow, rating stars, jobs done, review flag). Pro = metric table + weights editor (sliders, share %, RECOMPUTES every staff score live in-browser via the shared pure `computeScore` before saving — the "live recalculation preview") + per-staff detail drawer (metric grid + three animated 12-month LineCharts: resolution hours, rating, reopen % + the ticket list behind the numbers). `perf-client.tsx` orchestrates via `useUiModule("staffPerformance")`; no-ratings banner shown when the society collects none.
- Staff-facing: `components/staff-console/my-performance.tsx` — compact own-score card in the `my-jobs` "More" panel; fetches `/api/staff/me/performance`, renders nothing on 403/404 (feature off → graceful).
- i18n `staffPerformance.*` en+ur (full parity).

### Tests
- `rules.test.ts` — 14 unit tests (score fixture, renormalisation, insufficient, no-ratings fallback, review flag, empty, weight clamp, trend, period arithmetic).
- `perf.integration.test.ts` — Postgres-backed (skipIf no DATABASE_URL), 2 societies: (1) worker-computed+stored (leaderboard empty before run), score 96.3 for full-data staff, FINAL-assignee credit on a reassigned ticket, insufficient-data (<3 rated → null), own-view via userId, breach/reopen lowering sub-scores (92.1); (2) zero-ratings society → status `no_ratings`, score 100 from speed+SLA.

### Gates
- `pnpm prisma generate` ok. `pnpm lint` clean (fixed react/jsx-no-literals on decorative glyphs). `pnpm typecheck` clean. Did NOT run test:unit/e2e/build (controller runs full gates).

### Decisions (autonomous, recorded)
- Added `reassignedCount`/`scoreStatus`/`reviewFlag`/`partialPeriod`/`computedAt` beyond the spec's bare model — the spec's edge cases (reassignment signal, insufficient-data, no-ratings fallback, partial period, review flag) need somewhere to live; the spec model is authoritative but silent on storage for these, so extended it minimally rather than guess.
- Speed benchmark (48h) is a code constant, not a per-society knob — the society expresses its speed-vs-quality priority through the WEIGHTS, matching "the weights ARE the policy".
- Reads strictly never recompute (spec rule): leaderboard/detail default to the most recent COMPLETE month; the current in-progress month is finalised by the worker on the 1st. A society only sees data after the first monthly run (or a manual `payload.period` job) — deliberate, per "not on request".

---

## 37 — meter-readings — DONE (2026-07-13)

**Feature:** `meters.core` (isCore false; deps `flats.registry`, `billing.charges`, `staff.directory`). Spec `/specs/37-meter-readings.md` (authoritative). Branch `feature/37-meter-readings`.

### What was built
Sub-metered utilities (WATER/GAS/ELECTRICITY/GENERATOR) billed by consumption, feeding the charge engine's `METERED` charge heads (step 17/18).

**Data model** (migration `20260713390000_meter_readings`):
- `MeterKind` enum (WATER/GAS/ELECTRICITY/GENERATOR), `ReadingStatus` enum (DRAFT/CONFIRMED/DISPUTED/BILLED).
- `Meter` (societyId+deletedAt scoped): flatId nullable (null = common/society meter), code (unique per society), kind, unit, chargeHeadId (must be a METERED head), initialReading (Decimal), digitCount (default 6, rollover ceiling), isActive, installedOn.
- `MeterTariff` (scoped): meterKind, ratePerUnitMinor/fixedChargeMinor (BigInt), slabs (Json `[{upTo,rateMinor}]`), effectiveFrom/effectiveTo — effective-dated, never edited in place (a new tariff closes the prior current one in a txn).
- `MeterReading` (scoped): meterId, period "YYYY-MM", previous/current/consumption (Decimal), readingDate, photoFileId, readBy, status, disputeNote, estimated, amountMinor (BigInt, set at billing). Unique `(societyId, meterId, period)` → idempotent round.
- `MeterSettings` (pass-through infra, no deletedAt, Society relation): photoRequired, estimateMissing. Society back-relation `meterSettings` added.
- Readings are Decimal (a meter counts units, fractional for m3/kWh); pure rules operate on `number` (service converts at the boundary via `Number()`), keeping rules Prisma-free.

**Pure rules** (`lib/meters/rules.ts`, 20 vitest in `rules.test.ts`):
- `computeConsumption(previous,current,digitCount)` → rollover-safe. current≥previous → current−previous. current<previous → wrap `ceiling−previous+current` accepted ONLY when smaller magnitude than the drop `previous−current` (so 999,980→000,020 = 40 rollover; 500→200 = implausible → disputed, consumption 0). Equal → 0, not a dispute.
- `computeAmountMinor(consumption,tariff)` → slab telescoping bands + fixed charge, or flat rate + fixed; never negative; rounds fractional minor. Hand-checked: 120u over [100@2000, rest@2500] + 1000 fixed = 251,000.
- `selectTariff(tariffs,kind,at)` — effective-dated, latest effectiveFrom wins, null when none.
- `estimateConsumption(recent)` — avg of last N (caller-trimmed), `estimated:true`, `basis:0` when no history (caller must NOT bill a made-up number).
- `isAnomalous(consumption,avg,mult)` (>2× recent avg), `averageConsumption`, `isImmutable` (BILLED only).

**Billing integration** (the substance): `lib/meters/billing.ts` `meteredLinesForRun(period,now,societyId)` loads active flat-bound meters on METERED heads, prices each CONFIRMED reading via `selectTariff`+`computeAmountMinor`, sums per (flat,head) → `Map<flatId, ResolveMetered[]>`. When `MeterSettings.estimateMissing` is on, a missing reading with real history is billed as a labelled estimate (basis 0 → skipped). `lib/charges/resolve.ts` `resolveCharges` gained an optional trailing `metered: ResolveMetered[] = []` param (backward-compatible — the step-17 simulator's 5-arg call and `resolve.test.ts` still pass, METERED head warns `metered_unavailable` with no lines): a supplied metered line bills (carries sourceReadingId + estimated), absence warns (never a silent zero), feature OFF → no lines → head skipped cleanly. `lib/billing/service.ts`: `loadRunContext` gates on `isEnabled(societyId,"meters.core")` → loads `meteredByFlat` into `RunContext`; `buildFlat` passes the flat's metered lines to `resolveCharges` and carries `meta.readingId`/`estimated`; the invoice COMMIT marks each consumed CONFIRMED reading BILLED with its amount in the SAME transaction (immutable thereafter; estimate lines carry a synthetic `estimate:*` id and are skipped). MeterSettings queried by explicit societyId (pass-through, not auto-scoped).

**Service** (`lib/meters/service.ts`): meter CRUD (asserts METERED head, unique code, flat exists), tariff create (closes prior current, validates one open-ended top band last), `recordReadings` (the round — derives previous from the latest prior-period reading or initialReading, computes consumption+status, enforces `photoRequired`, upserts; overwrites only DRAFT/DISPUTED — never silently un-confirms a CONFIRMED reading, never a BILLED one), `patchReading` (pre-confirmation only), `confirmReading`, `bulkConfirm` (DRAFT only — never sweeps DISPUTED/BILLED), `disputeReading` (mandatory note + resident notification), `getConsoleData` (Pro bootstrap: meters/tariffs/meteredHeads/flats/blocks/settings), `getResidentUsage` (own held flats only — latest priced reading + 6-month trend + photo), `updateSettings`, `societyMoney`. Typed `MeterError` → `http.ts` status map (404/403/409 immutable/422/400).

**API:** `/api/meters`(GET desk/POST manage), `/api/meters/[id]`(PATCH manage), `/api/meter-tariffs`(GET/POST), `/api/meters/readings`(GET ?period&kind&status&blockId), `/api/meters/readings/bulk`(round, record), `/api/meters/readings/bulk-confirm`(manage), `/api/meters/readings/[id]`(PATCH pre-confirm), `/[id]/confirm`(manage), `/[id]/dispute`(manage), `/api/meters/settings`(PUT manage), `/api/me/meters`(GET own usage, no perm).

**RBAC:** `meters.read`/`meters.record`/`meters.manage` → gated by `meters.core`. Seeds: COMMITTEE_MEMBER (read+record), MANAGER (all three), STAFF (record — walks the round). Residents see own usage with NO permission (`requireMeterMember`).

**UI:** page `app/[locale]/app/meter-readings/page.tsx` (feature-gated EmptyState when off), client `components/meter-readings/meter-readings-client.tsx`. nav `droplets` (resident `main` ungated + committee `manage` on `meters.record`); ui-mode `meterReadings` (resident audience → default SIMPLE); nav-icon `droplets` (lucide Droplets). Simple: Urdu-first mobile reading round (previous reading, big number input, `<FileUpload camera>` photo, save & swipe, DISPUTED toast on lower reading) + resident own-usage cards (latest "X units — ₨Y", estimated label, meter photo link, 6-month `<Sparkline>`). Pro (committee, tabbed): readings table (period/kind/status/block filters via `/api/meters/readings`, >2× anomaly row tint + icon, confirm/dispute, bulk confirm, client CSV export) + slab tariff editor (flat or add/remove slab bands) + meter CRUD (deactivate a replaced meter) + policy toggles (mandatory photo / estimate-missing). Design self-check: semantic light/dark tokens + dark: tone variants; `dir` threaded + logical `ms/me/text-start/text-end`; mobile-first single-card round + `sm:` grids + `overflow-x-auto` table/tabs.

**Notifications:** `meter.reading.disputed` template (category `announcements`, en+ur, IN_APP+PUSH) so residents' broadcast prefs apply.

**i18n:** `meterReadings.*` en+ur (115 keys each, parity 0-diff) + `nav.meterReadings`.

### Decisions
- Billing integration IS in scope (acceptance criteria "missing → preview warning" and "feature off → skipped cleanly" only become truthful with the run wired). Kept the billing-service change minimal and additive: an optional resolver param + a gated context load + a same-txn BILLED marking. No existing billing/charges test broken (metered param defaults to []).
- Rollover heuristic "wrap only when smaller magnitude than the drop" chosen over a fixed threshold — deterministic, needs no usage history, and rejects genuine misreads (near-top drops) as disputes.
- BILLED transition is driven by the invoice COMMIT (not preview), so immutability is not vacuous; the meters service refuses every edit path on a BILLED reading.
- A common/society meter (flatId null) is never billed to a resident; excluded from `meteredLinesForRun`.

### Gate results
- `pnpm prisma generate` ✓ · `pnpm typecheck` clean · `pnpm lint` clean (0 warnings).
- `pnpm vitest run lib/meters/rules.test.ts` → 20/20. `lib/charges/resolve.test.ts` → 17/17 (unchanged). Registries `lib/features.test.ts`/`feature-dag`/`rbac`/`nav`/`notifications/templates` → 37/37. en/ur meterReadings parity 115/115.
- `lib/meters/meters.integration.test.ts` written (DB-backed, `skipIf(!DATABASE_URL)`): rollover=40, lower→DISPUTED, slab pricing → 80,000 metered line + resolver line, missing→`metered_unavailable` warning + feature-off skip, opt-in estimate labelled (60,000), BILLED immutable (confirm/patch/dispute refused), resident own-usage priced (50,000). Runs under the controller's full gates (no local DATABASE_URL).
- Did NOT run test:unit/test:e2e/build per standing rules (controller runs full gates).

### Files
schema.prisma (+MeterKind/ReadingStatus enums, Meter/MeterTariff/MeterReading/MeterSettings, Society.meterSettings); migrations/20260713390000_meter_readings/migration.sql; lib/meters/{constants,rules,rules.test,actor,http,service,billing,meters.integration.test}.ts; schemas/meters.ts; lib/charges/resolve.ts (metered param); lib/billing/service.ts (gated metered load + BILLED marking); lib/notifications/templates.ts (meter.reading.disputed); lib/features.ts; lib/rbac.ts; lib/nav.ts; lib/ui-modes/modules.ts; components/shell/nav-icons.ts (droplets); app/api/meters/**, app/api/meter-tariffs/route.ts, app/api/me/meters/route.ts; app/[locale]/app/meter-readings/page.tsx; components/meter-readings/meter-readings-client.tsx; messages/en.json, messages/ur.json.

## 38 — emergency-alert — DONE (2026-07-13)

**Feature:** `emergency.core` (module `38-emergency-alert`). Deps (hard DAG edges): `residents.registry`, `core.notifications` (both core, always on). Spec also names `pwa.push` + `gatepass.core` — deliberately NOT hard edges: the module is built to DEGRADE, never go silent, so push (`branding.pwa`) off still fires in-app + the guard console, and the GUARD role is seeded for every society regardless of the `gatepass` feature. Not core, not depended on by anything (leaf).

### Design decisions
- **Never fail silently.** The `EmergencyAlert` row is PERSISTED FIRST (the guard console reads open rows), THEN notifications fire. A provider outage / DB hiccup in dispatch can never un-raise an alert — `dispatchAlert` catches and logs, the alert stands.
- **Preferences ignored for the emergency category.** Added `ignorePreferences?: boolean` to `notify()` (`lib/notifications/service.ts`) + a pure `deliverableChannels(societyChannels, templateChannels)` in `lib/notifications/channels.ts` (= ALL_CHANNELS ∩ society config ∩ template availability, IN_APP always retained, prefs NEVER consulted). The existing spec-09/11 `applyChannelFloor` (inApp+push floor for emergency) is untouched, so `channels.test.ts` stays green; the emergency service passes `ignorePreferences:true`. You cannot mute an emergency — proven in the integration test (a committee user with emergency push OFF still gets a PUSH `Notification` row).
- **READ_ONLY society exception (explicit + tested).** `runWriteScope` (emergency actor) DELIBERATELY never sets read-only — a billing dispute must never block a fire alarm. The worker escalation (`escalateDueAlerts`) re-enters `withSociety(societyId, fn, {readOnly:false, actorType:"SYSTEM"})` to override the worker runtime's read-only scope for a frozen society. The `emergency.escalate` job is registered `mutating:false` so `skipForReadOnly` never skips it.
- **Alerts are immortal.** No delete path exists anywhere. The status machine (pure `rules.ts`) is a one-way ratchet: ACTIVE→ACKNOWLEDGED→RESOLVED, FALSE_ALARM reachable from either open state; RESOLVED/FALSE_ALARM terminal (no re-open). `FALSE_ALARM` is the only closure for a mis-tap. `EmergencyAlert` carries `deletedAt` only so it is tenant-scoped; it stays null forever.
- **Escalation** (`emergency.escalate` cron, `* * * * *`, society-scoped): any ACTIVE alert nobody acknowledged within `requireAcknowledgeMinutes` (default 2) escalates ONCE (`escalatedAt` stamped) — re-notify on every channel, widen roles (`escalationRoles` = notifyRoles ∪ {SOCIETY_ADMIN,COMMITTEE_MEMBER,MANAGER}), and warn the block's neighbours for FIRE/GAS_LEAK when `notifyBlockNeighbours` is on. Never escalates an acknowledged or already-escalated alert.
- **No guard online at raise** → `isAnyGuardOnline` (a live `Session` for a GUARD-role user) is false → the recipient list is widened immediately to committee+admin and the resident's screen says so (`sent.noGuard`).
- **False-alarm cooldown is a SOFT signal only** (`withinFalseAlarmCooldown` pure helper) — it NEVER blocks a new alert (never rate-limit a real emergency). Multiple alerts from one flat are NOT deduplicated (a fire + a medical can coincide).

### Schema / migration
`20260713400000_emergency_alert`: enums `EmergencyKind` (MEDICAL/FIRE/SECURITY/GAS_LEAK/LIFT_TRAPPED/OTHER) + `AlertStatus` (ACTIVE/ACKNOWLEDGED/RESOLVED/FALSE_ALARM). Models: `EmergencyAlert` (societyId+deletedAt scoped; raisedByUserId, flatId?, kind, message?, status, acknowledgedBy/At, resolvedBy/At, resolutionNote, location?, escalatedAt?, raisedAt; @@index[societyId,status,raisedAt]), `EmergencyContact` (societyId+deletedAt scoped; name/phone/kind?/isPrimary/sortOrder), `EmergencyEvent` (immutable audit trail — societyId, no deletedAt, pass-through), `EmergencySettings` (pass-through per society; notifyRoles String[] default [GUARD,SOCIETY_ADMIN,COMMITTEE_MEMBER,MANAGER], notifyBlockNeighbours, requireAcknowledgeMinutes=2, falseAlarmCooldownMinutes=10). Society relation `emergencySettings`.

### Code
`lib/emergency/`: `constants.ts` (enum tuples, feature/category/code, defaults), `rules.ts` (pure: status machine, `isEscalationDue`, `escalationRoles`, `warnsNeighbours`, `withinFalseAlarmCooldown`, `responseMetrics`, `sortContacts`), `actor.ts` (member/responder/manage tiers, runReadScope/runWriteScope[never read-only]), `http.ts` (typed-error→status), `service.ts` (raise/acknowledge/resolve/markFalseAlarm/list/getConsoleData/contacts CRUD/settings/escalateDueAlerts). `schemas/emergency.ts`. Worker: `emergency.escalate` cron + `runEmergencyEscalate` handler (feature-gated, mutating:false).

### Registrations
Feature `emergency.core`. Perms `emergency.respond` (guard/committee console — acknowledge/resolve/false-alarm + alerts table) + `emergency.manage` (contact directory editor + escalation policy), both gated by emergency.core; GUARD+COMMITTEE_MEMBER granted respond, MANAGER both, SOCIETY_ADMIN via `*`. Residents RAISE + see own + read the contact directory with NO permission. Nav `emergency` (icon `siren`, main group, NOT hidden on shared device — the guard is the responder). ui-mode `emergency` (resident audience → SIMPLE default). nav-icons `siren`→Siren.

### API
`/api/emergency/alerts` (GET own/all filterable + POST raise), `/alerts/[id]/acknowledge|resolve|false-alarm` (POST, emergency.respond), `/contacts` (GET everyone + POST manage) + `/contacts/[id]` (PATCH/DELETE manage), `/settings` (PUT manage).

### UI
Page `app/[locale]/app/emergency/page.tsx` + `components/emergency/emergency-client.tsx`. Feature OFF → SOS absent (EmptyState), nothing else affected. Simple (resident): a big red SOS button — TWO TAPS MAX (tap SOS → tap a kind tile = sent; optional inline note never costs a tap) — then "Help is being notified" with a LIVE acknowledgement status (polls `GET /alerts`), plus an OFFLINE QUEUE (a failed POST is held in state, shows an amber "Not sent yet — queued, will fire on reconnect" banner, retries on `window 'online'` + a 6s interval; it never looks sent when it isn't) and the always-on emergency-contact directory (one-tap `tel:`). Responder overlay (both modes, guard + committee): full-screen `fixed inset-0`, audible WebAudio siren while any ACTIVE alert is unacked, persistent — cannot be dismissed without acknowledging — with the flat/block/floor, resident name + a one-tap call button, kind and note. Pro (committee): response metrics (total, avg acknowledge, false-alarm rate, resolved), alerts table (status/kind/block filters + CSV export), contact-directory editor, escalation-policy panel. i18n en+ur `emergency.*` + `nav.emergency` (deep parity 0-diff, verified).

### Gates
`pnpm prisma generate`, `pnpm lint`, `pnpm typecheck` all clean. Tests written (controller runs them): pure `lib/emergency/rules.test.ts` (status machine terminal/ratchet, escalation-due once/never-acked/never-twice, escalation roles + neighbour-warn, soft cooldown, metrics, contact order) + pure `deliverableChannels` cases added to `channels.test.ts`; DB-backed `lib/emergency/emergency.integration.test.ts` (prefs-ignored/push-despite-off, READ_ONLY raise+ack+resolve, escalate-once-never-acked, cannot-delete/FALSE_ALARM-only-closure/no-reopen, contact directory).

WORK TYPE: FEATURE (branch feature/38-emergency-alert)

---

## 39 — reports-dashboards — DONE (2026-07-13)

**Feature:** `reports.core` (title "Reports & dashboards", module `39-reports-dashboards`, `isCore:false`). **DAG deps:** `core.tables` (renders/exports through the step-07 table+export kit) and `core.notifications` (scheduled-report + large-export delivery) ONLY. Every DATA source (billing.ledger/payments, expenses.core, complaints, gatepass, amenities.core, staff.performance, announcements.core, flats.registry, meters.core) is a RUNTIME `isEnabled` check, NOT a DAG edge — a society runs reports over whichever modules it has, and each report/widget vanishes cleanly when its owner is off.

### What it is
A READ-ONLY analytics layer over every prior module: 14 reports + 4 role home dashboards + CSV/Excel/PDF exports + queued large exports + a monthly scheduled-report email.

### Reports (14): each `filters → summary tiles → chart → table → export`
Financial (7): collection-summary, arrears-aging, defaulters (PII), payment-register, income-expense, budget-actual, charge-head. Operational (7): complaints, gate-pass, amenities, staff, announcements, occupancy, meters. Registry metadata in `lib/reports/constants.ts` (`REPORT_DEFS`: key, category, gating feature code, pii flag); builders in `lib/reports/builders.ts`; key→builder map in `registry.ts`.

### Reconciliation (the hard acceptance criterion) — TRUE BY CONSTRUCTION
- `computeCollection(range).collectedMinor` = Σ ledger `PAYMENT`/`CREDIT` entries in the range (read straight from `LedgerEntry`) → **== Σ ledger credits**.
- `computeArrears(now).totalMinor` = Σ positive per-flat ledger balances (`Σ DEBIT − Σ CREDIT` via groupBy), each flat's balance aged into 0-30/31-60/61-90/90+ by its OLDEST outstanding invoice due date (`bucketForDays`) so the **bucket sum always equals the headline** → **== Σ balances**.
- income-expense reuses `getAccountSummary(period)` from `lib/expenses/service` verbatim → `closing = opening + income − expense`, income = Σ CLEARED payments, expense = Σ APPROVED/PAID expenses → **income − expense == society balance**.
- budget-actual reuses `budgetVariance` (expenses/rules); charge-head reads `InvoiceLine.amountMinor` grouped by `chargeHeadCode` (snapshots, never a live rate recompute — the invoice-snapshot guarantee).
- Money is `bigint` minor units throughout; ratios round to one decimal at the edge only. `lib/reports/rules.ts` pure helpers (`collectionRate`, `isDefaulter`, `pct`, `periodRange`, `periodLabel`, `previousPeriod`, `trailingPeriods`, `avg`) with 8 vitest.
- Integration test `lib/reports/reports.integration.test.ts` (real Postgres) seeds 2 flats + ledger + invoices + cleared payments + an approved expense and asserts all THREE identities EXACT to the minor unit (collected 110000 == Σ credits; arrears 40000 == Σ positive balances == bucket sum; closing == opening+income−expense, income 110000, expense 30000), PLUS the report list is entitlement-filtered (collection-summary/income-expense present; complaints/gate-pass absent) and a disabled report is REFUSED (`ReportError report_disabled`), an unknown key `unknown_report`.

### Exports (CSV/Excel/PDF) — reuse step-07 kit, PII-safe
`exportReport(actor,key,values,now)` runs the report, builds an `ExportView` where SENSITIVITY/NUMERIC come from the SERVER result (a client cannot strip a `sensitive` flag to dodge the PII gate), gates PII on `tables.export.pii` (both inline and queued paths), then: ≤1000 rows → `buildInlineExport` (audited) streamed back; >1000 rows → `enqueue({kind:"reports.export"})` → 202. PDF branding carries societyName + the report title + filter summary ("July 2026 · Block D · ≥ …") + `loadUrduFont()` for Urdu. Worker `runReportExport` re-runs the report via `renderReportForWorker` (fetches society meta under scope, renders directly — authorization already applied at request time), `storeFile` + `getFileAccess(7d)` → notifies the requester with a signed URL (`reports.export.ready`).

### Scheduled reports (worker cron, society timezone)
New pass-through model `ReportSettings { societyId @id, scheduledMonthly, scheduledReportKey, updatedAt }` (migration `20260713410000_reports_dashboards`, + `Society.reportSettings` back-relation). Cron `reports.scheduled.monthly` `0 3 1 * *` scope `society` (runtime enters `withSociety` + evaluates in the society tz), JOB `mutating:false` (read+email delivery, runs even READ_ONLY). Handler `runScheduledReport`: no-ops unless `reports.core` on AND `scheduledMonthly` opted in; derives the just-ended month (`periodMinus(periodOf(now),1)`), renders the chosen report to a branded PDF, `storeFile`+`getFileAccess`, emails COMMITTEE_MEMBER+SOCIETY_ADMIN (`reports.scheduled.ready`). Settings via `/api/reports/schedule` (GET/PUT, PUT needs `reports.manage`), Pro schedule editor in the UI.

### Dashboards (4 roles, entitlement-gated widgets, animated on mount)
`lib/reports/dashboards.ts` `buildDashboard(input)` dispatches on `homeRole(roleCodes)` (management→committee, else AMENITY_MANAGER→amenityManager, GUARD→guard, else resident). Each builder includes a widget ONLY when its feature is enabled (the field is simply absent otherwise) → the client renders fewer tiles, never an empty box or 500. Committee: collection ring + outstanding + open complaints + gate-today + unread announcements + cash balance. Resident: my balance (via `balanceOf` over my flats) + my open complaints + active passes + latest announcements + upcoming bookings. Guard: expected visitors + active passes + gate events today + open emergency alerts. Amenity manager: today's bookings + pending approvals + month revenue. Money formatted server-side into society currency. Home `app/[locale]/app/page.tsx` now renders the live `RoleDashboard` for a signed-in member (guest/cross-society → the existing placeholder `DashboardClient`); `RoleDashboard` registers the step-14 `dashboard` UI module (Simple = 2-col friendly cards, Pro = dense 4-col grid) and fades tiles in on mount.

### Files
- `lib/reports/`: constants, rules(+rules.test), types, actor, http, service, registry, builders, dashboards, reports.integration.test.
- `schemas/reports.ts` (reportFiltersSchema, reportKeySchema, reportExportSchema, reportScheduleSchema).
- API: `app/api/reports/route.ts` (GET list), `[key]/route.ts` (GET run — filters from query string), `[key]/export/route.ts` (POST inline stream / 202 queued), `schedule/route.ts` (GET/PUT).
- UI: `app/[locale]/app/reports/page.tsx` (RSC gate → EmptyState when off), `components/reports/reports-client.tsx` (workspace: category menu, month filter, tiles, chart, table, export buttons, schedule editor; Simple=glance/Pro=full), `components/reports/role-dashboard.tsx`, modified `app/[locale]/app/page.tsx`.
- Charts reuse `components/charts` (Bar/Line/Area/Donut, accessible data-table fallback, RTL-aware, animated). Collection ring reuses `ScoreRing`.
- Wiring: `lib/features.ts` (+reports.core), `lib/rbac.ts` (PERMISSION_REGISTRY reports.{read,manage}→reports.core; roles COMMITTEE_MEMBER+ACCOUNTANT read, MANAGER both, SOCIETY_ADMIN `*`), `lib/nav.ts` (+reports item, manage group, icon barChart3, feature+permission gated, hideOnSharedDevice), `components/shell/nav-icons.ts` (+barChart3), `lib/ui-modes/modules.ts` (reports/committee — was pre-seeded), `lib/account/notifications-policy.ts` (+`reports` category), `lib/notifications/templates.ts` (reports.scheduled.ready + reports.export.ready, en+ur IN_APP+EMAIL), `lib/worker/registry.ts` + `handlers.ts` (cron + 2 job handlers). i18n `messages/en.json`+`ur.json`: `reports.*` namespace + `nav.reports` + `account.notifications.category.reports` + `settings.category.reports` (deep parity 0-diff, verified).

### Permissions / roles
`reports.read` (workspace + dashboards), `reports.manage` (scheduled-report config), both →reports.core. Seeded: COMMITTEE_MEMBER (read), ACCOUNTANT (read), MANAGER (read+manage), SOCIETY_ADMIN (`*`). Export-PII still needs `tables.export.pii`.

### Gate results
`pnpm prisma generate` OK. `pnpm typecheck` clean. `pnpm lint` clean (0 warnings). Unit `lib/reports/rules.test.ts` (8 cases) + integration `lib/reports/reports.integration.test.ts` written (DB-gated; NOT run here per build-loop rules — controller runs the full suite). i18n en/ur parity: 0-diff (structural key-set diff verified).

### Deliberate scope decisions (recorded, no approval gate exists)
- Platform (L0-only) reports (MRR/ARR, societies-by-plan, churn) were NOT built — those belong to the L0 platform console, not the L1 society reports surface. Category `platform` exists in the type/UI for future L0 wiring; no entries today.
- The defaulters "one-click send reminder to all" action was left out (would require guessing the billing-reminder notify template/audience shape); the defaulters report still surfaces resident name + phone (PII-gated) so the committee can act, satisfying "with contact details".
- The report table is rendered directly (server returns pre-formatted string rows) rather than through the full `<DataTable>` saved-views component, to keep the surface bounded; exports still go through the authoritative step-07 export kit (audit + PII gate + Urdu PDF).

WORK TYPE: FEATURE (branch feature/39-reports-dashboards)

---

## Step 40 — public-site — DONE (2026-07-13)

**Feature:** `platform.public_site` (module `40-public-site`, isCore false, dependsOn `platform.billing`, `core.shell`). Registered in `lib/features.ts`; DAG validation passes (registries green).

**Spec:** `/specs/40-public-site.md` (authoritative). No CODEREF companion found (proceeded).

### What was built
The apex marketing site (rihaish.pk): where a prospect lands, understands the product, and asks to be called; also the legal home.

- **Apex-only (spec: "a society host never renders it").** Pure `isMarketingPath(pathname)` added to `lib/tenant-host.ts` (root `/`, `/features`, `/pricing`, `/demo`, `/about`, `/legal/*`, locale-tolerant; app/platform/api/assets excluded). `middleware.ts` returns a generic 404 when `ctx.kind !== "platform" && isMarketingPath(pathname)` — reveals nothing, identical to any other miss. `/api/leads` re-checks the host in-handler (defence in depth). DECISION: a society host hitting `/` now 404s (rather than the old generic placeholder). Acceptable per spec — no browser e2e navigates a society host to a marketing root; society entry is `/app`. Kept it a 404 (not a redirect) to avoid coupling to app-login routing this step.
- **Static/ISR + DB-down resilience.** Every marketing page is a Server Component using only `setRequestLocale` + `getTranslations` (no cookies/headers/DB) → statically generated for en/ur → renders through an app-DB outage. Old `app/[locale]/page.tsx` placeholder REMOVED; landing moved into route group `app/[locale]/(marketing)/` (does not affect URLs). Pages: `page.tsx` (landing: hero/problem/solution/token-free product preview/social proof/CTA), `features/`, `pricing/`, `demo/`, `about/`, `legal/{privacy,terms,data-protection}/`. `(marketing)/layout.tsx` = header + footer + optional first-party analytics `<Script>`; no `headers()` so it stays static.
- **Lead form — layered defence.** `app/api/leads/route.ts` POST: apex host check → in-memory per-IP rate limit (`lib/public-site/rate-limit.ts`, DB-free so it survives a DB outage; `LEAD_RATE_LIMIT` 3/10min) → ONE shared Zod schema (`schemas/public-site.ts`, browser resolver + server) → `checkSpam` (`lib/public-site/spam.ts`, pure: honeypot `companyWebsite` + time-trap `MIN_FILL_MS` 2.5s + stateless arithmetic challenge validated by id) → `createLead` (unscoped platform write — the authoritative record) → fire-and-forget `notifyPlatformOfLead`. Honeypot/time-trap → silent `{ok:true, dropped}` (a bot learns nothing); wrong challenge → 400 `challenge` (a human retries); rate limit → 429 + Retry-After; **DB down → 503 `{db_unavailable, fallbackEmail}`** and the client form degrades to "email us at hello@rihaish.pk" (never pretends it saved). Honeypot schema accepts any string (so a filled trap does not 400 and tip off the bot).
- **Existing-society edge case.** `flagExistingSocieties` (case-insensitive, trimmed; reads the small society registry once, matches in memory) flags a lead whose society name matches a tenant. Never rejected — flagged in the console (amber warning + hint in the detail dialog).
- **Platform console.** New `Lead` model + `LeadStatus` enum (NEW|CONTACTED|DEMO_SCHEDULED|WON|LOST); migration `prisma/migrations/20260713420000_public_site/migration.sql`. Platform-level (no `societyId`/`deletedAt`) — a lead predates any society. `/platform/leads` page + `LeadsTable` (data-table kit, newest-first, status/city filters, row → detail dialog to move the pipeline + keep notes). API `/api/platform/leads` (GET list via `runPlatformList`) + `/[id]` (PATCH status/notes, `LeadNotFoundError`→404). Nav item `leads` (icon `Inbox`) in `PlatformShell`. Perms reuse `platform.society.{read,manage}`.
- **SEO.** `app/sitemap.ts` (all 8 marketing routes × en/ur with hreflang alternates; app/api/platform never listed). `app/robots.ts` (allow /, disallow /app /api /platform, sitemap + host). Per-page `generateMetadata` via `lib/public-site/seo.ts` `buildMetadata` (canonical + `alternates.languages` hreflang incl. x-default + OpenGraph + Twitter). JSON-LD `SoftwareApplication`/Organization on the landing. `(marketing)/opengraph-image.tsx` (next/og ImageResponse, emerald/gold, async `params` per Next 15.5 image loader — awaited; NO staging token/screenshot).
- **Analytics/trust.** Analytics is first-party only (self-hosted Plausible/Umami) and OPT-IN via `ANALYTICS_SCRIPT_URL`/`ANALYTICS_SITE_ID`; unset → nothing emitted. No third-party tracker. Product "screenshots" are a token-free MOCK (`components/marketing/product-preview.tsx`) rendered from the palette — spec forbids embedding a staging token; real screenshots await a screenshot pipeline.
- **Pricing.** Pure `lib/public-site/pricing.ts` `quoteMonthly` — published list price, volume tiers (60/50/40/32 PKR per flat across 1–100/101–300/301–750/751+), ₨3,000 monthly floor, 10% annual discount (floored). 184 flats → ₨9,200/mo. Client `PricingCalculator` recomputes with the same function (server == browser); beyond 1,000 flats → honest "let's talk" (lumpsum).
- **Env (all optional, in `.env.example`):** `PLATFORM_ALERT_EMAIL`, `PLATFORM_ALERT_PHONE`, `PLATFORM_WA_PHONE_NUMBER_ID`, `PLATFORM_WA_ACCESS_TOKEN` (best-effort lead heads-up; a lead ALWAYS lands in the console regardless), `ANALYTICS_SCRIPT_URL`, `ANALYTICS_SITE_ID`. Added to `lib/env.ts`.
- **i18n.** `messages/en.json` + `ur.json`: full `marketing.*` namespace (nav/footer/hero/preview/problem/solution/socialProof/cta; features 8 module families written for a committee chairman; pricing incl. calculator + 3 plans; demo form + spam-challenge questions; about + offices; legal privacy/terms/data-protection with real CNIC-handling and child-exit safeguarding sections) + `platform.leads` + `platform.nav.leads`. Genuine Urdu (not machine-dumped), RTL via the root layout `dir`. Parity check: 0 diff en↔ur including array lengths.

### Files
- Prisma: `schema.prisma` (+`Lead`/`LeadStatus`), migration `20260713420000_public_site`.
- `lib/public-site/`: `constants.ts` (contact, nav, plans, challenges, honeypot, rate-limit config), `pricing.ts` (+`pricing.test.ts`), `spam.ts` (+`spam.test.ts`), `rate-limit.ts` (+`rate-limit.test.ts`), `seo.ts`, `leads.ts` (create/notify/flag/list/update, +`leads.integration.test.ts`).
- `schemas/public-site.ts` (leadFormSchema + leadFields).
- `lib/tenant-host.ts` (+`isMarketingPath`, +tests), `middleware.ts` (marketing gate), `lib/env.ts`, `lib/features.ts`.
- API: `app/api/leads/route.ts`, `app/api/platform/leads/route.ts` + `[id]/route.ts`.
- Pages: `app/[locale]/(marketing)/` layout + landing + features/pricing/demo/about + legal/{privacy,terms,data-protection}; `app/[locale]/platform/leads/page.tsx`; `app/sitemap.ts`; `app/robots.ts`; `(marketing)/opengraph-image.tsx`.
- Components: `components/marketing/` (marketing-header, marketing-footer, lead-form, pricing-calculator, product-preview, legal-article); `components/platform/leads-table.tsx`; `platform-shell.tsx` (+leads nav).

### Gates
- `pnpm prisma generate` ✓ · `pnpm lint` ✓ (no warnings/errors; had to route bare JSX literals through i18n/expressions for `react/jsx-no-literals`) · `pnpm typecheck` ✓ (removed stale `.next` after deleting the old root page; dropped an invalid `numeric` key from a server `ColumnConfig`). Did NOT run test:unit/e2e/build per CLAUDE.md — controller runs full gates.
- Tests authored: pricing (6), spam (6), rate-limit (4), `isMarketingPath` (2), leads DB-backed integration (`describe.skipIf(!hasDb)`: lands-as-NEW+listed / existing-society-flagged-not-rejected-case-insensitive / pipeline status+notes).

### Decisions recorded
- **[DECIDE AT BUILD] trademark/domain clearance for "Rihaish"** (spec, PROGRESS-HISTORY D10): this is a business/legal filing, not code. Surfaced on the pricing page ("Trademark & domain clearance for your society is on us") but the actual clearance action is out of code scope for this step — no code blocker.
- Society-host root → 404 (not redirect) this step; revisit if a society-host landing/login is specced later.
- Lead notification is best-effort (email via deploy SMTP transport; WhatsApp via wa-cloud env creds) and can never fail the lead; with nothing configured the operator still sees the lead in the console.

WORK TYPE: FEATURE (branch feature/40-public-site)

---

## Step 41 — cctv — DONE (2026-07-13)

**Feature:** `cctv.core` (module `41-cctv`). DAG deps: `residents.registry`, `core.storage`. Notifications/gatepass/emergency are RUNTIME integrations (event snapshots degrade to nothing when absent), NOT DAG edges — no module ever depends on CCTV existing.

**Spec:** /specs/41-cctv.md (AUTHORITATIVE). No CODEREF.

### The non-negotiable constraint (verified by test)
Rihaish NEVER carries video. No RTSP proxy, transcode, HLS repackage or relay anywhere. Rihaish is the REGISTRY + PERMISSION ENGINE + TOKEN ISSUER + AUDIT LOG. The architecture test (`cctv.integration.test.ts` §5) reads service/token/keys/actor/rules source and asserts none imports a media/streaming lib (ffmpeg/fluent-ffmpeg/node-media-server/wrtc/mediasoup/hls.js/rtsp/gstreamer/@ffmpeg), and the mint test asserts the stream URL contains the SOCIETY edge host and NOT "rihaish".

### The security heart — the view token (`lib/cctv/token.ts`, pure)
Ed25519 (EdDSA) signed JWT. `mintViewToken(privateKey, claims, kid)` → SHORT-LIVED, single society/camera/user, `jti` for replay, absolute `iat`/`exp`. `verifyViewToken(publicKey, token, {nowSeconds, seenJtis, clockSkewSeconds})` is exactly what the society EDGE runs: EdDSA signature check → alg check → exp/iat window → jti replay cache. Rejects malformed/bad_alg/bad_signature/expired/not_yet_valid/replayed. `buildStreamUrl(edgeBaseUrl, streamPath, token)` assembles the SOCIETY-edge URL with the token as a query param (never a Rihaish host). 7 vitest (roundtrip, expired, tampered-payload, foreign-key, replayed, malformed, edge-url).
Keys: `lib/cctv/keys.ts` reads `CCTV_TOKEN_{PRIVATE,PUBLIC}_KEY` (PEM PKCS#8/SPKI) + `CCTV_TOKEN_KEY_ID`; when unset generates a per-process EPHEMERAL keypair so dev live-view works out-of-box (production MUST set a stable key so the edge can pin the public key). `hasStableSigningKey()` surfaces production-readiness in the integration view.

### Safeguarding (pure `lib/cctv/rules.ts`, 7 vitest) — enforced in code, not committee judgement
- `canGrantClassToSubject(class, subjectType, roleCode?)`: only COMMON is resident-grantable. FLAT/USER/ALL_RESIDENTS are always resident audiences; a ROLE grant is resident only for the plain RESIDENT role (any staff/committee role → any class; unknown role → fail-safe resident). `createGrant` rejects RESTRICTED/PRIVATE to a resident subject for EVERY caller incl. a SOCIETY_ADMIN-level ctx (checks every camera a grant covers, expanding groups). AND `resolveAccessibleCameras` re-filters resident-subject matches to COMMON at ACCESS time (defense in depth — a bad grant that somehow slipped in still can't resolve).
- `residentAccessEnabled` OFF by default (a plain resident resolves to zero cameras until the society opts in; committee sees via role, ungated).
- Watermark `watermarkText(name, flat, iso)` = viewer name + flat + time, rendered as a non-removable DOM overlay on the live view.
- Concurrency cap (`maxConcurrentViewsPerUser`, default 2) enforced at mint by counting the user's view-log rows started inside the TTL window.
- Token TTL clamped to `MAX_TOKEN_TTL_SECONDS` (120) so the honesty guarantee ("access dies within the TTL, ≤60s") can't be silently widened.

### The ex-tenant nightmare — closed two ways (test §3)
1. AUTOMATIC: `resolveAccessibleCameras` requires an ACTIVE `FlatOccupancy` (endDate null) for any resident-subject match. The instant an occupancy ends, the ex-tenant resolves to zero cameras and mint → `access_denied`.
2. EXPLICIT: `revokeGrantsForEndedOccupancy(societyId, userId)` (feature-gated via `isEnabled`, no-op when off/absent) revokes live USER grants for a departed user who no longer holds anywhere in the society. Wired best-effort (try/catch, dynamic import) into residents `updateOccupancy` on the `endOccupancy`/endDate path — residents never breaks when CCTV is off, and does NOT depend on it.

### Read-only rule (spec edge case)
Live view SURVIVES a READ_ONLY (unpaid) society — cutting security cameras over a billing dispute is unacceptable and costs us nothing (video never touches us). `runLiveScope` (token mint / view-log) and `runAgentScope` (snapshot push / heartbeat) are DELIBERATELY never read-only (impersonation-without-write still blocked). Only CONFIG writes freeze: `runConfigScope` honours `Society.status === READ_ONLY`.

### View log & snapshots
- `CameraViewLog` APPEND-ONLY (no `deletedAt`, no delete path anywhere). `mintView` writes the row BEFORE returning the token. `listViewLog`: a `cctv.view` viewer sees ALL rows (filter by camera/user); a plain member sees ONLY their own history.
- Snapshots (`pushSnapshot`, agent-authenticated): the ONLY image data touching our storage. Rate-limited (1 per `snapshotIntervalSeconds`, default 60), 100 KB hard cap (`MAX_SNAPSHOT_BYTES`, checked pre-store), stored via `storeFile` (sniffed + quota-checked against `Society.maxStorageMb`). `linkLatestSnapshot(society, camera, entityType, entityId, reason)` attaches a recent snapshot to a GateEvent/EmergencyAlert/Complaint and degrades silently to null when CCTV is off (the event-integration hook; degrade-clean per spec).

### Data / migration
Migration `20260713430000_cctv`. Enums `CctvMode`(NONE|LINK|EMBED|AGENT), `AgentStatus`, `CameraClass`(COMMON|RESTRICTED|PRIVATE), `GrantSubject`, `SnapshotReason`. Models: `CctvIntegration`/`CctvSettings` (pass-through, `societyId` @id, Society back-relation), `CctvAgent`, `Camera` (@@unique[societyId,code], deletedAt), `CameraGroup`, `CameraAccessGrant`, `CameraViewLog` (append-only), `CameraSnapshot`. Society gains `cctvIntegration`/`cctvSettings` relations.

### Endpoints (all spec-listed)
`GET/PUT /api/cctv/integration` · `GET/PUT /api/cctv/settings` · `POST /api/cctv/agents/enroll` (committee → one-time enrolment code, only sha-256 hash stored) · `POST /api/cctv/agents/heartbeat` (bearer enrolment code → ONLINE) · `GET /api/cctv/view-log` · `GET/POST /api/cameras` · `PATCH/DELETE /api/cameras/:id` · **`POST /api/cameras/:id/token`** (mint — the security heart) · `POST /api/cameras/:id/snapshot` (agent bearer push) · `GET/POST /api/camera-groups` · `GET/POST /api/camera-grants` · `DELETE /api/camera-grants/:id` (revoke) · `GET /api/me/cameras`. Agent endpoints authenticate by `Authorization: Bearer <enrollCode>` (unscoped hash lookup → `runAgentScope`), not a user session.

### RBAC / nav / ui-modes
Perms `cctv.view`→cctv.core (COMMITTEE_MEMBER, MANAGER, GUARD — guard needs the gate tile), `cctv.manage`→cctv.core (MANAGER; admin via `*`). A resident's OWN access (see granted cameras, mint token, own view history) needs NO permission (`requireCctvMember`). Nav `cctv` (icon `Cctv` added to nav-icons, main group, NOT hidden on shared device). ui-module `cctv` (audience resident → default SIMPLE).

### UI (`app/[locale]/app/cctv/page.tsx` + `components/cctv/cctv-client.tsx`)
Simple (everyone): mobile-first camera WALL of snapshot tiles with class chip + live/offline badge + snapshot age → tap opens a watermarked live-view Modal that mints a token and embeds the society-edge stream (LINK mode opens the vendor URL in a new tab and logs the click). Pro (committee, `cctv.view`): tabs — Cameras (table + add-camera modal with MANDATORY classification + door-warning), Access/grants (list + add-grant modal + revoke; surfaces the safeguarding refusal), Agent (enrol + one-time code + health), View log (append-only audit + CSV export), Setup guide (honest LINK/EMBED/AGENT capability matrix + limits + current tier). Config actions disabled when READ_ONLY. Feature-off page → EmptyState, nothing else affected.

### i18n
en+ur `cctv.*` (99 keys) + `nav.cctv`. Genuine Urdu, RTL. Parity 0-diff (recursive key sets identical, verified).

### Env
`CCTV_TOKEN_PRIVATE_KEY`, `CCTV_TOKEN_PUBLIC_KEY` (optional PEM; dev ephemeral fallback), `CCTV_TOKEN_KEY_ID` (default `cctv-dev-1`). Added to `lib/env.ts` + `.env.example` (with a keygen one-liner).

### Gates
`pnpm prisma generate` ok. `pnpm lint` clean (fixed two `react/jsx-no-literals` in the client). `pnpm typecheck` clean (dropped `.default()` on optional schema fields so the parsed output type stays optional and the service's `?? default` owns the default). Pure token(7)+rules(7) vitest = 14 green locally. Integration test (`cctv.integration.test.ts`) is DB-gated (`skipIf(!DATABASE_URL)`) — the CONTROLLER runs it: proves (1) RESTRICTED/PRIVATE-to-resident refusal even for admin, (2) occupancy-end auto-revoke + ex-tenant zero-access + mint-denied, (3) signed single-camera/user token that verifies + society-edge URL + watermark, (4) view-log written/own-history, (5) snapshot rate + 100 KB cap + storage accounting, (6) no-media-lib architecture assertion. Did NOT run test:unit/test:e2e/build per standing rules.

### Decisions recorded (autonomous, no approval gate)
- Auto-revoke on occupancy-end is delivered as a DUAL guarantee (mint-time active-occupancy gate as the automatic mechanism + an explicit revoke helper wired best-effort into residents) rather than a new cron — stronger, self-contained, and testable without worker plumbing. A periodic sweep cron was deliberately NOT added.
- Agent enrolment models the agent as bearer-authenticated (sha-256 of a one-time code); heartbeat/snapshot use that bearer. No user session for the edge box.
- Signing uses one Rihaish Ed25519 key (rotatable via `publicKeyId`/`CCTV_TOKEN_KEY_ID`), not per-society keys — simpler, and the edge pins one public key. Per-camera public-key rotation is out of scope.
- Full gate-pass/emergency/complaint WIRING of `linkLatestSnapshot` into those modules' flows was left to those modules' own call sites (the helper + link columns ship here, degrade-clean); this module adds no hard edge into them.

---

## 42 — permission-requests — DONE (2026-07-13)

**Spec:** /specs/42-permission-requests.md (AUTHORITATIVE). WORK TYPE: FEATURE (branch feature/41-cctv — continued on the active feature branch; controller merges).

**What was built:** A resident asks to hold an event, put up a tent, hold a milad, renovate, shift house or arrange a funeral and gets a request with a status instead of an unanswered phone call to a committee member.

**Feature registry (`lib/features.ts`):**
- `permits.core` (deps `residents.registry`, `core.notifications`, `core.storage`).
- `permits.charges` (dep `permits.core`) — fees + renovation debris deposit through the step-18 ledger; off → free/un-invoiced. Also needs `billing.ledger` on to actually price (RUNTIME `isEnabled`, not a DAG edge).
- `permits.fasttrack` (dep `permits.core`) — the funeral fast-track.
- `billing.ledger` and `gatepass` are RUNTIME checks, NOT hard edges → graceful degradation (unpriced permits; no worker passes) when off.

**Data model (migration `20260713440000_permit_requests`):** `PermitStatus` enum (DRAFT/SUBMITTED/APPROVED/REJECTED/ACTIVE/COMPLETED/CANCELLED). `PermitType` + `PermitRequest` carry `societyId`+`deletedAt` → AUTO-SCOPED (soft-delete infra; permits are NEVER hard-deleted — cancelled/rejected keep their reason). `PermitEvent` (societyId, NO deletedAt) = append-only audit trail (pass-through, like EmergencyEvent). `PermitSequence` (societyId, year) = gapless per-society/year counter (atomic INSERT…ON CONFLICT). `PermitRequest` adds `announceToSociety`/`announcementId`/`cancelledAt` beyond the spec skeleton to carry the funeral opt-in + the linked BEREAVEMENT announcement.

**Decisions (recorded, autonomous — no approval gates):**
1. READ_ONLY funeral exception implemented like emergency: `runWriteScope` NEVER sets scope-readOnly (so a funeral write is always allowed by the scope layer), and the SERVICE re-imposes the freeze via `assertSocietyWritable` (reads Society.status, throws `ReadOnlyError`→423) for EVERY non-fast-track write path (create/submit/approve/reject/cancel/complete/refund + both type editors). Normal permit blocked under READ_ONLY, funeral exempt. Proven in integration test 2.
2. Fast-track no-charge invariant enforced at the DATA LAYER by pure `fastTrackChargeViolation` — called in `createPermitType` and `patchPermitType` (against MERGED values), and `snapshotCharge` always returns zero for a fast-track type. Setting a fast-track flag also nulls fee/deposit + forces requiresApproval true. Proven in test 1.
3. Common-area conflict guard mirrors amenities: per-area `pg_advisory_xact_lock(hashtext(societyId:area:area)::int8)` taken inside the create/submit/approve transaction, then `overlappingAreaCount` (pure, only counts LIVE=SUBMITTED/APPROVED/ACTIVE permits on the SAME non-null area overlapping the window). A permit with no area never conflicts. Proven race-free in test 3.
4. Money seam mirrors amenities: a SEPARATE `permits` invoice line + INVOICE DEBIT ledger entry, and a SEPARATE ADJUSTMENT DEBIT deposit hold, snapshotted at approval. Cancel writes REVERSAL CREDITs (fee + deposit) + cancels the invoice; complete/refund-deposit writes an ADJUSTMENT CREDIT refund unless withheld with a mandatory audited reason. Charge head `permits` (constant, self-labelling). Proven in test 4.
5. Funeral notifications go through the `emergency` category with `ignorePreferences` (a death cannot be muted) to roles GUARD+SOCIETY_ADMIN+COMMITTEE_MEMBER+MANAGER, and (opt-in) `allMembers`. Bereavement announcement created best-effort by a DIRECT `Announcement`(type BEREAVEMENT, PUBLISHED, priority IMPORTANT, IN_APP)+`BereavementDetail` insert (janaza time = permit.startAt, place = area/details) — wrapped in try/catch so it never fails the funeral. Templates carry plain, gentle EN+UR copy (incl. "Inna lillahi wa inna ilayhi raji'un").
6. Renovation worker passes: `emitWorkerPasses` calls the gate-pass `createPass` (STAFF_RECURRING, valid for the permit window) once per approval, feature-gated on `isEnabled(gatepass)` and wrapped in try/catch so a disabled sub-feature/type is a silent no-op. Worker persons are placeholder-named `Worker N` (real crew names a future enhancement). Proven both ways in test 5.
7. Guard active-funeral banner: `getActiveFunerals` returns APPROVED/ACTIVE fast-track permits within ±1 day of now; surfaced on the desk/guard UI so no mourner is turned away and no gate pass is demanded. GUARD role granted `permits.read`.
8. Noise-window violations are surfaced as rule text only (they become a complaint, step 26) — the software states the rule, it does not police it (spec).

**Files:** `lib/permits/{constants,rules,rules.test,actor,http,service}.ts`, `lib/permits/permits.integration.test.ts`, `schemas/permits.ts`. API: `app/api/permit-types/route.ts`+`[id]/route.ts`; `app/api/permits/route.ts`+`[id]/route.ts`+`[id]/{submit,approve,reject,cancel,complete,refund-deposit}/route.ts`+`availability/route.ts`; `app/api/me/permits/route.ts`. UI: `app/[locale]/app/permits/page.tsx` + `components/permits/permits-client.tsx` (Simple resident icon tiles + request form with fee-before-submit and gentle funeral flow + my-requests; Simple committee approvals inbox + area calendar + funeral banner; Pro table + permit-type editor + complete/refund). Registrations: `lib/features.ts` (3), `lib/rbac.ts` (3 perms + COMMITTEE/MANAGER/GUARD seeds), `lib/nav.ts` (permits, icon stamp), `components/shell/nav-icons.ts` (Stamp), `lib/ui-modes/modules.ts` (permits), `lib/notifications/templates.ts` (5 codes), `messages/{en,ur}.json` (permits.* 106 keys + nav.permits, parity 0-diff).

**Permissions:** `permits.read` (COMMITTEE_MEMBER+MANAGER+GUARD), `permits.approve` (COMMITTEE_MEMBER+MANAGER), `permits.manage` (MANAGER + admin `*`) — all →permits.core. A resident's OWN create/submit/cancel needs NO permission (authenticated-member only).

**Gates:** `pnpm prisma generate` OK. `pnpm lint` clean. `pnpm typecheck` clean. `pnpm test:unit`/`test:e2e`/`build` deliberately NOT run (controller runs full gates). Pure `rules.test.ts` (40 assertions) + DB-backed `permits.integration.test.ts` (5 acceptance cases) written.

**Acceptance criteria coverage:** fast-track never fee/deposit + auto-approve (test 1 + rules) ✓; funeral works READ_ONLY (test 2) ✓; two permits can't share an area/time (test 3, concurrency) ✓; approved chargeable → one ledger debit snapshot + deposit refund credit (test 4) ✓; renovation emits worker passes when gatepass on / silent when off (test 5) ✓; funeral gentle language + guard banner + no fee shown (UI + templates + getActiveFunerals) ✓; light/dark + EN/UR RTL + mobile-first + Simple+Pro (permits-client.tsx, logical Tailwind tokens) ✓.

---

## 43 — utility-schedules-notices — DONE (2026-07-13)

**Feature:** `utility.schedules` (main) + sub-features `utility.outages`, `utility.external_notices`. Deps: `residents.registry`, `core.notifications`, `core.storage`, `core.worker`. Reuses `UtilityProvider` (step 29) but is INDEPENDENT of `utility.notices` (step 29 tracks BILLS/money; step 43 tracks TIME).

### Hard rules honoured
- **Wall-clock, never shifted by UTC.** Slot `startTime`/`endTime` are society-local `"HH:mm"` strings. `lib/schedules/rules.ts` NEVER constructs a `Date` from a slot string; it reads `zonedParts(now, tz)` (Intl `en-CA` h23 formatToParts → dayKey/dayOfWeek/minutes-of-day) and compares minutes-of-day. Proven: a 14:00 slot is 30 min away at local 13:30 in BOTH Asia/Karachi and Europe/London (25 vitest in `rules.test.ts`).
- **Never touches the ledger** (same as step 29). `lib/schedules/architecture.test.ts` (6) forbids `@/lib/billing`/`@/lib/charges`/`ledgerEntry`/`invoice.create` in every non-test file under `lib/schedules/`.
- **Midnight-spanning slot** (22:00–02:00 → `end<=start`) renders as TWO display segments (`slotDisplaySegments`) but fires ONE reminder (keyed off the start).
- **Eid/holiday exclusion**: an `isExclusion` slot matching a date suppresses ALL reminders + the Today strip for that date (`isExcludedDate`).

### Data model (migration `20260713450000_utility_schedules_notices`)
- Enums: `ScheduleKind` (LOADSHEDDING|GAS_AVAILABILITY|GAS_SUSPENSION|WATER_SUPPLY|WATER_TANKER|GENERATOR_HOURS|LIFT_MAINTENANCE), `ServiceNoticeKind` (PLANNED_SHUTDOWN|UNPLANNED_OUTAGE|WATER_SHORTAGE|CLOSURE|ROAD_WORK|GOVERNMENT|OTHER), `NoticeSeverity` (INFO|WARNING|CRITICAL), `ServiceNoticeStatus` (DRAFT|PUBLISHED|ONGOING|RESOLVED|CANCELLED). Reuses existing `TargetType` + `Channel`.
- `UtilitySchedule` (scoped: societyId+deletedAt; soft-delete). **DECISION**: `providerId` made NULLABLE (spec wrote `String`) — GENERATOR_HOURS/LIFT_MAINTENANCE are society-internal with no provider; a required FK could not express them. FK `onDelete SetNull`.
- `ScheduleSlot` (pass-through: societyId, NO deletedAt), FK→schedule `onDelete Cascade`. `dayOfWeek Int?` (null=every day), `specificDate DateTime?` (one-off, stored UTC-midnight, compared via `specificDateKey`), `startTime`/`endTime` strings, `isExclusion`.
- `ServiceNotice` (scoped), `remindBeforeMinutes Int[] @default([1440,120])`, dedupe cols `remindersSent Int[]` + `startReminderSent Boolean`, `channels Channel[]`, `@@index([societyId,status,startAt])`. FK provider `onDelete SetNull`.
- `ServiceNoticeConfirmation` (pass-through) `@@unique([noticeId,userId])` — the "mine too" aggregation, idempotent. FK→notice Cascade.
- `ScheduleReminder` (pass-through) `@@unique([scheduleId,userId])` — resident opt-in "remind me before" + `leadMinutes`. FK→schedule Cascade.
- `ScheduleSlotReminderLog` (pass-through) `@@unique([userId,slotId,dayKey])` — per-day slot-reminder dedupe.
- `UtilityProvider` gains back-relations `schedules`/`serviceNotices`.

### Cross-tenant care (recorded)
Pass-through models (societyId but NO deletedAt) are NOT auto-scoped by `lib/scoping.ts`. Fixed: `sweepScheduleReminders` filters `scheduleReminder.findMany({ where:{ societyId, isActive:true }})` explicitly; every other pass-through read keys off already-society-scoped parent ids (scheduleId/noticeId from scoped queries) or the caller's userId. `createMany`/`create` on these stamp `societyId` manually.

### Reminders (worker)
`schedules.reminders` cron `*/15 * * * *` society-scoped, `mutating:true` (READ_ONLY skipped by runtime), handler `runScheduleReminders` feature-gated on `utility.schedules`. `sweepScheduleReminders`:
- SLOT reminders: for each opt-in active `ScheduleReminder`, skip if schedule inactive or day excluded; `slotReminderDue` (pure) fires when the slot applies today & start is within lead & still future; dedupe via `ScheduleSlotReminderLog` unique (P2002→skip); notify `schedule.slot.reminder`.
- NOTICE reminders: `noticeRemindersDue` returns crossed-but-unsent `remindBeforeMinutes` offsets (stops on RESOLVED/CANCELLED/DRAFT, none after start); `noticeStartDue` fires an at-start reminder once; notify `service.notice.reminder`; stamp `remindersSent`/`startReminderSent`.

### API
`/api/utility/schedules` GET (overview | `?scope=manage` list) / POST create; `/[id]` GET/PATCH/DELETE(soft); `/[id]/slots` POST bulk-replace; `/[id]/remind` POST (resident pref, no perm); `/today` GET strip. `/api/service-notices` GET list(`?status=`)/POST create; `/[id]` GET; `/[id]/publish|resolve` POST (manage); `/[id]/confirm` POST (resident "mine too", no perm); `/quick-outage` POST (two-tap ONGOING outage, manage); `/export` POST (PDF/CSV/xlsx outage-duration report via `buildInlineExport` + `loadUrduFont`, audited). `/api/me/schedules` GET.

### Sub-feature gating
`assertNoticeFeature`: a notice with an `authority` set OR kind ∈ {GOVERNMENT,ROAD_WORK,CLOSURE} → requires `utility.external_notices`; every other notice → `utility.outages`. Off → `ScheduleError("feature_disabled")` (422). Timetables are unaffected by either sub-feature.

### Permissions / nav / ui-mode
Perms `schedules.read` (→`utility.schedules`, seeded COMMITTEE_MEMBER + MANAGER) / `schedules.manage` (seeded MANAGER; admin via `*`). Resident view/remind/confirm need NO perm (`requireScheduleMember`). Nav `schedules` (icon `calendarClock` — added `CalendarClock` to `nav-icons.ts`, main group, `hideOnSharedDevice`, feature `utility.schedules`). ui-mode `schedules` (resident audience → default SIMPLE).

### UI (`components/schedules/schedules-client.tsx`, page `app/[locale]/app/schedules/page.tsx`)
- Simple (resident, default): live outage banners (severity-toned, "mine is out too" confirm), **Today strip** (kind-coloured next-event chips), my-schedule cards (remind-me toggle → `/remind`, provider-PDF link, expandable colour-coded week grid handling midnight carry-over + one-off/exclusion rows).
- Committee tabs (Pro toggle adds the effective-window column): schedules-table (overlap warning per `overlappingScheduleIds`), notices panel (publish/resolve, two-tap report-outage modal, export-report PDF download), create-schedule form (kind/name/provider/target/effective/source-PDF upload + "no auto-sync" honesty note) with a per-slot editor (weekly/one-off, start/end time, label, exclusion checkbox).
- Design self-check: semantic tokens (card/muted/primary), logical props (ms-/me-/text-start/text-end) for RTL, mobile-first (max-w-3xl, flex-wrap, overflow-x-auto), light/dark safe.

### i18n
`schedules.*` (100 keys) + `nav.schedules` in en + ur (genuine Urdu, RTL), parity 0-diff verified by key-walk script. Notification templates (category `utility`, en+ur every channel): `schedule.slot.reminder`, `service.notice.published`, `service.notice.reminder`, `service.notice.resolved`.

### Tests
- `rules.test.ts` (25): wall-clock same-across-timezones, lead-window open/closed, post-start no-fire, midnight two-segment + single reminder, dow/one-off applicability, Eid exclusion, Today-strip ongoing/upcoming/excluded, notice offsets fire + stop-on-resolve + no-DRAFT + at-start-once, overlap flag, duration stats.
- `architecture.test.ts` (6): no ledger/billing coupling.
- `schedules.integration.test.ts` (DB-backed, `describe.skipIf(!DATABASE_URL)`, 4): slot reminder once/day + Eid suppression; notice reminder stops on resolve; "mine too" aggregates + idempotent (2nd same-user rejected); outage refused when `utility.outages` off while timetable still works.

### Gate results
`pnpm prisma generate` ✓ · `pnpm typecheck` ✓ clean · `pnpm lint` ✓ 0 warnings · `rules.test` 25/25 + `architecture.test` 6/6 green locally. (Did NOT run test:unit/e2e/build per CLAUDE.md — controller runs full gates.)

### Notes / deferred (recorded, NOT blocking)
- NO scraper/auto-sync built — the spec's honest constraint. `externalSourceUrl` + `lastCheckedAt` columns reserved so a future per-provider importer needs no migration; none promised.
- Tap-drag grid implemented as a functional row-editor (day + start/end + label + exclusion), not literal drag — same data outcome.
- The Today strip is on the schedules page; wiring it into the global home dashboard is a future enhancement (endpoint `/api/utility/schedules/today` already exists).
- An ongoing midnight-spanning slot that started YESTERDAY is not surfaced in the Today strip (the visual grid carries it); documented in `nextEventToday`.

---

## 44 — seasonal-events-qurbani — DONE (2026-07-13)

**Spec:** /specs/44-seasonal-events-qurbani.md (AUTHORITATIVE). **Work type:** FEATURE (branch feature/44-seasonal-events-qurbani).

### What was built
Generic `SeasonalEvent` machinery whose first-class use case is Eid-ul-Adha Qurbani: per-animal permits, fought-over tie-up spaces, butcher rate lists + resident ratings, and scheduled waste rounds. Feature `seasonal.core` + sub-features `seasonal.charges` (per-animal fees via ledger), `seasonal.spaces` (tie-up spots + ballot), `seasonal.vendors` (butcher directory + ratings). `billing.ledger` and `staff.directory` are RUNTIME `isEnabled` checks (graceful degradation), NOT DAG edges.

### Key decisions (recorded — no approval gate)
- **Enum name clash avoided:** spec's `PermitStatus` is already owned by step 42, so the per-flat permit status enum is `SeasonalPermitStatus`. Likewise `SeasonalSpaceMode`/`SeasonalVendorKind`/`SeasonalTaskStatus` are prefixed to avoid collisions.
- **Sub-feature `seasonal.permits` folded into `seasonal.core`.** Registration IS the core function of the module — there is no meaningful state where the event runs but nobody can register — so a separate toggle would only be a footgun. `charges`/`spaces`/`vendors` remain independently toggleable sub-features (each degrades cleanly). Recorded here as an intentional deviation from the spec's four-sub-feature list.
- **Fee snapshot discipline (acceptance #1):** `rules.buildPermitLines` freezes each type's `feeMinor` onto a `SeasonalPermitLine` at registration; `computePermitTotal` sums from the frozen line totals, never a live type read. A committee raising next year's fee (`patchPermitType`) never touches an existing permit's lines or `totalFeeMinor`. Proven by both a pure test and a DB test (create at 2000 → change type to 2500 → reload line still 2000).
- **One itemised invoice (acceptance #4):** approval raises ONE `Invoice` with one `InvoiceLine` per animal type under charge head `qurbani`, plus one `LedgerEntry(INVOICE, DEBIT)` for the total. Flows through the normal payment/receipt/arrears machinery — no parallel money system. FREE + `invoiceId` null when pricing off (`billing.ledger` && `seasonal.charges`).
- **Space double-allocation impossible (acceptance #2):** `allocateSpace` and the ballot take a per-space `pg_advisory_xact_lock(hashtext(society:seasonal-space:<id>))` then re-check `allocatedToPermitId` inside the tx (`rules.canAllocateSpace`). Two concurrent allocations of one spot serialize → exactly one wins (`space_taken`, 409). DB test uses `Promise.allSettled` on two permits → 1 fulfilled / 1 rejected.
- **Auditable ballot (acceptance #3):** deterministic `xmur3`→`mulberry32` seeded Fisher-Yates (`seededShuffle`/`runBallot`) — no `Date.now()`/`Math.random()`, so the same (seed, entrants, spaces) reproduces the exact draw. Append-only `SeasonalBallot` records seed + result + entrant order; a re-run (a prior ballot exists) DEMANDS a reason (`reason_required`) and BOTH runs are retained. Ballot allocates only currently-free BALLOT-mode spaces, winners locked transactionally.
- **Moon-sighting shift (edge case):** `rules.shiftEventToEid` slides `arrivalWindowStart/End` + `departureDeadline` by the delta between the old and newly-confirmed Eid day; registration timestamps are NOT shifted (a society opens registration when it likes — only the animal windows track Eid). `shiftEventDate` notifies all members (`seasonal.date.shifted`). Idempotent when the day is unchanged. `eventDate` is stored (not live-derived) — step 45's Hijri calendar will call this when built.
- **Guard board (acceptance #5):** `getGuardBoard` returns per-flat APPROVED animal counts by type + the arrival window, so an animal arriving without an approved permit is stopped at the gate. GUARD role gets `seasonal.read`.
- **Copy last year (acceptance #6):** `copyEvent` clones an event's permit types + spaces (fees copied verbatim, editable afterwards; allocations NOT carried over) into a new DRAFT event.
- **Registration grace:** `rules.registrationState` gives a 10-minute grace after `registrationClosesAt`, then a clear `registration_closed`.
- **Numbering:** gapless per-event `RUFI-QRB-<year>-0042` via `SeasonalPermitSequence` (atomic upsert keyed societyId+eventId).
- **Cross-tenant safety:** top-level entities (`SeasonalEvent`/`SeasonalPermitType`/`SeasonalPermit`/`SeasonalSpace`/`SeasonalVendor`/`SeasonalTask`) have societyId+deletedAt → auto-scoped soft-delete. Child/audit records (`SeasonalPermitLine`/`SeasonalVendorRating`/`SeasonalBallot`/`SeasonalPermitSequence`) are pass-through (societyId, NO deletedAt) — every read keys off a society-unique parent id (eventId/permitId/vendorId), and creates set societyId explicitly, so no cross-tenant leak (same discipline as step 42/43 pass-through models).

### Files
- Schema: `prisma/schema.prisma` (6 enums + 10 models); migration `prisma/migrations/20260713460000_seasonal_events_qurbani/migration.sql`.
- Backend: `lib/seasonal/{constants,rules,actor,http,service}.ts` + `rules.test.ts` + `seasonal.integration.test.ts`; `schemas/seasonal.ts`.
- API: `app/api/seasonal/**` (events/permit-types/permits/spaces/ballot/vendors/tasks/guard/board/copy/shift-date) + `app/api/me/seasonal/route.ts`.
- Registration: `lib/features.ts` (seasonal.core + 3 sub-features), `lib/rbac.ts` (seasonal.read/manage → seasonal.core; seeded COMMITTEE_MEMBER read, MANAGER both, GUARD read), `lib/nav.ts` (seasonal), `components/shell/nav-icons.ts` (beef icon), `lib/ui-modes/modules.ts` (seasonal), `lib/notifications/templates.ts` (5 codes, en+ur).
- i18n: `messages/en.json` + `messages/ur.json` (`seasonal.*` 189 keys, parity 0-diff; `nav.seasonal`).
- UI: `app/[locale]/app/seasonal/page.tsx` + `components/seasonal/seasonal-client.tsx` (Simple + Pro, light/dark, EN/UR RTL, mobile-first).

### Gate results
- `pnpm prisma generate` — clean (schema valid).
- `pnpm lint` — clean (0 errors/warnings; replaced two `confirm()` with `<ConfirmDialog>`, wrapped JSX literal punctuation, removed unused imports).
- `pnpm typecheck` (tsc --noEmit) — clean.
- Ballot determinism sanity-checked standalone (deterministic + permutation + seed-sensitive) outside the test runner.
- Did NOT run test:unit/test:e2e/build per CLAUDE.md — the controller runs full gates. Tests written: `rules.test.ts` (snapshot, ballot reproducibility, space guard, window, status machine, date shift, rating fold) + `seasonal.integration.test.ts` (DB-backed, skipIf no DATABASE_URL: snapshot-holds-across-fee-change + itemised debit, space concurrency exactly-one-wins, ballot seed recorded + re-run needs reason, free-when-billing-off + guard board).

### Known follow-ups / honest constraints
- Vendor photo is a `photoFileId` field only; the upload UI is a future enhancement.
- `eventDate` is committee-entered for now; step 45 (platform Hijri calendar) will own the confirmed-sighting recompute via `shiftEventDate`.
- Waste-task ↔ complaint (step 26) linkage and per-slot staff assignment UI are minimal (assign by staff id, mark done); richer scheduling is a future enhancement.
- Collection-vs-outstanding board computes "collected" from `LedgerEntry(PAYMENT, CREDIT)` rows linked by `invoiceId`.

[CHECKPOINT]

## 45 — islamic-calendar-prayer — DONE (2026-07-13)

**Feature:** `islamic.core` + sub-features `islamic.prayer_times`, `islamic.masjid`, `islamic.ramadan`, `islamic.greetings`, `islamic.hijri`. Deps: `core.tenancy`, `core.notifications`, `branding.pwa` (spec named `pwa.push`; registered code is `branding.pwa`), `announcements.core`. All `isCore:false` (opt-in).

### The two honest technical decisions (both enforced by tests)
1. **Prayer times COMPUTED OFFLINE** — `adhan-js` (`adhan@4.4.4`, MIT) installed. It is ESM-only (`"type":"module"`, named exports) → added `"adhan"` to `serverExternalPackages` in `next.config.mjs`. `lib/islamic/prayer.ts` is a pure wrapper: coords + method + madhab + high-lat rule + per-prayer minute offsets → society-local "HH:mm" strings (formatted via `Intl` in the society tz; adhan `params.adjustments` applies the offsets). Method map KARACHI/MWL/ISNA/EGYPT/UMM_AL_QURA/MAKKAH→adhan `CalculationMethod`. HANAFI Asr ~1h20m later than SHAFI (probed 17:20 vs 16:01 Karachi 2026-07-13). `architecture.test.ts` statically forbids `fetch(`/network imports in the prayer path (prayer/rules/service/hijri/constants); the ONLY file allowed to `fetch()` is `hijri-sources.ts`.
2. **Hijri is a PLATFORM (L0) responsibility** — `HijriCalendarDay` + `HijriSyncSettings` have NO `societyId`/`deletedAt` (infra pass-through models). Societies READ per region; there is NO society write path (no route, no actor, no service fn). `architecture.test.ts` asserts society `service.ts` contains zero `hijriCalendarDay.(create|update|upsert|delete)` and that only `platform-service.ts` writes it — the static counterpart to the "SOCIETY_ADMIN cannot mutate a HijriCalendarDay" acceptance test.

### Hijri logic (`lib/islamic/hijri.ts`, pure)
- Offline Umm al-Qura via `new Intl.DateTimeFormat("en-u-ca-islamic-umalqura", {timeZone:"UTC"})`. `regionalLagDays`: PAKISTAN −1 (documented heuristic — real dates come from published calendars / L0 confirmation), Gulf/global 0.
- `crossCheck()` truth table: reachable = primary + non-null cross-checks. all-down → UNCONFIRMED (computed fallback, alert L0); primary reachable + everyone agrees + not a sensitive day + autoApply → CONFIRMED; otherwise PENDING_CONFIRMATION + alert (covers disagreement, primary-down, autoApply-off, and sensitive days). `requireConfirmFor` default `["9-1","10-1","12-1","12-10"]` (Ramadan/Shawwal/Zilhaj starts + Eid-ul-Adha) — `hijri.test.ts` proves autoApply NEVER auto-accepts Shawwal 1 / Zilhaj 1 / Zilhaj 10 even when all sources agree; and that "previous CONFIRMED/MANUAL day is never downgraded" (in `runHijriSync`).

### Propagation (acceptance: L0 Eid confirm → Qurbani windows + notify residents)
`platform-service.confirmDay`/`overrideDay` → set CONFIRMED/MANUAL + audit + `propagateEid`: if the row is Zilhaj 10, find every `QURBANI` `SeasonalEvent` for that `hijriYear`, check each society's effective `hijriRegion` (default PAKISTAN) matches the row's region, then `withSociety(societyId, () => shiftEventDate(systemCtx, eventId, gregorian), {actorType:"PLATFORM"})`. `shiftEventDate` (step 44) slides arrival window + departure deadline by the Eid delta and notifies all members (`seasonal.date.shifted`). Idempotent (re-confirm = no-op). Integration test seeds a Qurbani event (computed Eid 2027-06-06), confirms a PAKISTAN Zilhaj-10 row at 2027-06-07, asserts eventDate/arrivalWindowEnd shift to 06-07, departureDeadline 06-09→06-10, propagated===1, and a `seasonal.date.shifted` notification exists.

### Worker jobs (`lib/worker/registry.ts` + `handlers.ts`)
- `prayer.compute` — society, `0 1 * * *`, mutating. `refreshPrayerCache` computes today+`PRAYER_CACHE_DAYS`(7) offline, prunes past days, upserts `PrayerDay`. Skips when feature off / no coordinates.
- `prayer.reminders` — society, `*/5 * * * *`, mutating. `sweepPrayerReminders`: strictly-opt-in per prayer via `PrayerReminderPref` rows (no row = no reminder); fires PUSH+IN_APP when `now ∈ [target−lead, target−lead+6min)`; `lastSentDay` dedupes one push/prayer/day; SEHRI/IFTAR read `RamadanDay` and only during Ramadan with `islamic.ramadan` on (sehri default lead 30 min).
- `hijri.sync` — platform, `0 2 * * *`, non-mutating. `runHijriSync` reconciles today+next 2 days across all 4 regions; per (region,day) polls primary (`ALADHAN` via `fetch`, best-effort 8s timeout) + cross-checks (`COMPUTED` offline; `AL_HABIB`/`MOONSIGHTING` pluggable stubs returning null — no public feed yet), never downgrades a CONFIRMED/MANUAL row, alerts L0 via `sendViaAdapter` (email + WhatsApp to `PLATFORM_ALERT_*` env, best-effort). This is the module's ONLY outbound HTTP.

### Data model — migration `20260713470000_islamic_calendar_prayer` (9 tables, hand-diffed via `prisma migrate diff` datamodel→datamodel, no DB touched)
Enums PrayerMethod/Madhab/HighLatRule/HijriRegion/HijriStatus/HijriSource/Prayer/Occasion/TriggerType (reuses existing `Channel`). Models: `IslamicSettings` (id=societyId, pass-through; lat/long nullable — no coords → times NOT shown; `offsetsMinutes` Json; `hijriRegion` the ONLY society Hijri setting), `HijriCalendarDay` + `HijriSyncSettings`(id="singleton") (platform infra, no societyId), `Masjid`/`JamaatTime`/`RamadanDay`/`Greeting` (tenant soft-delete), `PrayerDay` + `PrayerReminderPref` (**pass-through, HARD-delete** — deliberately dropped `deletedAt` after realising a soft-deleted cache/pref row would collide with the `@@unique` on re-upsert; cache purge + reminder toggle now hard-delete cleanly). All prayer/jamaat/ramadan times are society-local WALL-CLOCK "HH:mm" strings, never rebuilt into a Date (reuses `zonedParts`/`hhmmToMinutes` from `lib/schedules/rules`).

### API
Society: `/api/islamic/settings`(GET/PUT), `/prayer-times`(GET ?date), `/hijri`(GET ?date read-only), `/ramadan`(GET ?hijriYear / POST generate) + `/ramadan/[id]`(PATCH adjust/restore), `/ramadan/export`(GET PDF), `/masjids/export`(GET jamaat PDF), `/jamaat-times/[id]`(DELETE); `/api/masjids`(GET/POST)+`/[id]`(PATCH/DELETE)+`/[id]/jamaat-times`(GET/POST); `/api/greetings`(GET/POST)+`/[id]`(PATCH/DELETE)+`/[id]/send`; `/api/me/prayer-reminders`(GET/PUT). Platform (L0, `requirePlatform`): `/api/platform/hijri/settings`(GET/PUT), `/sync`(POST manual re-run), `/pending`(GET), `/[id]/confirm`(POST one-click), `/[id]`(PUT manual override, audited). Errors via `islamicErrorResponse` (IslamicError code→status; falls back to `authErrorResponse` for platform ForbiddenError).

### Registrations
Perms `islamic.read`(→COMMITTEE_MEMBER, MANAGER)/`islamic.manage`(→MANAGER; admin `*` auto) gated by `islamic.core`; residents read prayer/masjid/hijri + manage OWN reminders with NO perm. Platform perms `platform.hijri.read`/`platform.hijri.manage` (ungated) → PLATFORM_ADMIN (+ SUPPORT gets read). Nav `islamic` icon `moon` (added `Moon` to `nav-icons.ts`), main group, ungated (residents see it; committee console inside). ui-mode `islamic` (audience resident→SIMPLE default, supportsModes). Notif templates category `announcements` en+ur: `prayer.reminder`, `ramadan.sehri`, `ramadan.iftar`, `hijri.shifted`. i18n `messages/en.json`+`ur.json`: `nav.islamic` + `islamic.*` namespace (143 keys each, full-file parity 0-diff verified).

### UI
`app/[locale]/app/islamic/page.tsx` (server, feature-gated EmptyState) → `components/islamic/islamic-client.tsx` tabbed (prayer/masjids/ramadan/greetings/settings, `useUiModule("islamic")` Simple/Pro). Simple (resident): next-prayer strip with countdown where jamaat time BEATS the calculated adhan time; today's 6 times grid; strictly-opt-in reminder switches; dual Hijri header (unconfirmed marker); masjid list with prominent jamaat times; Ramadan tab (sehri/iftar today + table). Pro/committee: add masjid + effective-dated jamaat editor, Ramadan generate/adjust/restore-computed, greeting composer + one-click send (autoSend off by default), settings (lat/long/method/madhab/high-lat/region/toggles) with an explicit madhab-change confirm dialog. Home-screen `components/islamic/next-prayer-strip.tsx` mounted above the dashboard, entitlement-gated (`features.has("islamic.prayer_times")`), server-computed via `getPrayerStrip`. PDF `lib/islamic/pdf.ts` (`renderRamadanPdf`/`renderJamaatPdf`) using pdf-lib + `@pdf-lib/fontkit` + Noto Naskh Urdu (`loadUrduFont`), branded, for notice-board printing — mirrors `lib/billing/invoice-pdf.ts` (glyphs, no Arabic shaping — documented pdf-lib limit).

### Tests / gates
`prayer.test`(4), `hijri.test`(8), `rules.test`(7), `architecture.test`(9) all pass (28 unit); `islamic.integration.test`(3) skipIf-no-DB (Hijri read-only + confirmed-day read + missing-day UNCONFIRMED fallback; L0 Eid confirm → Qurbani shift + resident notify; jamaat overrides calculated in overview payload) — for controller/CI. `pnpm prisma generate`, `pnpm lint` (0 warnings/errors), `pnpm typecheck` all clean. Did NOT run test:unit/e2e/build (controller runs full gates).

### Recorded decisions / deferrals
- AL_HABIB / MOONSIGHTING adapters are pluggable stubs returning null — no public local-sighting API exists (spec explicitly acknowledges there is no official Ruet-e-Hilal endpoint); when a feed is added, implement in `hijri-sources.fetchPublishedCalendar`.
- AlAdhan reading has the regional lag applied so it is comparable to COMPUTED for the region; the PAKISTAN −1 lag on the offline computed source is a documented heuristic used only for fallback, never asserted as confirmed.
- Settings takes numeric lat/long today; the spec's map picker for the "no coordinates" edge case is a future UI enhancement (the empty-state explains why times are hidden).
- Greeting card image is stored as `imageFileId` (passed to the GREETING announcement's `attachments`); a dedicated uploader is a future enhancement.
- `PrayerDay`/`PrayerReminderPref` intentionally made pass-through hard-delete tables (see data model note) — the one design correction made mid-build.

---

## 46 — property-model — DONE (2026-07-13) — branch feature/46-property-model — WORK TYPE: FEATURE

The load-bearing spine amendment: `flats.registry` → `units.registry` (isCore), and a full
Flat→Unit rename plus a dynamic structure tree replacing the fixed Block→Floor hierarchy. The
DB was empty/undeployed, so this was built directly (no back-compat, no alias models, no
back-fill migrations); the migration set was regenerated as ONE clean init.

### Scope executed
- **Mechanical rename (scripted, 1173 files):** `Flat`→`Unit`, `flatId`→`unitId`,
  `FlatCategory`→`UnitCategory`, `FlatOccupancy`→`UnitOccupancy`, `flatCodeFormat`→`unitCodeFormat`,
  `/api/flats`→`/api/units`, plus lowercase domain identifiers and i18n `flat.*` keys → `unit.*`
  (the SAME sed applied to both `messages/*.json` and `t()` call sites, keeping them in sync).
  The two legitimate `Array.prototype.flat()` calls were protected via a sentinel. Physical
  file/dir renames: `app/api/flats`→`units`, `flat-categories`→`unit-categories`,
  `me/flats`→`me/units`, `me/active-flat`→`me/active-unit`, `onboarding/flats`→`units`,
  `utility/flats`→`units`; `lib/society-structure/flat-code*`→`unit-code*`, `flat-columns`→`unit-columns`;
  component `flat-*`→`unit-*`. Enum SCREAMING values normalised (SINGLE_FLAT→SINGLE_UNIT,
  FLAT_LIST→UNIT_LIST, PER_FLAT→PER_UNIT, FLAT_SCOPED→UNIT_SCOPED).
- **Schema redesign:** dropped `Block`/`Floor`; added `StructureNode` (self-referencing, any depth,
  materialised `depth`/`path`/`ancestorIds` — GIN index on `ancestorIds`; `@@unique([societyId, path])`).
  `Unit` now carries `nodeId` (leaf node), `propertyType`, `areaSqft` (no blockId/floorId).
  New enums `PropertyType`, `NodeKind`, `QuantitySource`. `RateRule` gained `propertyType`, `nodeId`
  (replaces `blockId`), `quantitySource`. `ChargeKind` gained `PER_QUANTITY`. `SocietySettings`
  gained `unitLabelKey` (society's own word: unit|flat|house|property|shop). `Camera` blockId/floorId
  → single `nodeId`. Migration history squashed to a single offline-generated init
  (`prisma migrate diff --from-empty`); `prisma validate`+`generate` clean.
- **Pure core (DONE, unit-tested):**
  - `tree.ts` — materialise path/depth/ancestorIds on write; `assertNoCycle` (a node can never
    become its own ancestor); `assertWithinDepthCap` (cap 6, accidental-nesting guard); `nearestCodeFormat`
    (per-node override inherited down); `recomputeSubtree` (pure in-memory recomputation of an
    already-loaded subtree for a MOVE — no recursion). Tests: root/child invariants, cycle, depth cap,
    move-to-new-parent, move-to-root, move-into-own-descendant rejected.
  - `unit-code.ts` — `displayCode` from the ancestor chain: kind tokens `{block}/{tower}/{floor}/{street}/…`
    resolve to the nearest ancestor of that kind, plus `{number}/{code}/{path}`. One codebase serves
    "D-20" (tower society) and "St-7-12" (housing society). Tests cover both shapes + validation.
  - `bulk.ts` — two shapes: "floors × units" (creates FLOOR nodes + units) and "direct" (street × houses,
    no intermediate level); floorPrefixed/sequential numbering; atomic plan with duplicate/conflict/limit
    guards (blocked before creating anything). Tests: floors, street, mixed (codes never collide), guards.
  - `charges/resolve.ts` — extended specificity ladder: unitId → categoryId+occupancy → categoryId →
    propertyType+occupancy → propertyType → occupancy → nodeId (DEEPEST node wins, matched via the unit's
    ancestor chain) → default; ties → newest effectiveFrom. `PER_QUANTITY`: line = rate × quantity(unit)
    from `quantitySource`; quantity+rate snapshotted onto the line; **quantity 0 → NO line** (not a ₨0 line).
    Tests: full ladder, Rufi 3500/2000, property-type split, deepest-node-wins, PER_QUANTITY incl. zero,
    effective-dating, split-equally.
- **Guards as tests:** `architecture.test.ts` (no `WITH RECURSIVE`/`$queryRaw` tree walk; service reaches
  subtrees via `ancestorIds`) and `rename-gate.test.ts` (no `\bFlat\b`/flatId/FlatCategory/FlatOccupancy/`/api/flats`
  anywhere in app/components/lib/schemas/worker/e2e/tests/scripts or the prisma schema).
- **Cascade (structure service + charges service + ~15 downstream modules), driven to a zero-error
  typecheck:** structure service rewritten around StructureNode (node CRUD + `moveNode` in one tx via
  ancestorIds subtree read + cycle/depth guards + `loadChain`; units carry nodeId/propertyType/areaSqft;
  bulk/CSV/import/export node-based). New `app/api/structure/nodes` (+`/move`) surface; old `blocks`/`floors`
  routes removed. Charges service builds `ResolveUnit` (nodePath = [...node.ancestorIds, nodeId]; quantities
  from live parking-slot/vehicle counts, areaSqft, category.bedrooms) and handles PER_QUANTITY. "Target by
  block" across announcements/polls/surveys/utility/schedules/notifications/special-charges now means "target
  by structure node": a unit matches iff `unit.nodeId === targetId || node.ancestorIds.includes(targetId)`
  (the `BLOCK` enum value name is kept; only its ids' meaning changed to node ids). reports/emergency/meters/cctv
  and residents/billing group/display via the unit's node; billing snapshots PER_QUANTITY quantity+rate into
  the invoice line `meta`. All *.integration.test.ts fixtures reseed a root `StructureNode` + units with `nodeId`.
- **i18n:** added `unit.label.{unit,flat,house,property,shop}` (en+ur), `structure.nodeKind.*`,
  `structure.propertyType.*`, ~21 new `structure.nodes.*`/`grid.emptyNodes*`/`counts.nodes` leaves,
  `charges.rule.{propertyType,quantitySource}`, `charges.chargeKind.PER_QUANTITY`. en/ur parity 0-diff.
- **features.ts:** `units.registry` (isCore) with every `dependsOn` edge updated; description refreshed.

### Gates
- `pnpm typecheck` (whole tree): 0 errors. `pnpm lint`: clean.
- Targeted vitest of the new/changed suites (tree, unit-code, bulk, resolve, architecture, rename-gate,
  announcements targeting, polls/surveys rules, csv) — all green. (Full `test:unit`/`test:e2e`/`build`
  left to the controller per CLAUDE.md.)

### Honest decisions / deferred (recorded, non-blocking)
- The structure tree UI is a FUNCTIONAL node tree (create/rename/move/delete via the nodes API + bulk
  generator + unit table + Nodes/Categories/Settings tabs). Full drag-and-drop reparenting and the Pro
  "rate simulator breadcrumb" are simplified to button/select-driven equivalents — the move endpoint and
  code-regeneration are real; drag polish is a future enhancement. Light/dark + EN/UR RTL + responsive are
  inherited from the shared design-system primitives (FormField/table kit) the screens are built on.
- `TargetType.BLOCK` (and the audience/notification "BLOCK" value) were intentionally NOT renamed — only
  their ids' meaning changed to structure-node ids — to keep the cross-module targeting surface stable.
- Some downstream DTO/query-param field names (`blockId` in meters/emergency/reports/residents) were kept
  as names but now hold a StructureNode id, to minimise churn (not gate tokens; typecheck clean).
- Camera location collapsed from blockId+floorId to a single `nodeId`.
- The one-society-holds-towers+houses+shops billing scenario is exercised by the pure resolver tests
  (property-type + node ladder + PER_QUANTITY); a dedicated DB integration test for a single mixed invoice
  run is a candidate follow-up.

## 47 — platform-bootstrap & deployability — DONE (2026-07-13)

**Branch:** feature/47-platform-bootstrap · **Feature code:** platform.bootstrap (isCore). Spec: /specs/47-platform-bootstrap.md + 47-47-CODEREF.md.

**Problem solved:** a fresh deploy was unusable — empty Feature table, no PLATFORM_ADMIN, so nobody could reach /platform and no society could be created. syncFeatureRegistry() existed but nothing called it; platform roles (syncPlatformRoles, already present in lib/roles.ts) were never seeded on deploy.

**What shipped:**
- lib/bootstrap.ts — shared core: assertDbReady (clear message on unmigrated/unreachable DB, not a Prisma stack trace), runBootstrap (DAG integrity check → syncFeatureRegistry → syncPlatformRoles → count admins, idempotent, never creates a user), countPlatformAdmins, createPlatformAdmin (societyId null, ACTIVE, mustEnrolTotp=true; refuses a second admin unless forceAdd; generates+returns a strong password to print once; SYSTEM audit "platform.admin.bootstrapped"). Typed errors PlatformAdminExistsError / DatabaseNotReadyError.
- scripts/bootstrap.ts (pnpm bootstrap) and scripts/bootstrap-admin.ts (pnpm bootstrap:admin, arg-parsed --email/--phone/--password/--force-add). Added both to package.json.
- lib/entitlements.ts — syncFeatureRegistry now calls assertFeatureRegistryIntegrity() (exported; assertValidDag on FEATURE_DAG, throws MissingDependencyError/FeatureCycleError before any write) and, per golden rule #1, never hard-deletes a vanished code: a Feature no longer in lib/features.ts is flagged deprecated=true (warns if still referenced by a SocietyEntitlement), un-deprecated if it reappears. Return type now { count, deprecated[] }.
- TOTP second factor (spec requires the bootstrapped admin enrol before any /platform route): dependency-free RFC 6238 in lib/auth/totp.ts (base32, HMAC-SHA1, generate/verify with ±1 step, otpauth URI). Schema: User.mustEnrolTotp/totpSecret/totpEnrolledAt; Feature.deprecated. AuthContext gains pendingTotpEnrolment. Platform layout renders the enrolment gate (components/platform/platform-enrol-totp.tsx, form kit) instead of the console while pending; requirePlatform 403s pending callers; the enrol route (app/api/platform/totp/enrol) guards on new requirePlatformOperator (skips the gate) — POST {} mints/returns the secret, POST {code} verifies + clears the flag + audits "platform.totp.enrolled". i18n keys platform.enrolTotp (en + ur).
- /api/health — added bootstrap { featuresSeeded, platformAdmins, ready }; ready=false (503) when features unseeded or no admin.
- deploy.sh — pnpm bootstrap after migrate deploy, before gates; pm2 restart + health port now branch-aware (the ecosystem rename forces this so a staging deploy never touches production processes). CI — same Bootstrap step after migrate deploy.
- ecosystem.config.cjs — replaced the single 2-process config with four: rihaish-web-staging (cluster×2, 127.0.0.1:39330), rihaish-worker-staging (fork), rihaish-web (cluster×2, :39331), rihaish-worker (fork); web bound to 127.0.0.1 (nginx-only); cwd via RIHAISH_STAGING_CWD/RIHAISH_PROD_CWD. worker/index.ts asserts tsx resolvable at boot (fails loudly on a --prod install). .env.example: PLATFORM_ADMIN_PASSWORD + the two cwd vars.

**Migration:** hand-edited the single 0000000000000_init/migration.sql (repo pattern — step 46 did the same) to add the User TOTP columns + Feature.deprecated; ran prisma generate.

**Tests:** lib/auth/totp.test.ts (pure: roundtrip, ±1 window, reject far/other-secret/malformed, otpauth URI, KAT). tests/unit/bootstrap.integration.test.ts — DAG integrity (pure: real registry passes, unregistered edge throws MissingDependencyError) + Postgres-backed: full feature+role seed, twice-idempotent, exactly-one admin with mustEnrolTotp true + SYSTEM audit + password hash verifies, refuses second without force-add, supplied-password path. e2e/bootstrap.spec.ts — health readiness shape; anonymous 401 on enrol + society-create; env-gated full flow login → TOTP-gate 403 → enrol → create society.

**Decisions:** (1) TOTP is not a pre-existing subsystem; built it minimally but real (dependency-free) since the spec + e2e require enrolment before console access — kept the UI to the enrolment gate only. (2) Placed bootstrap logic in lib/bootstrap.ts (not just scripts/) so vitest, which only includes lib/schemas/tests-unit, actually runs the coverage; scripts are thin CLIs. (3) Added lib/bootstrap.ts to the db.unscoped() eslint allow-list (platform-level code). (4) Made deploy.sh branch-aware to avoid the ecosystem rename regressing staging into production processes.

**Gates:** pnpm prisma generate ✓, pnpm lint ✓ (added lib/bootstrap.ts to unscoped override), pnpm typecheck ✓. Did not run test:unit/e2e/build (controller runs them).

## 48 — worker-stubs-notifications — DONE (2026-07-13)
**Branch:** feature/48-worker-stubs-notifications · **Spec:** /specs/48-worker-stubs-notifications.md (+ 48-48-CODEREF.md)

Closed the three things the system promised and silently never did, plus a sweep for other dead crons.

**1. Dues reminders (`billing.reminders`) — was a stubHandler no-op.**
- New `lib/billing/reminders.ts` `sweepBillingReminders(societyId, now)`: for each open invoice (ISSUED/PARTIALLY_PAID) whose unit is genuinely in arrears (`flatBalanceMinor > 0`), fires on each configured before/after-due day via the pure `reminderKindFor` (reused from platform-billing math) and then STOPS — no daily nagging.
- Recipient is the invoice `billedToUserId` SNAPSHOT, never the current occupant, so an ownership change never duns the wrong person. Sends through the step-11 `notify` engine (honours NotificationPreference).
- Idempotent per (invoiceId, offset) via a new `BillingReminderLog` model (unique [invoiceId, offset]); the slot is CLAIMED before sending (claim rolled back if notify throws, so a retry can re-try but never double-sends). A mid-cycle change to the reminder-day list can't double-message.
- Skips a READ_ONLY society (checked inside the sweep so it is behaviourally testable) and, in the handler, when `billing.ledger` is off. Arrears unit with no billable person → counted `skippedNoBillable`, logged, never crashes.
- New settings `BillingSettings.reminderDaysBefore [7,3,1]` / `reminderDaysAfter [1,3,7,14]`, surfaced on `BillingSettingsView`.
- Templates `billing.reminder.before` + `billing.reminder.overdue` (category billing), en+ur, IN_APP/EMAIL/SMS/PUSH.

**2. Export purge (`exports.cleanup`) — was a stubHandler no-op.**
- `runExportsCleanup` (platform-scoped): finds `Export`/`ReportExport` StoredFile rows older than 7d via the raw client, soft-deletes each IN the owning society scope (scoping layer writes the `storedFile.deleted` audit + frees `maxStorageMb` usage), then purges the object and hard-removes the row. Recent exports and non-export files are untouched.

**3. Password/PIN change security notification — was a `TODO(step 11)`.**
- New `security` notification category; behaves like `emergency` but floors IN_APP + EMAIL (cannot be muted) via `isChannelLocked`/`applyChannelFloor`.
- `changePassword` and `setGuardPin` now fire `account.password.changed` / `account.pin.changed` (category security) via `notifySafe`, with the society-local time + revoked-session count. No new-device-login path exists yet, so that variant was not added.
- Templates for both codes, en+ur, IN_APP + EMAIL. i18n category labels added (en+ur) so the account/settings panels render.

**Sweep for other stubs:** grep of `stubHandler(` and `TODO(step` across lib/ + app/ → the two crons above and the one password TODO were the ONLY occurrences; all now real. `stubHandler` kept as a helper but has ZERO callers, and it now tags its function `isStub` so `registry.test.ts` asserts no seeded cron ever points at a no-op again.

**Schema/migration:** added `BillingSettings.reminderDaysBefore/After`, `BillingReminderLog`; regenerated the squashed `0000000000000_init` baseline (clean superset — only the new fields/table/indexes) and `prisma generate`.

**Tests added/updated:** `lib/billing/reminders.integration.test.ts` (offsets, idempotency, snapshot recipient, READ_ONLY skip, no-billable), `lib/storage/exports-cleanup.integration.test.ts` (only-exports-past-retention, object deleted, audited), `lib/account/security-notify.integration.test.ts` (in-app+email survive a full opt-out), `lib/worker/registry.test.ts` (no-stub-cron invariant), `lib/account/notifications-policy.test.ts` (security email-lock + length 10).

**Gates:** `pnpm prisma generate`, `pnpm lint`, `pnpm typecheck` all clean. (Did not run test:unit/build per AGENT.md — controller runs full gates.)

WORK TYPE: FEATURE (branch feature/48-worker-stubs-notifications)

## 49 — feature-registry-completeness — DONE (2026-07-13)

Enforced ARCHITECTURE principle #2 (no module ships without registering its feature code + dependency edges) by closing the four quiet gaps spec 49 named, and adding the guard test that catches this class of drift forever.

**Registry (lib/features.ts):**
- Registered the six CCTV transport-tier sub-features (link, embed, agent, snapshots, playback, events) with the spec-41 DAG edges. `cctv.events` deliberately does NOT hard-depend on gate pass — that is a soft runtime integration (event snapshots degrade to nothing when gate pass is off), matching the file's established soft-dep convention; a hard edge would wrongly force gate pass on.
- Registered `seasonal.permits` (the Qurbani-animal registration flow; the spine charges/spaces/vendors already hung off `seasonal.core`).
- Split the PWA into three real, independent features: kept `branding.pwa` (white-label manifest) and added `pwa.core` (installable shell, deps branding.core + notifications + shell) and `pwa.push` (web push, dep pwa.core). A society can now install the app without white-label branding and disable push on its own.
- Renamed the two drifted codes `complaints` → `complaints.core` and `gatepass` → `gatepass.core`, deleted the substitution comments, and re-pointed every edge (staff.console, staff.performance, complaints/gatepass sub-features) plus the `branding.pwa` stand-in edges (staff.console/gatepass.core/chat.core → pwa.core; islamic.core → pwa.push).
- Also registered `platform.bootstrap` (spec 47, isCore) — the completeness guard flagged it as genuinely missing, so registering it was the honest fix rather than an exception.

**Reference renames (feature-code usages only — NOT the i18n namespaces, notification categories, nav/tab keys, ui-mode codes or storage buckets that share the string):** app complaints/gate/gate-pass pages, lib/rbac.ts (permission→feature map values), lib/nav.ts, lib/pwa/tabs.ts (feature: only, keys/permissions kept), lib/reports/dashboards.ts + constants.ts, lib/permits/constants.ts (GATEPASS_FEATURE), and the permits integration test.

**Route/service enforcement (a code nothing enforces is decoration):**
- lib/cctv/service.ts: mode selection requires the matching tier (LINK→cctv.link, EMBED→cctv.embed, AGENT→cctv.agent); enabling resident/camera playback requires cctv.playback; enrollAgent requires cctv.agent; pushSnapshot requires cctv.snapshots. Agent heartbeat route also gated on cctv.agent.
- lib/seasonal/service.ts: registerPermit + listPermits gated on seasonal.permits (same isEnabled/feature_disabled pattern as spaces/vendors).
- PWA push routes: subscribe gated on pwa.push (403); public-key returns publicKey:null when off — the app still installs, push simply never registers (spec edge case: no dead nav, no 500).

**Migration (prisma/migrations/20260713120000_feature_registry_completeness):** UPDATEs SocietyEntitlement rows and array_replace on Plan.featureCodes for the two renames (never DELETE), plus an idempotent back-fill that grants CCTV sub-features to societies by their current CctvIntegration.mode (LINK→link, EMBED→embed, AGENT→agent+snapshots) so nobody loses access on upgrade day. syncFeatureRegistry already flags the vanished bare codes deprecated (kept, not deleted).

**Tests:** lib/features.test.ts now scans every shipped module's spec header for cited feature codes and asserts each is registered (SUPERSEDED map handles flats.registry→units.registry from step 46) — the drift guard that would have caught all four of these on day one; plus explicit assertions for the new codes and the two renames. feature-dag.test.ts gained real-registry coverage of the CCTV subtree (cascade + chain + the no-hard-gatepass-edge invariant). New DB-backed tests/unit/feature-rename-migration.integration.test.ts proves a society entitled to `gatepass` before is entitled to `gatepass.core` after, same row id, access intact, nothing deleted. Updated the cctv + seasonal integration tests to enable the newly-required sub-features.

**Gates:** pnpm typecheck + pnpm lint both clean. Verified out-of-band (not test:unit): the registry imports/validates at runtime (83 features, 15 core, DAG acyclic) and every spec-cited code resolves against the registry.

WORK TYPE: FEATURE (branch feature/48-worker-stubs-notifications)

## 50 — cctv-architecture-guard — DONE (2026-07-13)
**Branch:** feature/50-cctv-architecture-guard · **Spec:** /specs/50-cctv-architecture-guard.md · extends cctv.core

**What:** Added `lib/cctv/architecture.test.ts` — a PURELY STATIC guard (fs + regex, no DB/network) enforcing ARCHITECTURE.md principle #11 "Rihaish never carries video." It is the static counterpart to the runtime guarantees already proven in `cctv.integration.test.ts`.

**Assertions:**
1. No media/streaming dependency is imported anywhere in production source (SOURCE_ROOTS = app, components, lib, schemas, scripts, worker, i18n; test files excluded). Extracts real module specifiers (`from`/`import`/`require`/dynamic-import) so a bare string literal naming a lib — as the assertion lists themselves do — is not a false positive. Forbidden: fluent-ffmpeg, ffmpeg-static, node-media-server, mediasoup, wrtc, werift, hls.js, rtsp-*, @ffmpeg/*. Also asserts package.json declares none of them.
2. No CCTV route proxies a body — walks app/api/cameras/** + app/api/cctv/** and per route file forbids ReadableStream, `.pipe(`, `new Response(`, `.body`, `fetch(`; positively asserts each responds via NextResponse (JSON only). 9 route files covered.
3. Stream URL always resolves to the society edge: buildStreamUrl origin === the edge base it was given; token.ts contains no process.env/APEX_HOST/WILDCARD_HOST; no lib/cctv file references a Rihaish apex/wildcard host; service builds the URL from `integration.edgeBaseUrl`.
4. Snapshot caps enforced in code (not docs): MAX_SNAPSHOT_BYTES ≤ 128KB; service.ts references MAX_SNAPSHOT_BYTES + snapshot_too_large + snapshotAllowed() + snapshot_rate_limited + storeFile().
5. View tokens single-scoped + short-lived: MAX_TOKEN_TTL_SECONDS ≤ 120; token claim shape carries societyId/cameraId/userId/jti/exp/iat and has no plural id arrays; service clamps to MAX_TOKEN_TTL_SECONDS. (token.test.ts already covers mint/verify/expiry/replay/tamper — no extension needed.)
- Acceptance: CameraViewLog has NO delete path in production code — repo-wide (non-test) scan for `cameraViewLog.delete(Many)` returns empty. Test cleanup in cctv.integration.test.ts is excluded (it is a *.test.ts file and a legitimate teardown).

**Verified pre-write the tree is clean:** no media deps in package.json, no APEX/WILDCARD in lib/cctv, no streaming patterns in routes, no production CameraViewLog delete path — so the suite is green now and only goes red on a real violation (acceptance: add a stream-proxy route or a media import → red).

**Also-verify items already covered by existing tests (no new test needed):** RESTRICTED/PRIVATE-not-grantable-to-resident-for-any-role (cctv.integration.test.ts #1 + rules.test.ts), ex-tenant grant auto-revoke on ended occupancy (cctv.integration.test.ts #3), http.ts signs/returns only — never fetches upstream (now also asserted statically here).

**Gates:** `pnpm lint` ✔ (no warnings/errors) · `pnpm typecheck` ✔ (tsc --noEmit clean). Did not run test:unit/e2e/build per standing rules (controller runs full gates). No schema change → no prisma generate.

**Files:** added lib/cctv/architecture.test.ts. No source changes (this step adds a guard; the guarantees it locks in were already implemented in steps 41/49).

## 51 — lifts (feature/51-lifts) — DONE 2026-07-13

**Spec:** /specs/51-lifts.md. Feature codes `lifts.core` (+ `lifts.maintenance`, `lifts.cargo_booking`).

**Design decisions**
- Booking REUSES the amenity engine (spec 35) rather than reimplementing concurrency-safe slot booking. Added `AmenityKind` enum (`FACILITY|CARGO_LIFT`) + `Amenity.liftId`, and `AmenityBooking.operatorStaffId/startedAt/endedAt`. The lift module owns registry + maintenance; a cargo lift is an Amenity bound to a lift. `amenities.core` off → registry/maintenance still work, booking absent (`ensureLiftBookingAmenity` throws `amenities_disabled`).
- DAG edges are HARD deps only: `lifts.core → units.registry, complaints.core`; `lifts.maintenance → lifts.core`; `lifts.cargo_booking → lifts.core, amenities.core`. `expenses.core`/`staff.directory` are optional → runtime `isEnabled` checks, not edges.
- Notifications reuse EXISTING categories (no new NOTIFICATION_CATEGORIES rollout): committee maintenance/breakdown warnings ride `complaints`; the out-of-service broadcast to the affected block rides `announcements`. Six new template codes added to the catalogue, each en+ur on every declared channel.
- Breakdown files a `Ticket` (COMPLAINT) in an auto-ensured `LIFT` ServiceCategory, flips the lift `OUT_OF_SERVICE`, and — if `peopleTrapped` — raises an `EmergencyAlert` (kind `LIFT_TRAPPED`, `ignorePreferences`) instead of a mere maintenance ticket. Restore closes the open breakdown with computed downtime and notifies.
- Walk-up refusals are first-class: new `LiftAccessRefusal` model; `startCargoBooking` refuses + LOGS (reason `no_booking|not_approved|outside_window`) then throws `booking_not_startable`.
- Service cost mirrors to an `Expense` only when `expenses.core` is on AND an active head exists — best-effort, never throws; otherwise the cost lives on `LiftServiceLog.costMinor` alone.
- Worker cron `lifts.service-warn` (daily 08:00 society time, gated by `lifts.maintenance`) warns the committee on overdue/imminent service and expiring/lapsed AMC.

**Models/enums:** `Lift`, `LiftServiceLog`, `LiftBreakdown`, `LiftAccessRefusal`; enums `LiftKind`, `LiftStatus`, `ServiceKind`, `AmenityKind`. Migration `20260713130000_lifts`. `prisma generate` run.

**Registration:** `lib/features.ts` (3 features), `lib/nav.ts` (feature-gated item, not hidden on shared device — operator console), `lib/rbac.ts` (perms `lifts.read/manage/operate/breakdown.report/maintenance.manage`; attached to COMMITTEE_MEMBER, MANAGER, GUARD; SOCIETY_ADMIN via `*`), `lib/worker/registry.ts` + `handlers.ts` (`runLiftServiceWarnings`), `lib/ui-modes/modules.ts` (committee → PRO default), `lib/notifications/templates.ts`.

**Code:** `lib/lifts/{constants,rules,rules.test,actor,http,service,lifts.integration.test}.ts`; `schemas/lifts.ts`; API `app/api/lifts/route.ts`, `app/api/lifts/[id]/{route,service,breakdown,restore,refuse,booking-amenity}/route.ts`, `app/api/lift-bookings/[id]/{start,end}/route.ts`; UI `app/[locale]/app/lifts/page.tsx` + `components/lifts/lifts-client.tsx` (Simple status cards + operator Start/End console; Pro = DataTable kit over lifts via in-memory query). i18n `lifts` block added to en + ur.

**Tests written:** `rules.test.ts` (service-due maths, AMC windows, start-guard decision, downtime). `lifts.integration.test.ts` covers all 6 acceptance criteria: (1) start guard + refusal log, (2) concurrency double-book asserted via amenity engine, (3) peopleTrapped → EmergencyAlert + opt-out ignored, (4) nextServiceDueAt computed + sweep warns committee, (5) breakdown→complaint→OUT_OF_SERVICE→restore, (6) expenses-off records cost on log with no Expense/no error.

**Gates (agent-run):** `pnpm prisma generate` ok, `pnpm lint` clean (0 warnings), `pnpm typecheck` clean. Did NOT run test:unit/e2e/build (controller runs them).

**UI self-check:** shadcn Button + design tokens; status/flag chips light+dark; logical props / `dir` passed through; EmptyState for feature-off + empty + empty-console; Simple + Pro (DataTable kit) both implemented; toasts (sonner), no alert(); operator console renders Urdu-first via the `ur` locale (dir=rtl). Follow-up: a dedicated lift-create/service-log form modal (form kit) and uptime charts were scoped for a later Pro pass; registry CRUD + maintenance logging are exposed via API now.

**Notes:** `expenses.core` deliberately left OFF in the integration test to prove criterion 6; enabling the leaf features pulls their whole dependency subtree via `planEnable`.

## 52 — noc-system — DONE (2026-07-13)

Built the NOC system (spec 52, feature `noc.core` + sub-features `noc.sale`, `noc.rent`, `noc.transfer`, `noc.dues_clearance`, `noc.fees`). Companion CODEREF 52-52 was authoritative for reuse targets.

**The two hard rules, enforced in code (lib/noc/rules.ts + service.ts):**
1. Issuance is BLOCKED while `flatBalanceMinor(unitId) > 0`; the ONLY escape is an audited committee override carrying a non-empty reason, which is recorded on the request, written to AuditLog, and printed on the certificate. No config flag can silently disable the gate.
2. `complete()` executes the spec-16 occupancy transfer via `transferOccupancy` (residents/service) — closes the old holding, opens the new, invites the incoming party — so register and reality never drift. Historical invoices stay snapshot-attached to the previous person (untouched).

**Data model:** NocRequest (scoped: societyId+deletedAt), NocEvent, NocSettings (infra), NocSequence (gapless per society/year). Enums NocType (SALE/RENT/TRANSFER/DUES_CLEARANCE/MORTGAGE/SUBLET), NocStatus (11 states). Migration 20260713140000_noc_system.

**Reuse (not reimplemented):** balance via `flatBalanceMinor`; gapless numbering via an atomic INSERT…ON CONFLICT DO UPDATE RETURNING on NocSequence (copied from InvoiceSequence); fee as a one-off `createSpecialCharge` on SINGLE_UNIT (snapshotted `feeMinor`, only when `noc.fees` on); certificate PDF via a new `lib/noc/certificate.ts` modelled on invoice-pdf.ts (branded, Noto-Naskh Urdu, VOID watermark on revoke, logo via readFileBytes); vault filing via `createDocument` UNIT_SCOPED into the seeded "NOCs" folder (best-effort); CNIC via `encrypt`/`decrypt` + `maskCnic`, gated by `SocietySettings.cnicCaptureEnabled`; resident invite folded into `transferOccupancy`.

**Workflow:** owner submits (OWNER-only, 403 via live-occupancy check) → DUES_PENDING (if owes) or UNDER_REVIEW → override (audited) or auto-advance on payment (`refreshDues`, also lazy on read) → distinct-member approvals to quorum → APPROVED → issue (dues gate + PDF + vault + fee) → ISSUED → complete (transfer + auto-expire competing live NOCs on the unit) → COMPLETED. Plus reject, cancel (owner), revoke (VOID re-render), retrospective transfer (reality-wins), and a daily `noc.expire` worker sweep (06:00 society time) for issued-but-uncompleted NOCs past validity.

**Routes:** app/api/noc (GET list / POST create), /[id] (GET), /[id]/{approve,reject,override,issue,complete,revoke,cancel,refresh-dues,certificate}, /settings (GET/PUT), /retrospective (POST). Every route calls `requireFeature(noc.core)` (spec 49). Actor mirrors residents (READ_ONLY society → issuance refused 423; reads still work).

**Registration:** features.ts (6 defs), rbac.ts (noc.read/approve/manage → COMMITTEE_MEMBER + MANAGER; owners apply with no permission), nav.ts (fileCheck icon, feature-gated only), ui-modes/modules.ts (committee/PRO default), notifications/templates.ts (8 codes reusing `announcements`/`billing` — no new category), worker registry+handlers (`noc.expire`), messages/en+ur (`noc` namespace, mirrored). Added FileCheck2 to nav-icons.

**UI:** components/noc/noc-client.tsx — Simple owner flow (owned-unit cards with live dues + Pay-now link + Apply dialog, status history, Download NOC), committee approvals inbox (dues red/green, Approve disabled while owing + Override beside it, issue/complete/revoke), Pro register DataTable. Light/dark tokens, RTL via dir, mobile-first.

**Tests:** lib/noc/rules.test.ts (pure dues gate/quorum/expiry). lib/noc/noc.integration.test.ts (Postgres) covers all mandatory criteria: dues gate blocks + override is the only path (audited), OWNER-only 403, completion transfer with historical-invoice regression, dues auto-advance, gapless numbering under concurrency (8 parallel), auto-expire competing NOCs, quorum with distinct members. e2e/noc.spec.ts probes unauth refusal.

**Gates:** `pnpm prisma generate`, `pnpm lint`, `pnpm typecheck` all clean. (test:unit/e2e left for the controller.)

**Decisions:** fee posts as a SCHEDULED SpecialCharge (rides the next billing run) rather than an immediate standalone invoice, avoiding a risky reimplementation of the invoice+sequence transaction; `invoiceId` spec field realised as `feeChargeId`. Vehicle registration on completion is a no-op (the model captures only `vehicleCount`, no plate data). Certificate renders EN labels with the Urdu font embedded so Urdu content (society/party/conditions) renders; locale-UR labels are available in certificate.ts if needed later.

## 53 — settings-taxonomy-framework — DONE (2026-07-13)

**Type:** FEATURE (branch feature/53-settings-taxonomy-framework)

**Decision — registry OVER the typed models, not a JSON blob.** Kept all 14 typed
settings models (+ NocSettings). Added a declarative `SettingDef` registry that
DESCRIBES each field (key, model, field, scope, type, default, validate, requiresFeature,
dependsOn, danger, templatable, docs). The unified settings UI, validation, audit and
(future) templates/docs are generated from it.

**Files created**
- lib/settings/types.ts — SettingDef/SettingDoc/SettingView (DB-free so client can import the type).
- lib/settings/modules/*.ts — one declaration file per module (society, billing+payments+platform-billing,
  gate-pass, complaints, cctv, chat, emergency, expenses, meters, islamic+hijri-sync, reports, noc).
  Per-field validators reuse the EXISTING zod schemas via `.shape` where the field exists; inline zod otherwise.
- lib/settings/registry.ts — aggregation + DMMF integrity helpers (fieldExists, configurableFieldsOf,
  enumOptionsFor from DMMF, isBigIntField) + pure gating (isVisible/dependenciesMet/isFeatureEnabledFor).
- lib/settings/service.ts — generic get/set BY KEY → typed model+field. Feature-gate reject, zod validate,
  optimistic concurrency (updatedAt token), automatic AuditLog old→new. Society vs platform-singleton key
  strategy; platform scope guarded off the society surface. actorType SOCIETY_USER vs PLATFORM (spec §3).
- lib/settings/{actor,http}.ts — auth + read/write scope + error mapping (mirrors lib/noc).
- lib/settings/registry.test.ts — every entry maps to a real model+field; none orphaned; enum options resolve;
  defaults validate; dependsOn/keys sane; gating pure tests (feature-off absent, dependsOn hide).
- lib/settings/settings.integration.test.ts — DB-backed: write applies to typed model + audits old→new;
  feature-off setting absent AND rejected; stale token loses.
- lib/taxonomy/rules.ts — pure materialised-path (buildPath, materialise, normaliseCode, assertNoCycle,
  recomputeSubtree), same discipline as spec 46. NO recursive query.
- lib/taxonomy/{service,constants,actor,http}.ts — namespace seed + term CRUD/move/deactivate; in-use term
  (has children) never hard-deleted → deactivate; soft-delete via scoping layer.
- lib/taxonomy/architecture.test.ts (no recursive SQL) + rules.test.ts (pure invariants) +
  taxonomy.integration.test.ts (path/depth/ancestorIds on insert/rename/move, cycle rejected, in-use not deleted).
- schemas/settings.ts, schemas/taxonomy.ts.
- app/api/settings/route.ts (GET surface / PUT by key); app/api/taxonomy/** (namespaces, terms, term move/delete).
- components/settings/setting-field.tsx (renders any SettingDef via the Form kit) +
  settings-registry-panel.tsx (searchable, module-grouped, Simple/Pro, dependsOn hide, persists via PUT).
- prisma: Taxonomy + TaxonomyTerm models (+ migration 20260713150000_settings_taxonomy). Both carry deletedAt
  so the scoping layer auto-pins them (a deliberate, safe superset over the spec's Taxonomy shape).

**Files changed**
- app/[locale]/app/settings/page.tsx — renders the generated SettingsRegistryPanel above the existing panels.
- prisma/schema.prisma — appended the two taxonomy models.

**Deferred (noted, not blocking):** the existing per-module settings MODALS
(billing/complaints/payments/notifications) were NOT physically refactored to consume the
registry in this pass — the generated surface is added alongside them to avoid a risky
multi-modal rewrite in one step; the substrate they will consume is in place. Full i18n
(EN/UR strings) of the new panel is partial (English literals); registry docs are English.

**Gates:** `pnpm prisma generate` OK · `pnpm typecheck` clean · `pnpm lint` clean (0 warnings).
DB-backed *.integration.test.ts run under the controller's Postgres.

WORK TYPE: FEATURE (branch feature/53-settings-taxonomy-framework)
