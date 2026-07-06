> ⚠️ **PUBLIC FILE** — pushed to a separate public progress repo (`projects-abovenext/marham-patti-progress`). **NO secrets, hostnames, real subdomains, API endpoints, credentials, or keys — ever.** Placeholders only.

# ARCHITECTURE.md — Marham Patti

> **This is the lean, always-loaded "table of contents".** Overview, principles, stack, data-model summary, **brand tokens**, the **canonical numbered build order**, and the deployment spec. DETAIL for each build-step lives in `specs/NN-slug.md` (private repo only) and is **authoritative** for that step. Missing/empty spec → the agent STOPS with `[HUMAN_REQUIRED]` rather than guessing.
>
> **Numbering (§ canonical):** build-step numbers here are the single source of truth. "Module N" / "build-step N" = the entry numbered N in §14 BUILD ORDER. Numbers are **continuous 1→N across all phases**; the phase is only a label.

---

## 1. Overview

**Marham Patti** is an AI-powered, **multi-tenant, white-label, vertical-configurable** healthcare operating system for Pakistan — clinics (OPD), laboratories, pharmacies, and home-healthcare. One platform unifies a patient/customer record across clinic, lab, pharmacy, home sample collection, billing, accounting, patient & doctor apps, an own online pharmacy, and an assist-only AI suite — and is **sold per vertical** (Pharmacy / Lab / Clinic / Clinic+Pharmacy / Full) with **sub-feature-level control per tenant**.

- **Product:** Marham Patti (the app). **Built & delivered by:** AboveNext. **First tenant:** Ganatra Clinic (Full suite).
- **Tenancy:** Multi-tenant from day one; isolation enforced by Postgres RLS. Branch-aware within a tenant.
- **SaaS model:** every capability is a **feature flag**; flags are bundled into **vertical presets**; each tenant starts from a preset and is **fine-tuned per tenant** (overrides). White-label-ready (theme/identity/sender hooks built into the foundation).
- **Core job-to-be-done:** give a non-technical owner **total, accountable control** (every rupee and every strip of medicine traceable to a person and a time) while letting **inexperienced staff perform like experts** via assist-only AI — across whichever vertical they bought.

## 2. Principles (non-negotiable, every step)

1. **AI is assist-only.** AI never independently diagnoses, prescribes, approves lab results, or sells. It drafts; a **licensed human confirms** (doctor / pharmacist / pathologist). Every AI output is a suggestion with approve/reject + audit.
2. **Accountability everywhere.** Every create/update/delete and every stock movement logs to a user, time, and reason. Sensitive reads (CNIC reveal, confidential data) are audited too.
3. **Offline-first for critical paths.** Token, vitals, POS sale, capture work offline and sync on reconnect. AI features require connectivity and **queue** until online.
4. **Paper-first reality.** Design *for* paper (one-tap tokens, print/handwrite vitals, doctor comfort levels, end-of-day photo→AI register import).
5. **Tenant isolation is sacred** and is **always-on infrastructure, never a feature flag.** `tenant_id` + RLS on every business table.
6. **Configurable per tenant (SaaS).** Every capability is a **feature flag** (sub-feature granularity). Flags bundle into **vertical presets** with per-tenant overrides. Disabled features hide in UI **and** are forbidden server-side, cleanly (no orphan routes/nav).
7. **White-label ready.** Tenant **theme tokens, app identity (name/logo/PWA/letterhead), notification sender identity, and domain resolution** are honored everywhere from the foundation, so a tenant's branding is applied at runtime across every surface.
8. **No secrets in the repo.** Only `.env.example` placeholders. ARCHITECTURE.md + PROGRESS.md are PUBLIC.
9. **Design quality + performance are requirements, not polish.** Polished shadcn/ui + tokens, light/dark, motion, animated charts, skeletons — never raw HTML. **Performance is a gate:** 90+ Lighthouse on public/patient-facing surfaces; a strict fast-&-smooth budget on internal tools (§13).
10. **"Simplicity" = ease of use for non-technical/illiterate users** (plain language, few clicks, large touch targets, icons, voice, Urdu) — NOT a basic look.
11. **Two-tier i18n.** UI text → **English + Urdu** (parity hard gate). AI-generated/conversational output → **English + Urdu + Roman Urdu**.
12. **Production & real data are human-gated.** Agent works on `staging`/dummy only. `main` deploys ONLY on the owner's explicit Telegram command.
13. **Specs are authoritative.** Current `specs/NN-*.md` is the source of truth; missing → `[HUMAN_REQUIRED]`.

