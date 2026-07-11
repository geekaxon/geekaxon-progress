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
