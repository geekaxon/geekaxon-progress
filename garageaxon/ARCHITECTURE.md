# GarageAxon — ARCHITECTURE

> **What to build.** This document is the single source of truth for *what* GarageAxon is and *in what order* it is built. The autonomous agent reads this with `AGENT.md` (how to work) and `PROGRESS.md` (current state) at the start of every session.
>
> **PUBLIC REPO WARNING:** This file is pushed to a public progress repo. It contains **NO secrets, credentials, hostnames with credentials, tokens, or sensitive data — ever.** Only placeholders.

---

## 1. Overview

GarageAxon is a multi-tenant SaaS platform that runs UK independent garages end-to-end: bookings, workshop, parts/inventory, invoicing, built-in double-entry accounting, staff, customer communication, an integrated call-center, and customer-facing booking surfaces. It is the operational "nervous system" of the garage.

It is part of the **Geek Axon Ltd** garage software family and a sibling to **GarageBrainPro (GBP)**, an existing live AI diagnostic product. GarageAxon and GBP are **separate products with separate databases** that interoperate as a **suite** (one login, one bill, button-driven data sharing). GBP integration is **mocked in Phase 1** and **wired for real in Phase 2**.

- **Production:** garageaxon.com
- **Staging:** garageaxon.geekaxon.com
- **Repo:** separate from GBP (its own repo; not the GBP monorepo).
- **Servers:** two dedicated Oracle Cloud VPS — one for the CRM (aaPanel + Nginx + PM2), one for telephony (FreePBX/Asterisk, set up by a human runbook, not by the agent).

---

## 2. Core principles

1. **Comprehensive underneath, dead-simple on top.** Full operational depth, operable by non-technical staff and customers. Plain language ("Money In/Out", not "journal"), few clicks, large touch targets, minimal typing.
2. **Simple ↔ Pro progressive disclosure.** Every module ships **two faces** over the **same data**: a Simple view and a Pro view. Toggleable per-module or globally, anytime, **lossless** (full detail always recorded regardless of mode). The garage sets the ceiling; the user's role decides what they see. Some modules are Simple-only (workshop board) or Pro-only (deep tools) by nature.
3. **Familiar concepts, modern execution.** Match the workflow and vocabulary UK staff already know from incumbents (TechMan etc.) — reg-first job creation; the "New Job / New Estimate / New Customer / Parts-Only Sale" primary actions; the work-in-progress board with status buckets — but render everything in the modern design system. No re-learning of concepts; far better look and feel.
4. **Always-full data capture.** No module may skip recording detail because it is in Simple mode.
5. **Multi-tenant isolation first.** Every row is tenant-scoped (Org + Branch). Cross-tenant access is impossible by construction. This is checkpoint-gated.
6. **API-first, suite-ready.** Clean, versioned API; identity & entitlements read through a swappable "suite account" interface (GBP-as-hub behind it). Designed so the hub can later become a standalone identity service with no CRM rewrite.
7. **Free vs paid is explicit.** Only services that cost us money per use are credit-metered. DVLA/DVSA are free and never metered.
8. **Design quality is non-negotiable.** Modern, polished UI via the design system below; never raw/unstyled HTML. Part of every UI module's definition of done.
9. **Staging-only autonomy; production human-gated.** The agent works on staging/dummy data only. Production deploys only on explicit owner command.

---

## 3. Tech stack

Mirrors the GBP family conventions for consistency and future integration.

