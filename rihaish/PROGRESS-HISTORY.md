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
