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

> **The build is fully autonomous: every step ends with `[CHECKPOINT]` (a marker, not a gate — the controller auto-merges and continues). The ONLY stop is `[HUMAN_REQUIRED]`, reserved for a missing/empty spec or infra the agent physically cannot do.** Gates 1–4 are executed by the **controller** (Node, zero tokens) and block the staging merge on failure; the agent writes the code and the tests but never runs them. Gates 5–13 remain the agent's own self-checks, done by inspection.

1. `lint` clean. *(controller-run)*
2. `typecheck` green (all packages). *(controller-run)*
3. `test:unit` green (Jest), **new tests added** for the step. *(controller-run)*
4. full `build` green (web + api). *(controller-run)*
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

**PHASE 4 — APP SHELL, DEMO DATA, ENTRANCES & DESIGN SYSTEM (usability + polish; post-launch-scope)**
40. `40-app-shell-session` — **`<AppShell>`** (role + flag-aware nav, brand-tokened, EN+UR/RTL, light/dark), `GET /auth/me` session context, **centralized API-client auth** (token + silent refresh + 401 handling; removes 34 per-component copies), post-login role-home redirect, route guard (`?next=` return), logout, SPA no-reload contract (client-side nav + AJAX forms + skeletons).
41. `41-demo-data-seed` — **Staging demo seed** (separate from baseline; gated `APP_ENV=staging` + `SEED_DEMO=1`, never in deploy): one test user per role (`Test@1234`), Pakistani-realistic patients/doctors/medicines+batches/lab catalog/suppliers + 2–3 weeks of invariant-correct history (sales, shifts w/ variance, udhaar, expiry, approvals) so every screen + dashboard shows real data; staging-only MFA relaxation (owner still enforced; production unchanged, fail-closed).
42. `42-branded-entrances` — **Foodpanda-style separate app entrances** (Staff / Patient / Rider / Phlebotomist): per-audience scoped PWA manifests + icons + logins + homes (ONE codebase/backend underneath — route groups, never forks); `/` becomes the brand entrance chooser (90+ public surface); role-mismatch redirects; white-label honored per entrance.
43. `43-auth-ui-polish` — **Dark-mode WCAG-AA contrast fix** on all auth surfaces (token-layer), premium login redesign (all entrances) + dashboard visual upgrade, public routes back within the 250 KB script budget + Lighthouse ≥ 90 (reduce weight, never raise budgets), CI contrast guard.
44. `44-design-system` — **`@mp/ui` hardened into the single premium component system** (buttons/inputs/selects/tables/cards/modals/toasts/badges/tabs/filters/search/empty/skeletons/charts — light+dark, RTL, AA, large-touch) and **applied across ALL module screens** (presentation-only migration; behavior/tests unchanged); `/ui` gallery as living reference; drift guard; perf budgets hold.

