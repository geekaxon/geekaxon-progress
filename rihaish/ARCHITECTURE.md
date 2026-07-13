# ARCHITECTURE.md — Rihaish

> Table of contents only. **All module detail lives in `/specs/NN-slug.md`.** Open a section only when cited.

## 1. Overview
Rihaish is a **multi-tenant SaaS** for apartment / residential-society management (Pakistan-first, Urdu + English).
Sold per society; priced **per unit** or lumpsum. Serves any residential form: single tower, multi-tower, gated compound, housing society of houses, mixed-use — and any combination inside **one** tenant.

**Three layers.** L0 **Platform** (Rihaish staff) · L1 **Society** (tenant) · L2 **Society users** (committee, manager, accountant, moderator, amenity manager, guard, staff, resident).
Guards/staff are roles inside L1, not a separate tenancy layer.

## 2. Principles (non-negotiable)
1. **Tenant isolation**: every domain row carries `societyId`, injected by one enforced data layer. A query without scope is a build failure.
2. **Everything is a feature flag**: no module ships without registering its feature code + dependency edges. L0 toggles per society; the DAG blocks bricking a tenant.
3. **Two billing domains, never merged**: *Platform billing* (Rihaish → society) and *Society billing* (society → residents). Separate tables, modules, UI.
4. **Money is a ledger**, not a mutable `paid` column. Invoice = debit, payment = credit. Never edit history; reverse it.
5. **Money = integer minor units** (BigInt) + per-society `currency` + `minorUnitDigits` (PKR = 0). Never floats.
6. **Timestamps UTC**, rendered in the society's timezone (default `Asia/Karachi`).
7. **Soft delete + audit log only.** Nothing financial, security, or user-related is ever hard-deleted.
8. **UI is part of done** (see §5). No raw HTML, no hand-rolled tables, no hand-rolled inputs.
9. **Scalable by default**: stateless web (pm2 cluster), separate worker, server-side pagination, indexed on `societyId`, heavy work queued.
10. **Secrets never in repo.** Society-supplied credentials encrypted at rest.
11. **Rihaish never carries video.** CCTV (step 41) is registry + permissions + signed view tokens only; the stream always flows society-edge → resident device, never through our servers or bandwidth.
12. **The structure is a tree, and the tree is never walked recursively.** `StructureNode` (step 46) is self-referencing at any depth; every subtree read uses the materialised `path`/`ancestorIds`. A `WITH RECURSIVE` or an N+1 tree walk is a build failure.
13. **Settings are declared, not scattered.** Every setting is registered in `lib/settings/registry.ts` (step 53) with its type, validation, feature gate and **documentation**. The settings UI, the templates and the help centre are all generated from it. A setting without docs (en **and** ur) fails the build.
14. **Most societies configure nothing.** Templates (step 54) carry the defaults; a template never carries another society's data.

## 3. Stack
Next.js 15 (App Router, RSC) · React 19 · TypeScript strict · **pnpm** · Node 22 LTS
Tailwind + **shadcn/ui** · Lucide (+ Phosphor supplement) via one `<Icon>` abstraction · Recharts · Framer-Motion-light transitions
Prisma 6 · **PostgreSQL 16** (`binaryTargets = ["native","linux-arm64-openssl-3.0.x"]` — Oracle Ampere ARM64)
Auth.js v5 · Zod (one schema, client + server) · react-hook-form · TanStack Table · next-intl (en, ur + RTL)
pm2 (**web** + **worker**) · nginx via aaPanel · SMTP email · Vitest + Playwright
Storage adapter: `local` (default) → `s3`/R2 → `gdrive`. One env var.

## 4. Data-model summary
`Society` → **`StructureNode`** (self-referencing tree, any depth: PHASE/SECTOR/TOWER/BLOCK/WING/STREET/FLOOR/COMPOUND) → **`Unit`** (`PropertyType`, `UnitCategory`, `OccupancyStatus`, `ParkingSlot`, `Vehicle`)
`User` ⇄ **`UnitOccupancy`** (many-to-many; `OWNER` | `OCCUPANT`; one account, many units) · `AuthorizedPickupPerson`
Billing: `ChargeHead` → `RateRule` (effective-dated) · `SpecialCharge` → `Invoice`/`InvoiceLine` (snapshotted) → `LedgerEntry` ← `Payment` (`PENDING|CLEARED|BOUNCED|VOID`) → `Receipt`
Platform: `Plan` · `Feature` (+`dependsOn`) · `SocietyEntitlement` · `PricingProfile` · `PlatformInvoice` · `PlatformPayment`
Ops: `Announcement` · `Complaint`/`ServiceRequest` · `Pass` (5 types) · `UtilityBillNotice` · `Expense` · `Document` · `Thread`/`Message` · `Poll` · `Survey` · `Amenity`/`Booking` · `StaffMember`/`Attendance` · `MeterReading` · `EmergencyAlert` · `Camera` · `Lift` · `NocRequest` · `Taxonomy`/`Term` · `Template` · `AuditLog` · `Job`

