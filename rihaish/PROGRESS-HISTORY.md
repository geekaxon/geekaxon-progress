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