**PHASE 5 — PLATFORM & MARKETPLACE (the foodpanda model: one platform account across all tenants; vendor console; money; premium design)**
45. `45-design-system-v2` — **Inter + surface/elevation system** (light grey page, white sidebar/cards + shadows; teal becomes ACCENT, never wallpaper), three-mode theming (Light/Dark/System, default System→Light) with **per-tenant palette personalization** (aaPanel-style presets + custom, AA guard), skeleton loaders on every data surface, standard EmptyStates, Dashboard v2.
46. `46-platform-identity` — **THE MOST SAFETY-CRITICAL STEP.** Platform accounts (patient/rider/phlebotomist; phone+OTP) above tenants; tenant links; **merged patient timeline assembled only via per-tenant scoped reads** (never cross-tenant joins); consented per-document sharing (audited, revocable); claim/migration for existing patients. Isolation proofs are hard gates: a tenant must NEVER see another tenant's records or relationships.
47. `47-marketplace-discovery` — Foodpanda surface: tenant listing profiles (explicit publication), browse/search clinics/doctors/tests/medicines (preset-aware; public ≥90), cross-tenant booking/ordering via the tenants' OWN services (auto-link on first booking, `source: MARKETPLACE`), demo-seed extension adds a standalone pharmacy + lab.
48. `48-platform-settlement` — **Money (provider-AGNOSTIC: mock adapter now, JazzCash/Easypaisa/card = future config):** online-collect (platform holds, tenant payable minus commission) + cash-at-facility (tenant owes commission); double-entry append-only platform ledger; per-tenant statements + settlement runs. Ends `[CHECKPOINT]` like every step (the former owner-verification halt is retired — the agent decides, logs its reasoning in PROGRESS-HISTORY.md, and continues).
49. `49-field-pools` — Riders/phlebotomists both ways: tenant-employed (unchanged) AND shared platform pool with own-staff-first dispatch (offer/accept/expiry), job-scoped DTOs only, pool earnings → 48 ledger.
50. `50-saas-console` — Vendor portal (VendorAdmin, mandatory 2FA): create-tenant wizard (preset, branding, commission, owner invite), suspend/archive (fail-closed), per-tenant flags/branding/commission, listing approval, cross-tenant oversight (operational/financial metadata ONLY — no medical data reachable), platform audit.
51. `51-subscriptions-ai-modes` — Subscription packages (flag CEILING above presets + limits + grandfathering), tenant subscription enforcement, **AI two ways:** MANAGED (platform keys; metered per-use billing → 48 ledger at package rates) vs BYOK (tenant's own encrypted key; provider bills them) per package entitlement.
52. `52-mobile-app-experience` — Native-app feel per audience (bottom tab bar from the ONE nav registry, view transitions, safe areas, pull-to-refresh, ≥44px targets) + **QR install** on every home/dashboard (local SVG QR to that entrance URL) + install prompts/iOS instructions (EN+UR).
53. `53-tenant-addressing` — Domain-agnostic addressing: `PLATFORM_BASE_DOMAIN` + `HOSTNAME_MAP` config, precedence custom-domain → map → `<slug>.base` → shared fallback (current staging preserved byte-for-byte), tenant-host branded experience (no clinic field), custom domains with verification (package-gated), ops runbook. **No hostname is ever hardcoded — the real product domain drops in as pure config when purchased.**

**PHASE 6 — PLATFORM CONTROL, COMMERCE & THE FINAL DESIGN PASS**
54. `54-brand-config-vendor-fixes` — Real Marham Patti logo system (8 SVG variants → platform default, tenant override still wins; PWA icons/favicon/email/PDF); vendor console **English-only**; honest auth errors (a wrong password says so — never "Something went wrong"); vendor dashboard grid; **BYOK fail-closed** without `AI_CREDENTIAL_ENCRYPTION_KEY`; `any`-escape sweep in the isolation-bearing modules; **seed idempotency PROVEN by a twice-run DB test**; every Phase-5/6 env var documented.
55. `55-vendor-oversight` — Platform **audit viewer** (metadata; clinical payloads redacted), **support impersonation** done safely (reason + time-box + banner + tenant-revocable consent + owner notice + two-sided audit; VENDOR_OWNER only; the ONLY path to tenant clinical data, by *becoming* a tenant user), **global search** (⌘K, never returns patients), **per-tenant analytics + advanced charts**, vendor dashboard v2.
56. `56-advanced-rbac` — **Custom roles from a permission matrix** (clone a system role, tick permissions; system roles immutable), full **user management** (branch scope, force reset, 2FA reset, deactivate), **login history + device revoke**. Precedence becomes **flag → package ceiling → permission**; no endpoint guard is rewritten.
57. `57-ai-command-center` — **Central AI key vault** (vendor-only, encrypted, rotatable) + **per-feature model routing** (Claude here, GPT there — switch live, no deploy) + fallback; MANAGED (billed) vs BYOK resolved at the single `runAiTask` bite-point; metered **or flat** AI pricing; **learning loop** (👎 + correction → review queue → promote to few-shot / edit versioned prompt / re-route) + quality dashboard. *(Honest: this improves prompts/routing — an LLM does not self-train from thumbs-down, and the UI never claims it does.)*
58. `58-patient-directory-invites` — **Consented patient directory**: a package-gated clinic can find a patient **only if she opted in**, and sees **identity only** — never records, never which other clinics she uses; every lookup is audited and visible to her. Plus **reception → app invite** (QR/SMS) that claims and links her existing clinic record (46) — the marketplace flywheel.
59. `59-tenant-personalization-approvals` — Full tenant self-service identity (all logo variants, per-audience marks, palette, theme, domain) with **live preview**, and a **change-approval workflow**: public-facing changes enter PENDING → Marham Patti approves/rejects with reason → published atomically. SVG uploads sanitised. Finishes 53's custom-domain loop.
60. `60-notifications-email` — **Email transport** (provider-agnostic: SMTP/API/no-op) + **branded, localized templates** (EN+UR, RTL) + the full notification catalog (invites, resets, new-device, impersonation notice, appointment reminders, **report ready**, order status, receipts, settlement statements, subscription invoices, ops digests) + retry/idempotency/bounce handling + preferences (transactional mail is never opt-out; **no clinical content in messages by default**).
61. `61-billing-automation` — The business bills itself: one **consolidated periodic invoice per tenant** (subscription + AI + commission + pool fees − platform payable), gapless numbering, branded PDF, emailed; **idempotent automated run**; **dunning ladder that restricts commercial function (marketplace, AI) BEFORE anything clinical**; payments/credit notes; **revenue dashboard** (MRR/ARR, ageing, churn).
62. `62-privacy-compliance` — **Data export** (her whole record across clinics, per-link assembled), **honest erasure** (platform-level immediate; clinic-level routed as a task — the platform never unilaterally deletes a clinic's clinical record), **session/device management + revoke**, consent ledger, **public status page** (renders with the API down), declared retention + pruning.
63. `63-design-system-v3` — **The final design pass, LAST so it styles everything above.** An explicit visual target, not adjectives: Stripe-grade light (white cards on grey, hairline+shadow, **teal is an accent — never a page/card/login background**), Linear-grade **neutral dark** (no teal surfaces), **one control anatomy** (an input and a select are indistinguishable but for the chevron), **glass on chrome only, never on data**, purposeful motion, every login redesigned, **fully responsive 360→2560 (tables collapse to cards on mobile)**, drift guard that bites. Presentation-only: zero behaviour change.

> **Launch scope = 1–39 (COMPLETE). Phase 4 (40–44) usability (COMPLETE). Phase 5 (45–53) platform/marketplace (COMPLETE). Phase 6 (54–63) is the control/commerce/design scope; the specced build order currently ends at 63.** The build runs fully autonomously end-to-end — no approval gates, no sign-off halts; every step ends `[CHECKPOINT]`. Any further capability is added as a new, fully-authored `specs/NN-slug.md` (64+) only when its requirements are defined — never improvised. A missing spec is always a hard stop (`[HUMAN_REQUIRED]`), never a cue to guess.

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