- **Monorepo tooling:** Turborepo (single GarageAxon repo; separate from GBP).
- **Backend:** NestJS (TypeScript).
- **Database:** PostgreSQL. **ORM:** Prisma (generated client — `prisma generate` is mandatory before typecheck/build; see Deployment).
- **Frontend:** Next.js 14 (App Router), TypeScript.
- **Styling/UI:** Tailwind CSS + shadcn/ui component system, design tokens (Section 5). Light + dark mode.
- **Auth:** JWT via HTTP-only cookies; identity/entitlements via the suite-account interface (mock in Phase 1).
- **Realtime:** Socket.io (workshop board, QR drop-off, screen-pop).
- **Queue/jobs:** BullMQ + Redis (reminders, notifications, metered sends).
- **Storage:** S3-compatible (MinIO) for photos, signatures, recordings.
- **Search:** Postgres full-text first; Meilisearch optional later.
- **Process manager:** PM2. **Reverse proxy:** Nginx. **Panel:** aaPanel.
- **Telephony (separate server, human runbook):** FreePBX/Asterisk; an AMI-bridge service + WebSocket feeds screen-pop into the CRM; recordings sync to CRM storage.
- **Customer & technician apps:** PWA (no app-store friction; launched from the garage's site/booking page). Technician PWA is ultra-minimal (fewest taps, big targets, glanceable).
- **WordPress plugin:** separate small sub-project (PHP), distributed via WordPress.org and as a direct download; authenticates with a per-tenant garage key.
- **i18n:** scaffolding from day one; English default.

External services (providers may be swapped; configured via env, never hardcoded):
- **Email:** Postmark (domain-authenticated per garage). **SMS:** Twilio (per-garage alphanumeric sender ID). **WhatsApp:** WhatsApp Business API via Meta/BSP (per-garage business profile).
- **Payments (Phase 1):** Stripe, PayPal, GoCardless, Dojo, Payment Assist + manual (cash/bank/card-machine/cheque).
- **Vehicle data (free, unmetered):** DVLA (live per job), DVSA (MOT history).
- **Phase 2 integrations:** GBP API (real), GSF + Euro Car Parts catalogues, HaynesPro (labour times + Service Assist), accounting export (Xero/Sage/QuickBooks).

---

## 4. Data model (high level)

Tenancy root and identity (identity mirrored from the suite hub; mock in Phase 1):
- **Organisation** (garage business; tenant root) — plan/entitlements, mode settings, branding, VAT status.
- **Branch / Site** (location within an Org).
- **User** (staff; belongs to Org, optionally Branch) — roles.
- **Role** (tenant-scoped; editable defaults + custom) and **Permission** (+ dependency graph). **RolePermission**.
- **SuiteAccountRef** — interface record linking Org to the hub identity + entitlements (mock in Phase 1).

Customers & vehicles:
- **Customer** — contact info, customer group, account-customer flag, on-stop, payment terms, priority message, accounting code, tax code, **three consent streams** (operational / reminders / marketing) each with channel prefs, lead source.
- **Vehicle** — reg (VRM), DVLA snapshot (per-fetch), MOT cache (DVSA), customer link. DVLA re-fetched live per job; snapshot stored on job.
- **NotificationPreference** — resolved per job from reg → customer's last reg → garage default.

Operations:
- **Booking** — reg, DVLA snapshot, date/time, reference, reason, source (widget/plugin/hosted/microsite/API/phone/walk-in), branch.
- **BookingConsent** — signed T&Cs (QR drop-off): ticked confirmations + signature + timestamp.
- **VehicleConditionReport** — damage markers (coords + notes), tyre treads, photos, customer + technician signatures, collection/drop-off variant.
- **Job** — the spine. Status pipeline (Section 6a), branch, vehicle, customer, assigned technician(s), clocking, DVLA snapshot, links to estimate/EVHC/parts/invoice. Job-level P&L.
- **JobStatusEvent** — audit of pipeline transitions.
- **ServiceSchedule / EVHC** — checklist items (RAG), measurements (tread/pressure/brakes), photos, advisories, diagnostic scan uploads; customer authorisation state.
- **Estimate** — line items; delivery + read tracking (email/SMS viewed); authorised flag; converts to Job.
- **FollowUp / Task** — due-dated, attached to any record (estimate/EVHC/job/customer), overdue flag, assignee.

Parts, inventory, suppliers:
- **ProductCategory** (two-level: category → sub-category).
- **Product** — cost price, sale price, stock, supplier, EAN/barcode, cross-reference/ordering number, "add to every job/estimate" flag, tax code.
- **StockMovement**, **StockTransfer** (inter-branch), **StockLocation**, min/max levels.
- **Supplier**, **PurchaseInvoice** (+ line items), **SupplierLedger / balances**, returns.
- **PartsRequest** — technician → admin → ordered → arrived (drives board recolor + notify).
- **PartsOnlySale** — counter sale without a job.

Courtesy cars:
- **CourtesyCar** — make/model/reg, MOT/road-tax/service dates, availability (from/to + reason), enabled, current job/customer link.
- **CourtesyCarFuelRecord** — date, mileage, litres, amount (posts to ledger).

Collection & delivery:
- **CollectionDelivery** — driver assignment, courtesy-car-left-with-customer link, parking info per vehicle/address, pre-call confirmation, collection/drop-off photos.

Money & accounting:
- **Invoice** (+ line items) — statuses: paid/partial/outstanding; insurance-excess split-billing.
- **Payment** — gateway (Stripe/PayPal/GoCardless/Dojo/Payment Assist) or manual (cash/bank/card-machine/cheque).
- **Account** (chart of accounts), **JournalEntry** + **JournalLine** (double-entry; every money action posts here automatically), **Voucher** types, **DeferredRevenueSchedule** (for plans/memberships).
- **VATRecord** — standard UK VAT (per-customer tax code).

Plans & coverage:
- **ServiceItem** (priced), **PartItem** (priced) — building blocks.
- **CoverageProduct** — built from primitives (discount / inclusion / waiver / warranty / prepaid-balance) in a **Package** (one-off) or **Subscription/Membership** (recurring).
- **CustomerCoverage** — an active holding (term, balance, status); multiple per customer.
- **CoverageResolution** — at invoice time, stacks all active coverage and computes remainder.
- **Warranty** — per-part-instance, fitment-dated, garage-verified from job history.

Staff & assets:
- **Attendance**, **Holiday/Leave**, **TechnicianNote**, **ProductivityMetric**.
- **Asset** (tools/equipment) — calibration/service due dates, assignment, maintenance log, depreciation (Pro).

Platform & billing:
- **Plan** (public/hidden; feature flags + limits), **Subscription**, **Entitlement** (per-Org feature flags, e.g. crm.plan, callcenter.addon, whitelabel, gbp.addon).
- **Credit** — single unified balance per Org; **CreditPriceList** (credits per action), **CreditPackage**, **CreditTopUp**, **CreditTransaction** (every debit logged); pre-flight check before metered calls; free actions (DVLA/DVSA) excluded.
- **Branding** — per-tenant logo/colours/domain (plan-gated white-label).
- **AuditLog** — who did what (legal/consent record).
- **Setting** — per-Org + per-module mode (Simple/Pro), reminders, documents, etc.

Telephony (app-side; engine on separate server):
- **CallRecord** — unique call ID, direction, caller, linked customer/job, recording URI, DID→tenant routing.
- **Lead** — auto-created for unknown inbound callers.

> Detailed per-field schemas are produced **at build time per module** by the agent, not pre-listed here. Where a modelling decision is deferred it is marked `[DECIDE AT BUILD]` and logged in PROGRESS.md.

---

## 5. Design system (tokens)

The Builder must theme everything through these tokens (themeable per-tenant at runtime for white-label). Never hardcode colours.

**Brand (GeekAxon family):**
- `--brand-primary: #f99d23` (orange) — primary actions, accents, "Axon" wordmark
- `--brand-secondary: #8730d7` (purple) — secondary accents, gradients
- `--brand-base: #00012f` (deep navy) — dark base
- `--brand-white: #ffffff`

**Neutrals (slate family, harmonised with the family look):**
- bg `#0f172a` · card `#1e293b` · border `#334155` · text `#f1f5f9` · muted `#94a3b8` (dark mode); light-mode equivalents defined by the agent.

**Semantic:** success / warning / error / info — defined as tokens.

**Workshop job-status colours (functional, consistent everywhere):** distinct tokens for Booked, Vehicle Arrived, Under Investigation, Awaiting Authorisation, Awaiting Parts (waiting), Parts Arrived, On Hold, Work In Progress, Work Complete, Ready/No Payment, Invoiced–Awaiting Payment, Collected. (Waiting-for-parts vs parts-arrived must be visually distinct, per workshop requirement.)

**Typography:** one rounded geometric sans (e.g. Inter); defined type scale.
**Spacing / radius / shadow / motion:** tokenised; smooth transitions; animated charts where data is shown; skeleton loaders instead of blank states.
**Modes:** light + dark.
**White-label:** per-tenant override of logo + brand tokens at runtime when the plan enables it; otherwise GeekAxon defaults.

---

## 6. Module breakdown

### 6a. Job status pipeline (canonical)
Booking → On Site (Vehicle Arrived) → Under Investigation → Pending/Awaiting Authorisation → Approved → Awaiting Parts → Parts Arrived / Waiting to Rebook → Work In Progress → Work Complete → Ready (No Payment) → Ready to Drop Off → Collected. (Plus side states: On Hold, Body Work, Warranty, Invoiced–Awaiting Payment.) Simple mode collapses these into a shorter friendly set; Pro shows all. Full detail always stored.

### 6b. Modules
Grouped; the numbered **Build Order** (Section 7) is what the agent follows.

**Foundation & platform:** Tenancy/Org/Branch · Suite-account interface (mock) + SSO · Auth · Dynamic Roles & Permissions (dependency graph, tenant-scoped defaults) · Simple↔Pro mode framework · Design-system/tokens + theming · Dynamic Plans + Entitlements + public pricing site + hidden plans · Single unified Credits/metering engine · White-label branding · Super-admin console (garages, plans, billing, usage, health, audit, impersonation) · Settings · Audit log · Notifications hub + per-garage sender identity · i18n scaffolding.

**Customers & vehicles:** Customer (account controls, 3 consent streams, groups, priority message) · Vehicle + live DVLA + DVSA cache · Notification-preference resolver.

**Booking & front-of-house:** Smart Booking (widget JS embed, hosted page, microsite + custom domain + auto-SSL, API) · WordPress plugin (separate sub-project) · QR drop-off + signed T&C consent (realtime) · Vehicle Condition Report.

**Workshop & jobs:** New Job flow (reg-first) · Job pipeline · Workshop board (realtime, status buckets) · Technician PWA (queue, clocking, parts request, advisories, diagnostic scan upload) · EVHC/Service Schedule + customer online authorisation · Daily job sheet (tech/branch/garage).

**Estimates & follow-ups:** Estimates (+ read/authorise tracking, convert to job) · Follow-Up/Task engine.

**Parts, inventory, suppliers:** Products (2-level categories, cost/sale, EAN, cross-ref, add-to-every-job) · Inventory (stock, locations, min/max, transfers) · Suppliers + Purchase Invoices + ledger + returns · Parts-request flow · Parts-Only/counter sale.

**Courtesy cars & logistics:** Courtesy Car module (+ fuel records) · Collection & Delivery (drivers, parking info, photos, pre-call).

**Money & accounting:** Invoicing (+ insurance-excess split) · Payments (5 gateways + manual) · Double-entry engine + Simple/Full views + deferred revenue + UK VAT · Job P&L.

**Plans & coverage:** Service/Part item lists · Coverage engine (primitives → packages/memberships) · Coverage-resolution at invoicing · Warranties.

**Staff & assets:** Staff/attendance/holidays/notes/productivity · Assets & tools.

**Call-center (Phase 1, app-side; engine via human runbook):** DID→tenant routing · inbound screen-pop (AMI-bridge + WebSocket) · WebRTC click-to-call · call recording link by call ID · book-from-call · missed-call recovery · auto-lead for unknown callers.

**Reporting:** KPI dashboard · report suite (sales/customer/labour/stock/accounting/personnel/technician) · exports.

**Customer app (PWA):** bookings, approve/decline work, invoices, wallet, packages/memberships.

**Phase 2:** GBP real integration · GSF + Euro Car Parts · HaynesPro (labour + Service Assist) · accounting export · migration importer (TechMan/Garage Hive/Dragon2000/GA4/My Garage CRM) · fleet/insurer accounts · insurance/warranty claim workflow · tyre module depth · marketing/reviews · advanced BI.

---

## 7. BUILD ORDER (the sequence the agent follows — one module at a time)

Foundation/security first. Each step is a module; the agent completes, verifies, commits, and updates PROGRESS.md before the next. **[CHECKPOINT]** = human approval required before proceeding.

**Phase 1**
1. Repo + Turborepo skeleton + NestJS/Next scaffolding + CI + deploy.sh (per Section 8) + design tokens/theming foundation. **[CHECKPOINT]** (foundational architecture)
2. Database + Prisma baseline + multi-tenant scaffolding (Org/Branch) + tenant-isolation guards. **[CHECKPOINT]** (data model + isolation)
3. Auth + suite-account interface (mock hub) + SSO scaffolding. **[CHECKPOINT]** (auth)
4. Dynamic Roles & Permissions (dependency graph, tenant-scoped default-role edits). **[CHECKPOINT]** (permissions)
5. Simple↔Pro mode framework (per-module + global, role-aware) + UI contract (every later module ships both faces).
6. Dynamic Plans + Entitlements + public pricing site + hidden plans. **[CHECKPOINT]** (billing/entitlements)
7. Single unified Credits/metering engine (price list, packages, top-ups, pre-flight check, free-service exclusion). **[CHECKPOINT]** (billing)
8. Super-admin console (garages, plans, billing, usage, health, audit, impersonation). **[CHECKPOINT]** (admin/impersonation)
9. White-label branding (plan-gated runtime theming).
10. Settings + Audit log + i18n scaffolding.
11. Notifications hub + per-garage sender identity (email domain auth, SMS sender ID, WhatsApp business profile) + credit metering of sends.
12. Customers (account controls, 3 consent streams, groups, priority message) + customer search.
13. Vehicles + live DVLA (per-job, mismatch alert) + DVSA MOT cache (free, unmetered) + notification-preference resolver.
14. New Job flow (reg-first → DVLA → existing/new customer) + Job entity + status pipeline + JobStatusEvent. **[CHECKPOINT]** (core data model)
15. Workshop board (realtime status buckets) + Technician PWA (queue, clocking, parts request, advisories, diagnostic scan upload).
16. QR drop-off + signed T&C consent (realtime) + Vehicle Condition Report (damage markers, photos, dual signature).
17. EVHC / Service Schedule (RAG, measurements, photos) + customer online authorisation + Daily job sheet (tech/branch/garage).
18. Estimates (+ read/authorise tracking, convert-to-job) + Follow-Up/Task engine.
19. Products (2-level categories, cost/sale, EAN, cross-ref, add-to-every-job) + Inventory (stock, locations, min/max, inter-branch transfer) + Parts-request flow + Parts-Only/counter sale.
20. Suppliers + Purchase Invoices + supplier ledger/balances + returns.
21. Accounting engine (double-entry, chart of accounts, journals) + Simple "Money In/Out" and Full views + UK VAT + deferred revenue. **[CHECKPOINT]** (financial integrity)
22. Invoicing (+ insurance-excess split) + Payments (Stripe, PayPal, GoCardless, Dojo, Payment Assist + manual) → all post to ledger. **[CHECKPOINT]** (payments/financial)
23. Plans & Coverage engine (service/part items, primitives → packages/memberships, coverage-resolution at invoicing, warranties).
24. Courtesy Cars (+ fuel records) + Collection & Delivery (drivers, parking info, photos, pre-call).
25. Staff (attendance, holidays, notes, productivity) + Assets & tools (calibration, assignment, depreciation).
26. Job P&L + KPI dashboard + report suite + exports.
27. Customer app (PWA): bookings, approve/decline, invoices, wallet, packages.
28. Smart Booking surfaces: JS embed widget + hosted page + microsite (custom domain + auto-SSL) + booking API. **[CHECKPOINT]** (public-facing + custom-domain/SSL infra)
29. WordPress plugin (separate sub-project; WordPress.org-ready + direct download; garage-key auth). Directory submission is human-gated.
30. Call-center app-side: DID→tenant routing + screen-pop (AMI-bridge + WebSocket) + WebRTC click-to-call + recording link + book-from-call + missed-call recovery + auto-lead. **[CHECKPOINT]** (telephony auth, recording privacy/GDPR, tenant isolation). *Depends on the human FreePBX runbook standing up the engine.*

**Phase 2** (after Phase 1 is live and stable)
31. GBP real integration (replace mock): SSO + entitlements + button-driven two-way data sharing (Org + VRM join). **[CHECKPOINT]**
32. Parts-supplier catalogues: GSF + Euro Car Parts (credit-metered if charged). `[DECIDE AT BUILD]` order.
33. HaynesPro: labour times + Service Assist animations (credit-metered).
34. Accounting export: Xero / Sage / QuickBooks.
35. Migration importer: TechMan, Garage Hive, Dragon2000, GA4, My Garage CRM.
36. Fleet/insurer accounts + insurance/warranty claim workflow (insurance-excess split-billing).
37. Tyre module depth (brands, focus brands, tyre stock/sales/turnover).
38. Marketing & reviews (post-service review funnel, lead-source analytics).
39. Advanced cross-branch BI.

---

## 8. Deployment (deploy.sh spec for this stack)

> The agent **creates the actual `deploy.sh` in the repo** during Build Order step 1, following this spec and the hard rules in `AGENT.md`. `deploy.sh` only pulls/resets an **existing** clone (the repo is cloned onto the server once, by hand, before the first deploy).

**Servers**
- **CRM VPS:** Oracle Cloud, Ubuntu, aaPanel + Nginx + PM2. Node/psql live under aaPanel paths (`/www/server/...`), not the system PATH.
- **Telephony VPS:** Oracle Cloud, FreePBX/Asterisk — **set up by the human runbook, not the agent.** Asterisk is isolated from the app/Builder server. Recordings sync from the Asterisk box to CRM storage.

**Branches / environments**
- `staging` branch → auto-deploys to staging (garageaxon.geekaxon.com) on push.
- `main`/production (garageaxon.com) → deploys **only** on explicit owner Telegram command; never automatically.
- Agent works on staging/dummy data only.

**`deploy.sh` must include:**
1. **Node-detection block** covering: nvm (`~/.nvm/versions/node/*/bin`), aaPanel (`/www/server/nodejs/*/bin`), and system paths (`/usr/local/bin`, `/usr/bin`). Resolve a working node/npm before anything else.
2. **Pull/reset existing clone** (never clone; never touch a dir matching `production` destructively).
3. **Install with dev deps even in production:** `npm ci --include=dev` (build tooling — turbo/tsc/next — lives in devDependencies; a production-only install fails with "not found").
4. **Prisma client generation:** `npx prisma generate` **after install, before typecheck/build** (in both CI and deploy.sh) — otherwise real builds fail with "has no exported member" even when ORM-mocked unit tests pass.
5. **Migrations:** run Prisma migrations (deploy mode) against the target DB. Prefer **alphanumeric DB passwords** (no `$`, backtick, quotes) so `.env` sourcing in shell doesn't break.
6. **Fresh BUILD_ID** set before build; then build (turbo/next).
7. **Service restart:** `pm2 restart` (NOT reload).
8. **Reverse-proxy path mapping** must match the API's actual route prefix (if the API has no `/api` prefix, Nginx must strip it).
9. **Graceful health-check** after restart: poll the app's health endpoint until healthy (or fail with a clear error) before declaring success.

**Secrets**
- Never in repo. Only `.env.example` with placeholders (incl. `APP_ENV` placeholder for the staging screenshot-token mechanism). Real values live on the server / GitHub Secrets.

**Staging screenshot token (env-gated, fail-closed)**
- The app accepts `?screenshot_token=<token>` granting **read-only** preview of authenticated pages **only when `APP_ENV` (or `NODE_ENV`) is exactly `staging`**. If the env var is missing or anything other than `staging`, the bypass is **OFF** (fail-closed). Never active in production. Token never hardcoded; documented via `.env.example` (`APP_ENV` placeholder).

**Deploy error handling (in deploy/CI)**
- If a deploy fails and is **code-fixable** (build error, missing dep, bad script): the agent fixes it (max 2 attempts).
- If **not code-fixable** (server down, SSH/secret/auth failure, DNS, disk, infrastructure): do **not** attempt a fix — end the response with the literal token `[HUMAN_REQUIRED]` so the controller stops and alerts the human.

**Smart-booking infra notes (build order steps 28–29)**
- Wildcard subdomain (`*.book.garageaxon.com` or equivalent) on Nginx for hosted pages/microsites.
- Custom-domain connection: customer adds a DNS record; the app verifies and auto-provisions SSL (Let's Encrypt). DNS-verification + cert-automation is a CRM-server concern; document in the runbook portion. Anything requiring server/DNS/cert actions the agent cannot perform → `[HUMAN_REQUIRED]`.

---

## 9. Deferred decisions

All open items are tracked in `PROGRESS.md` under "Decisions / Assumptions" and marked `[DECIDE AT BUILD]` at the relevant module. Foundational/auth/billing/data/payment/telephony decisions are additionally **[CHECKPOINT]**-gated in the build order above.