## 5. Design tokens & UI contract
Primary **emerald `#0B5F4A`** · Accent **warm gold `#D9A441`** (used sparingly) · full light + dark · society brand color overrides the **primary token only**.
Radius `0.625rem` · Inter (Latin) + Noto Naskh Arabic (Urdu UI) · Noto Nastaliq reserved for logo only.
Every UI module must ship: shadcn components only · light **and** dark · **RTL verified** (logical properties `ms-/me-`, never `ml-/mr-`) · responsive (guard's phone → desktop) · skeleton + empty + error states · Data-Table kit for every list (sort, column visibility, search, filters, pagination, list/card toggle, CSV/Excel/PDF export) · Form kit for every input (searchable select, date/time picker, intl phone, CNIC mask, unified field anatomy) · toasts for all feedback · animated charts · client-side navigation only (no full reload) · Simple **and** Pro variants where applicable · keyboard-navigable, AA contrast.

## 6. BUILD ORDER

| # | Spec | Module | Checkpoint |
|---|---|---|---|
| 01 | `/specs/01-foundation.md` | Foundation, design tokens, i18n/RTL, CI, deploy.sh, pm2 web+worker | [CHECKPOINT] |
| 02 | `/specs/02-tenancy-core.md` | Tenancy, host resolution, scoping layer, audit, soft delete, money/tz, screenshot token | [CHECKPOINT] |
| 03 | `/specs/03-entitlements-engine.md` | Feature registry, dependency DAG, plans, resolver, read-only mode | [CHECKPOINT] |
| 04 | `/specs/04-auth-rbac.md` | Auth, roles, multi-unit accounts, guard PIN, OTP seam, 3-layer permission guard | [CHECKPOINT] |
| 05 | `/specs/05-app-shell.md` | Sidebar, top bar (lang/theme/mode), ⌘K, motion, states, responsive shell | [CHECKPOINT] |
| 06 | `/specs/06-form-input-kit.md` | Unified fields, searchable select, date/time, intl phone, masks, modals, toasts | [CHECKPOINT] |
| 07 | `/specs/07-data-table-export-kit.md` | TanStack table, filters, saved views, list/card, CSV/Excel/PDF export engine | [CHECKPOINT] |
| 08 | `/specs/08-platform-console.md` | L0 console, societies, plans, entitlements, **impersonation** | [CHECKPOINT] |
| 09 | `/specs/09-user-account.md` | Profile, password/PIN, language, notification prefs, sessions | [CHECKPOINT] |
| 10 | `/specs/10-storage-media.md` | Storage adapter, uploads, sharp, signed URLs, camera capture | [CHECKPOINT] |
| 11 | `/specs/11-notifications-core.md` | Channel engine: in-app, SMTP, SMS/WhatsApp adapters, templates, recipients | [CHECKPOINT] |
| 12 | `/specs/12-worker-scheduler.md` | pm2 worker, job queue, cron registry, retries, DLQ | [CHECKPOINT] |
| 13 | `/specs/13-branding-domains.md` | White-label tokens, subdomain + custom domain, settings shell | [CHECKPOINT] |
| 14 | `/specs/14-ui-modes.md` | Simple/Pro: society policy + per-user prefs (entitled) | [CHECKPOINT] |
| 15 | `/specs/15-society-structure.md` | Structure + units, categories, parking modes, vehicles *(superseded by 46)* | [CHECKPOINT] |
| 16 | `/specs/16-residents-occupancy.md` | User⇄Unit, invites, CSV import, occupancy, authorized pickup persons | [CHECKPOINT] |
| 17 | `/specs/17-charge-engine.md` | Charge heads, effective-dated rate rules, special charges | [CHECKPOINT] |
| 18 | `/specs/18-ledger-invoicing.md` | Per-unit ledger, invoice runs (manual + cron), gapless numbering | [CHECKPOINT] |
| 19 | `/specs/19-payments-receipts.md` | Partial, advance/credit, bounce/reversal, late fee, proof upload, PDF receipts | [CHECKPOINT] |
| 20 | `/specs/20-resident-finance.md` | My ledger, invoices, dues, receipts, submit payment proof | [CHECKPOINT] |
| 21 | `/specs/21-society-onboarding-wizard.md` | Guided go-live for a new society | [CHECKPOINT] |
| 22 | `/specs/22-platform-billing.md` | Per-flat / lumpsum, rates, society invoices, grace → read-only, Stripe seam | [CHECKPOINT] |
| 23 | `/specs/23-pwa-mobile-shell.md` | Installable white-label PWA, bottom tabs, splash, push (VAPID), offline | [CHECKPOINT] |
| 24 | `/specs/24-announcements.md` | Targeting, pin, expiry, read receipts, comments/reactions | [CHECKPOINT] |
| 25 | `/specs/25-staff-directory-attendance.md` | Staff records, service skills, attendance | [CHECKPOINT] |
| 26 | `/specs/26-complaints-service-requests.md` | Complaint + service-request types, categories, assignment, SLA, ratings | [CHECKPOINT] |
| 27 | `/specs/27-staff-operations-console.md` | Staff "my jobs": assigned / upcoming / completed, mobile-first | [CHECKPOINT] |
| 28 | `/specs/28-gate-pass.md` | ITEM_EXIT, CHILD_EXIT, VISITOR_ENTRY, DELIVERY_LOG, STAFF_RECURRING; guard console | [CHECKPOINT] |
| 29 | `/specs/29-utility-bill-notices.md` | Providers, targeted notices, reminders, per-unit paid tracking (never in ledger) | [CHECKPOINT] |
| 30 | `/specs/30-expenses-accounts.md` | Expense heads, vendors, entries + attachments, income vs expense | [CHECKPOINT] |
| 31 | `/specs/31-document-vault.md` | Bylaws, minutes, notices; role-scoped | [CHECKPOINT] |
| 32 | `/specs/32-messaging-chat.md` | Resident ⇄ committee/admin/manager threads, attachments, push | [CHECKPOINT] |
| 33 | `/specs/33-polls-voting.md` | One vote per flat, eligibility, results | [CHECKPOINT] |
| 34 | `/specs/34-surveys.md` | Multi-question surveys, targeting, anonymity, results | [CHECKPOINT] |
| 35 | `/specs/35-amenities-booking.md` | Amenity manager, slots, rate list + approval, bookings, invoices, feedback | [CHECKPOINT] |
| 36 | `/specs/36-staff-performance.md` | Resolution time, reopen rate, ratings, leaderboard | [CHECKPOINT] |
| 37 | `/specs/37-meter-readings.md` | Readings → charge engine | [CHECKPOINT] |
| 38 | `/specs/38-emergency-alert.md` | Panic alert → guard + committee | [CHECKPOINT] |
| 39 | `/specs/39-reports-dashboards.md` | Collection %, arrears aging, defaulters, expenses, animated charts, exports | [CHECKPOINT] |
| 40 | `/specs/40-public-site.md` | `rihaish.pk` apex marketing + lead capture | [CHECKPOINT] |
| 41 | `/specs/41-cctv.md` | Bring-your-own CCTV: LINK / EMBED / AGENT tiers, per-camera grants, signed view tokens, event snapshots | [CHECKPOINT] |
| 42 | `/specs/42-permission-requests.md` | Tent/event/renovation permits + **funeral fast-track** (auto-approved, free, works when read-only) | [CHECKPOINT] |
| 43 | `/specs/43-utility-schedules-notices.md` | Loadshedding/gas/water timetables, outages, shutdowns, govt & UC notices (manual entry — no fake auto-sync) | [CHECKPOINT] |
| 44 | `/specs/44-seasonal-events-qurbani.md` | Qurbani permits (per-animal fees, per year), tie-up spaces, ballot, butcher rates, waste tasks | [CHECKPOINT] |
| 45 | `/specs/45-islamic-calendar-prayer.md` | Hijri (platform-synced), prayer times (madhab/method), masjid jamaat times, Ramadan, greetings | [CHECKPOINT] |
| **46** | `/specs/46-property-model.md` | **`Flat`→`Unit` rename + dynamic `StructureNode` tree (any depth) + property types + node/type/quantity rate rules** — RUN FIRST | [CHECKPOINT] |
| 47 | `/specs/47-platform-bootstrap.md` | 🔴 Feature-registry seed, first platform admin, multi-env pm2, `/api/health` readiness | [CHECKPOINT] |
| 48 | `/specs/48-worker-stubs-notifications.md` | Dues reminders (was a no-op), export cleanup, password-change alert | [CHECKPOINT] |
| 49 | `/specs/49-feature-registry-completeness.md` | CCTV tiers, PWA, `seasonal.permits`, `.core` naming + drift guard | [CHECKPOINT] |
| 50 | `/specs/50-cctv-architecture-guard.md` | The "Rihaish never carries video" test | [CHECKPOINT] |
| 51 | `/specs/51-lifts.md` | Lift registry, AMC/maintenance, breakdowns, cargo-lift booking (reuses the amenity engine) | [CHECKPOINT] |
| 52 | `/specs/52-noc-system.md` | NOC for sale/rent/transfer + dues-clearance gate + executes the occupancy transfer | [CHECKPOINT] |
| 53 | `/specs/53-settings-taxonomy-framework.md` | Settings registry over the 14 typed models; tenant taxonomies; L0 configures on behalf | [CHECKPOINT] |
| 54 | `/specs/54-template-library.md` | Society archetypes + module presets; apply-with-diff; versioned; never carries tenant data | [CHECKPOINT] |
| 55 | `/specs/55-settings-documentation.md` | Docs on every setting (en+ur, build-enforced), help centre, printable admin guide | [CHECKPOINT] |
| 56 | `/specs/56-platform-ops-console.md` | L0 team roles (least-privilege), guardrails on dangerous actions, in-app handbook, activity view | [CHECKPOINT] |

**Checkpoints are progress markers, not approval gates.** The build is fully autonomous: every step ends `[CHECKPOINT]`, auto-approves, and continues. There is no `[FIXED_CHECKPOINT]` and no operator approval anywhere. The **only** stop is `[HUMAN_REQUIRED]` — a missing spec, or infra the agent cannot do in code (DNS, wildcard/custom-domain SSL, Oracle firewall ports, aaPanel site config, credentials).
**Steps 46–56 are the post-audit fix + configurability + operations set.** 46 runs first (it renames the core entity); 47 is a hard blocker for any deployment. See `AUDIT.md`.

## 7. Deployment
- **Staging**: `*.rihaish.geekaxon.com` — auto-deploys on push to `staging`. **Production**: `*.rihaish.pk` — owner Telegram command only.
- Host: Oracle Cloud Free Tier, **Ubuntu, ARM64 (Ampere A1)**, 4 vCPU / 24 GB, **aaPanel** (tools under `/www/server/...`).
- Repo cloned once before first deploy. Reverse-proxy path must match the app route prefix (`/`). **Wildcard DNS + wildcard SSL (DNS-01)** and Oracle iptables/Security-List port opening are `[HUMAN_REQUIRED]`.
- Two pm2 processes: `rihaish-web` (cluster) and `rihaish-worker` (fork, single).

### `deploy.sh` (repo root, per-stack)
1. **Node detection block** — probe in order: `~/.nvm/versions/node/*/bin`, aaPanel `/www/server/nodejs/*/bin`, system paths. Export `PATH`. Fail loudly if Node < 22.
2. **Load root env** — `set -a; . ./.env; set +a`. Secrets must stay alphanumeric (no quotes/specials).
3. `git fetch --all && git reset --hard origin/$BRANCH`
4. **Install including dev deps** — `pnpm install --frozen-lockfile` (dev deps required for typecheck/build).
5. **`pnpm prisma generate`** — AFTER install, BEFORE typecheck/build (same order in CI).
6. `pnpm prisma migrate deploy`
7. `pnpm typecheck && pnpm test:unit`
8. **Fresh `BUILD_ID`** (`git rev-parse --short HEAD` + timestamp) → `pnpm build`
9. **`pm2 restart rihaish-web rihaish-worker --update-env`** (restart, never reload)
10. **Health check** — `GET /api/health` (DB + worker heartbeat) with retries; non-zero exit on failure.