## 3. Always-on infrastructure (never a feature flag, never sellable as optional)

Auth, **tenant isolation/RLS**, audit log, the offline core, the capability/flag engine itself, branding resolution, i18n, and the design/perf system. These are safety/integrity, not features. Every tenant gets them regardless of vertical or plan.

## 4. Feature flags, verticals & presets (the SaaS control layer)

- **Flag catalog** in `@mp/shared` — every gated capability is a string key (e.g. `pharmacy.pos`, `pharmacy.online`, `pharmacy.aiReader`, `lab.results`, `lab.homeCollection`, `clinic.consultation`, `clinic.commission`, `accounting.full`, `accounting.lite`, `patient.app`, `ai.suite`). **Sub-feature granularity** — modules expose multiple flags, not one. Synced + reconciled on boot (like the permission catalog).
- **Vertical presets** (a preset = a flag bundle):
  - **Pharmacy** — POS, inventory, purchases/suppliers, customer-lite record, `accounting.lite` (toggle to full); online-pharmacy / AI-reader / barcode-labels as flags.
  - **Lab** — test catalog, orders, samples + chain-of-custody, results/approval, branded reports, home-collection (flag), `accounting.full` (toggle to lite); patient app (flag).
  - **Clinic (OPD)** — patients, appointments/token, vitals, consultation, e-prescription, doctor commission, `accounting.full`; pharmacy/lab off.
  - **Clinic + Pharmacy** — OPD + pharmacy together, **lab off** (your friend's-style / "OPD & pharmacy" target).
  - **Full Suite** — everything (Ganatra).
- **Per-tenant overrides:** start from a preset, then flip any individual flag for that tenant (e.g. Clinic + a small pharmacy on + lab off).
- **Precedence (deliberate):** **flag first** (does the tenant have the capability) **then permission** (can this user use it). A feature absent by flag is invisible/forbidden regardless of role; a feature present by flag still respects RBAC.
- **One Patient model, flag-driven views.** Pharmacy-only tenants see a **light "customer" view** (name + phone + udhaar/credit + purchase history) via `customers.medicalRecord=off`; Clinic/Full see the full medical record. Same model — a pharmacy that later adds a clinic just lights up the medical fields (no migration).
- **Accounts is a flag with two modes:** `accounting.lite` (simple cash in/out) vs `accounting.full` (double-entry). Vertical-aware: it seeds the right chart of accounts and only wires posting consumers for **enabled** modules (a lab posts no COGS/inventory).

## 5. Tech stack

- **Monorepo:** pnpm + Turborepo. Packages: `@mp/shared`, `@mp/ui`, `@mp/db`, `@mp/config`, `@mp/ai`, `@mp/flags` (capability engine), `@mp/brand` (white-label theming/identity).
- **API:** NestJS (TypeScript), modular per domain. **No `/api` route prefix** (proxy strips it; §13/§15).
- **Web:** Next.js 15 (App Router) + React + Tailwind + shadcn/ui. PWA. **Performance-budgeted** (route code-split, streaming/RSC, image + font strategy) for 90+ on public surfaces.
- **ORM/DB:** Prisma + PostgreSQL 16 + **RLS**. `prisma generate` after install, before build, in CI **and** deploy.sh.
- **Cache/queue:** Redis (sessions, rate-limits, jobs, offline-sync coordination).
- **Search:** Postgres `pg_trgm` (patient/medicine fuzzy) at launch; Meilisearch optional later for the 20k–50k medicine catalog.
- **Files:** Cloudflare R2 (reports, images, proofs) — presigned, tenant-scoped keys.
- **AI:** provider-agnostic `@mp/ai`; per-tenant keys; per-task model routing; quality-first tiering; spend metering; budget caps. `[DECIDE AT BUILD]` default models/providers per task.
- **Notifications:** WhatsApp + SMS + in-app, provider-agnostic; **per-tenant sender identity hook**. WhatsApp provider + per-tenant verified number `[DECIDE AT BUILD]`.
- **Infra:** AboveNext Oracle Cloud ARM VPS, aaPanel, Nginx, PM2, Node v22, Cloudflare DNS. Staging + human-gated production. All tenants under `marhampatti.com` at launch; per-tenant custom-domain resolution is a built-but-dormant hook.
- **Printing:** 80mm thermal via browser-print CSS at launch; raw ESC/POS `[DECIDE AT BUILD]`. Barcode: USB keyboard-wedge; self-printed labels.

## 6. Domains (placeholders — never commit real hosts)

Production `<PROD_APP_HOST>`, Staging `<STAGING_APP_HOST>`. All tenants resolve under one app host at launch; the **tenant-domain resolver** maps host→tenant and is ready for future custom domains (dormant). API base / R2 host / etc. → placeholders; real values in server `.env` / GitHub Secrets.

## 7. Repos & branch flow

- **Private code:** `projects-abovenext/marham-patti` (contains `/specs`).
- **Public progress:** `projects-abovenext/marham-patti-progress` (ONLY `ARCHITECTURE.md` + `PROGRESS.md`).
- **Branches:** `feat/<NN-slug>` → **`staging`** (auto-deploy) → **`main`** (production; human-gated). Branches may stack on an unmerged predecessor.

## 8. Roles (RBAC; `RolesGuard` + `@Roles()` + permission catalog)

`SUPER_ADMIN` (AboveNext, cross-tenant, NON_TENANT keys), `TENANT_OWNER`, `ADMIN`/`MANAGER`, `DOCTOR`, `RECEPTION`, `NURSE`, `LAB_TECH`, `PATHOLOGIST`, `PHARMACIST`, `SALESMAN`, `CASHIER`, `FINANCE`/`ACCOUNTANT`, `PHLEBOTOMIST`, `RIDER`, `PATIENT`. Custom tenant roles supported (SaaS). Permissions are catalog-driven string keys, seeded + reconciled on boot. **Permissions are gated behind flags** (a role only matters for a capability the tenant has).

## 9. Auth

- **Staff:** email + password + JWT (short access + rotating refresh). **2FA mandatory** for `ADMIN`/`FINANCE`/`TENANT_OWNER`.
- **Patients/customers:** phone + WhatsApp/SMS **OTP** (no password). CNIC optional (secondary, masked + reveal-audited).
- Session timeout, lockout, reset. All auth events audited.

## 10. Data model (summary — full schema per spec; every business table carries `tenant_id`, most `branch_id`)

- **Tenant** 1─* **Branch**; **Tenant** 1─* **User**; **Tenant** 1─ **TenantFlags**/**BrandingProfile**/**Capabilities**.
- **Patient/Customer** (one model; phone primary, CNIC secondary) 1─* **PatientRelation**, **Appointment**, **Encounter**, **Prescription**, **LabOrder**, **Sale**, **PatientCredit** (udhaar), **ConsentRecord**, **NotificationPref**. Medical fields gated by `customers.medicalRecord`.
- **Appointment** ─ **Token**; **Encounter** ─ **Vitals**, **Diagnosis**, **Prescription**.
- **Prescription** 1─* **PrescriptionItem** (→ **Medicine**); privacy + dual-copy flags.
- **LabOrder** 1─* **LabOrderTest** (→ **LabTest**/**LabPackage**) ─ **Sample** (barcode, chain-of-custody) ─ **Result** (per **ResultTemplate**, age/gender ranges) ─ **ReportApproval**.
- **Medicine** (brand/generic, strength/form/pack) 1─* **Batch** (expiry, FEFO); **StockMovement** (every change → user+reason); **Sale** 1─* **SaleItem**; **CartHandoff**.
- **PurchaseOrder**/**GRN**/**Supplier**/**SupplierPayment**; **StockTransfer**.
- **HomeCollection** ─ **PhlebotomistAssignment** ─ **CollectionProof**.
- **Accounts:** **Account** (chart) ; **JournalEntry** 1─* **JournalLine** (double-entry) ; **FiscalPeriod**. Lite mode: **CashLedger**. Auto-posted from enabled modules only.
- **Commission**: per-doctor config → **CommissionEntry**.
- **AI:** **AiProviderConfig**, **AiTask**, **AiSuggestion**, **AiFeedback**, **AiMemory**, **AiBudget**.
- **Audit:** **AuditLog** (actor/action/entity/reason/time), RLS-scoped.

## 11. Brand tokens — Marham Patti (exact, from the logo; the white-label DEFAULT theme)

These are the canonical defaults baked into `@mp/brand`/`@mp/ui`; a tenant may override via white-label.
- **Brand Teal `#06888D`** — primary (icon + wordmark).
- **Teal Mid `#0A8A8E`** — links/hover.
- **Teal Deep `#045E62`** — dark surfaces / headers / dark-mode base.
- **Mint Bright `#9AF0D8`** — highlight.
- **Mint `#5DD3BD`** — accent.
- **Mint Soft `#7FCAC7`** — subtle accent / dividers.
- **Ice `#E8F8F5`** — tinted backgrounds / hover fills.
- **Ink `#0B2E30`** — text on light.
- **Slate `#5B7173`** — muted/secondary text.
- **Background `#FEFDFD`**.
- Reserved, used sparingly: a warm **Gold `#E0A458`** (primary-CTA/attention only) and **Alert `#B4453C`** (errors).
- **Accessibility:** teal+mint are neighbors (low intra-contrast) — interactive text/elements must meet WCAG AA; never mint-on-teal for anything that must be read. Primary actions = white on `#06888D`. Serif display headers + clean sans body. Light/dark. Logical CSS only; RTL-correct.

## 12. Verification gates (every step's definition of done — recorded in PROGRESS-HISTORY.md)

1. `typecheck` green (all packages).
2. `lint` clean.
3. full `turbo build` green (web + api).
4. API tests green (Jest), **new tests added** for the step; report `N/N (S suites; +X)`.
5. **i18n parity:** new **UI** keys EN+UR (hard gate). **AI** steps: AI handling EN+UR+Roman, tested.
6. **Isolation:** tenant scoping/RLS correct; no query widens beyond tenant; `➖` if N/A.
7. **Feature-flag gate:** every gated capability checks the flag **server-side** AND hides in UI; **vertical smoke tests** pass — the module builds and behaves correctly under **Pharmacy / Lab / Clinic / Clinic+Pharmacy / Full** flag sets (no orphan routes, no broken nav, clean "off" state).
8. **Performance gate:** public/patient-facing surfaces meet **Lighthouse 90+** (perf/PWA/best-practices/a11y where applicable) on mobile; internal tools meet the fast-&-smooth budget (LCP/CLS/TBT + bundle budgets, §13). Report the numbers.
9. **Design-quality self-check:** polished shadcn/ui + tokens, light/dark, skeletons, RTL — not raw HTML.
10. **Accountability self-check:** state changes write AuditLog / StockMovement where applicable.
11. **Offline self-check:** critical-path steps work offline + queue/sync.
12. **White-label self-check:** the surface reads brand tokens/identity from `@mp/brand` (no hardcoded logo/color/name) so a tenant theme override reskins it.
13. `.env.example` updated (placeholders) for new config; **no secrets** committed.

## 13. Performance budget (the "90+ where possible & best" gate)

- **Public/patient-facing** (patient PWA, online pharmacy storefront, login, any pre-auth/SEO page): **Lighthouse mobile ≥ 90** for Performance + Best-Practices + Accessibility; PWA installable. Targets: **LCP < 2.5s**, **CLS < 0.1**, **TBT < 200ms** on mid-range mobile.
- **Internal authed tools** (POS, lab, consultation, accounts, dashboards): "fast & smooth" — interaction-ready quickly, no jank; route-level **JS budget** enforced; virtualized long lists; skeletons/streaming. Lighthouse SEO not chased behind auth.
- **How:** Next.js RSC/streaming, route code-splitting, dynamic import of heavy widgets (charts/editors), image optimization (R2 + next/image), **Urdu/Nastaliq font strategy** (subset + `font-display: swap` + system fallback so Urdu never tanks LCP), defer non-critical JS, cache static assets. CI runs Lighthouse on the public routes and fails the gate below budget.

## 14. BUILD ORDER (canonical — continuous numbering; phase = label)

> Strict order, one step at a time, foundation/security first. Each step has `specs/NN-slug.md`. The specced product is **steps 1–39** (below); that is the whole build. If a referenced spec file is missing or empty, the agent STOPS with `[HUMAN_REQUIRED]` rather than guessing.

**FOUNDATION (platform, security, SaaS control, white-label, design+perf)**
1. `01-foundation-monorepo` — Monorepo, packages, DB/Prisma base, health endpoint, deploy.sh, CI, base config.
2. `02-tenancy-rls` — Tenant + Branch, `tenant_id` everywhere, **RLS**, `runWithTenant`, tenant-domain resolver (dormant custom-domain hook).
3. `03-auth` — Staff email/pw + JWT + 2FA; patient phone+OTP; lockout/reset; auth audit.
4. `04-rbac-permissions` — Role model, permission catalog (seed + boot reconcile), guards.
5. `05-capabilities-flags` — **Feature-flag engine** (`@mp/flags`): flag catalog (sub-feature), **vertical presets** (Pharmacy/Lab/Clinic/Clinic+Pharmacy/Full), per-tenant overrides, `isFeatureEnabled`, flag-then-permission precedence, clean "off" behavior.
6. `06-audit-log` — Global AuditLog, sensitive-read auditing, StockMovement pattern, query UI.
7. `07-branding-whitelabel` — `@mp/brand`: tenant theme tokens (default = §11), app identity (name/logo/PWA/letterhead), **notification sender identity hook**, domain-aware branding — applied at runtime across all surfaces via a `brand.manage` settings screen.
8. `08-i18n-base` — EN+UR UI i18n + parity gate + RTL; AI-language scaffold (EN+UR+Roman).
9. `09-notifications-engine` — Provider-agnostic WhatsApp/SMS/in-app, templates, delivery status, OTP transport, per-tenant sender. `[DECIDE AT BUILD]` provider/number.
10. `10-offline-core` — Local-first store, action queue, idempotent sync, conflict policy + compensating-write hook.
11. `11-consent-terms` — Versioned consent (staff/patient/tenant) incl. AI disclaimer; gate; editable wording. `[DECIDE AT BUILD]` legal text.
12. `12-import-export` — Generic bulk Excel/CSV import (medicines 20–50k, tests, patients, suppliers) + dry-run + error report; universal filtered export.
13. `13-ui-foundation` — `@mp/ui` design system (brand tokens, shadcn/ui, light/dark, skeletons, `<DataList>`, `<ModuleTabs>`, shortcuts, large-touch, voice), **performance foundation + Lighthouse gate**, screenshot-token (staging-only, fail-closed).

**PHASE 1 — CORE CLINIC / LAB / PHARMACY**
14. `14-patient-management` — One Patient/Customer model (phone primary/CNIC secondary), family/guardian, history/allergies/timeline, udhaar, tags, merge, inactive/deceased, notification consent. **Flag `customers.medicalRecord`** drives light-customer vs full-medical view.
15. `15-appointment-token` — One-tap offline token, doctor calendar/availability, queue + display, reschedule/no-show, end-of-day photo→AI register import (draft→verify→import). *(clinic flags)*
16. `16-vitals-station` — Nurse vitals (enter→print / print-blank→write→type/AI-read), into timeline. *(clinic flags)*
17. `17-doctor-consultation` — Comfort levels L0–L3, history, notes, diagnosis, referrals, follow-up, templates, favorites, AI-assist hook. *(clinic flags)*
18. `18-eprescription` — Paper-friendly + digital, per-doctor privacy, dual prescription, dosage/timing (UR/Roman), auto-route, letterhead, repeat, allergy check. Creates minimal **Medicine** (extended by 20/28). *(clinic/pharmacy flags)*
19. `19-laboratory` — Catalog + prep, packages, smart result templates (age/gender ranges), order→sample→result→pathologist approval, critical alerts, branded PDF, portals, TAT, re-test, amend-with-reason. *(lab flags)*
20. `20-pharmacy-pos` — Multi-salesman cart → cashier handoff → thermal slip; USB barcode + self-printed labels; medicine master (extend); discounts/returns; alternatives; per-medicine profit; AI prescription-reader→cart (assist-only). *(pharmacy flags incl. `pharmacy.aiReader`, `pharmacy.barcodeLabels`)*
21. `21-billing-cashier` — OPD/lab/pharmacy/package billing, discounts/tax, refunds w/ reason-codes + approval, cashier shift (open→count→close→variance), reconciliation, receipts, end-of-day per-person close report, pending payments.
22. `22-doctor-commission` — Per-doctor config (salary/per-patient/%/hybrid/referral), per-tenant toggles, calculation + reports. *(clinic flag `clinic.commission`)*
23. `23-accounts-finance` — **Flag `accounting.full` vs `accounting.lite`.** Full: chart, journal (debit=credit), AR/AP, expenses, cash/bank, P&L, Balance Sheet, Trial Balance, day book, period close/lock, export; **vertical-aware** posting (only enabled modules). Lite: simple cash in/out ledger + basic reports. Auto-posts from enabled billing/pharmacy/suppliers/commissions/udhaar.
24. `24-admin-dashboard` — Daily sales, doctor/lab/patient metrics, stock & expiry risk, **stock-loss/variance alerts**, pending payments, profit, audit highlights; animated charts, date/branch filter, export. **Cards adapt to enabled flags.**

**PHASE 2 — HOME COLLECTION & INVENTORY**
25. `25-home-collection` — Booking, service-area verify, slots, phlebotomist assignment, route, status timeline, charges, recollection, patient WhatsApp. *(flag `lab.homeCollection`)*
26. `26-phlebotomist-app` — PWA (offline): visits, map, doorstep barcode, signature, photo proof, status. *(flag `lab.homeCollection`)*
27. `27-sample-tracking` — Unique barcode, scan per stage, chain-of-custody, lost/damaged, mismatch guard (lab + home collection). *(lab flags)*
28. `28-pharmacy-inventory` — Batch + expiry, **FEFO**, expiry/low-stock alerts, reconciliation (expected vs physical → variance→person→reason), damaged/expired write-off, valuation, locations, role-locked adjustments (logged). *(pharmacy flags)*
29. `29-purchase-suppliers` — Suppliers, POs, GRN (creates batches), supplier payments/credit, purchase history, price tracking, reorder suggestions, returns; posts payables. *(pharmacy/lab inventory flags)*
30. `30-branch-transfer` — Branch↔branch stock transfer + tracking, branch-wise stock, optional central warehouse (activates with 2nd branch). *(multi-branch flag)*

**PHASE 3 — PATIENT & DOCTOR APPS + OWN ONLINE PHARMACY**
31. `31-patient-app` — Book appointment/home test, view/download reports, upload prescription, health records, family profiles, notifications, refill reminders, loyalty; easiest UX (voice, Urdu, big buttons). **90+ surface.** *(flag `patient.app`)*
32. `32-patient-ai-assistant` — Voice+text health assistant (EN+UR+Roman): info + **triage only**, **red-flag → doctor/emergency**, booking via chat, explain own report/prescription; consent+disclaimer+logging. *(flag `patient.aiAssistant`)*
33. `33-doctor-portal-ai` — Remote appointments/history/reports/notes/e-prescription, earnings/commission, self-schedule; **AI clinical decision support** (assist-only, logged). *(flag `clinic.doctorPortal`)*
34. `34-online-pharmacy` — Own pharmacy online: catalog, cart, prescription upload, pharmacist validation, rider delivery+tracking, rider PWA, delivery charges, COD. **90+ storefront.** *(flag `pharmacy.online`)*

**AI SUITE (cross-cutting; assist-only, quality-first)**
35. `35-ai-gateway` — `@mp/ai`: per-tenant keys, per-task model routing, tiered quality, caching, token/cost capture, AiTask/AiSuggestion base, human-in-the-loop approve/reject primitive. *(flag `ai.suite`)*
36. `36-ai-suite-clinical` — Prescription reader (→cart, human-confirm), smart medicine search, drug-interaction warning, lab-report explanation, abnormal-result flagging (human signs), note summarizer. *(flag `ai.clinical`)*
37. `37-ai-suite-ops` — No-show prediction, stock-demand prediction, expiry-risk prediction, business analytics (read-only), chatbot, call-center voice scaffold. *(flag `ai.ops`)*
38. `38-ai-feedback-learning` — 👍/👎 + correction → per-tenant memory/rules; safety-critical rules human-reviewed; no unsafe retraining. *(flag `ai.suite`)*
39. `39-ai-cost-control` — Live spend meter, per-task provider/model selection, budget caps, quality-first routing surface. *(flag `ai.suite`)*

> **This is the complete build. Launch scope = build-steps 1–39, and 1–39 is the entire specced product.** The build order ends here. There are no further specced steps; any future capability is added as a new, fully-authored `specs/NN-slug.md` (starting at 40) only when its requirements are defined — never improvised. A missing spec is always a hard stop (`[HUMAN_REQUIRED]`), never a cue to guess.

## 15. Deployment (deploy.sh spec for THIS stack)

> The agent authors `deploy.sh` in build-step 01 per this spec and extends it as services are added. `deploy.sh` only **pulls/resets an existing clone** — the repo is cloned onto the server ONCE before the first deploy.

**deploy.sh must:**
1. **Node detection** — find Node via `nvm` (`~/.nvm/versions/node/*/bin`) → **aaPanel** (`/www/server/nodejs/*/bin`) → system (`/usr/local/bin`, `/usr/bin`). Export to `PATH`; fail loudly if none.
2. **Pull/reset** the existing clone on the target branch (never clone; never touch `main` automatically).
3. **Install with dev deps:** `npm ci --include=dev` (or `pnpm install --prod=false`) — turbo/tsc/next/prisma live in devDependencies.
4. **`npx prisma generate`** AFTER install, BEFORE typecheck/build, in CI **and** deploy.sh.
5. **`npx prisma migrate deploy`** (staging). Dangerous-command fence blocks `production`/`prod`, `drop database`, `rm -rf /`, force-push.
6. **Build:** set a fresh `BUILD_ID`; `turbo build` (web + api). **Run the Lighthouse perf gate** on public routes; fail below budget (§13).
7. **Restart:** `pm2 restart` (NOT reload) via `ecosystem.config.js`; set `BUILD_ID` in env first.
8. **Graceful health-check** after restart; on failure report (controller decides fix vs `[HUMAN_REQUIRED]`).
9. **Secrets** from server `.env` (never in repo). Prefer **alphanumeric DB passwords** (no `$`, backtick, quotes).
10. **aaPanel paths:** psql/node/nginx under `/www/server/...`, not system PATH.
11. **Reverse-proxy:** API has **no `/api` prefix**; Nginx **strips `/api`**. The **tenant-domain resolver** maps host→tenant (all under one host at launch; ready for custom domains).

**Screenshot token (staging-only, fail-closed):** `?screenshot_token=<token>` grants read-only preview of authenticated pages **only** when `APP_ENV`/`NODE_ENV` is exactly `staging`; otherwise OFF. Never in production; token never hardcoded; `APP_ENV` in `.env.example`.

**Public-repo safety:** the pipeline pushing ARCHITECTURE.md + PROGRESS.md to the public repo strips/verifies no secrets or real hostnames.

---

_End ARCHITECTURE.md — detail per step in `specs/NN-slug.md` (private repo). Keep this file lean; do not inline step detail._
