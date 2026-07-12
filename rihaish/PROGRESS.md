# PROGRESS.md — Rihaish

**Current build-step:** 22 — (next per ARCHITECTURE build order)
**Last completed:** 21 — society-onboarding-wizard — DONE (2026-07-12)
**Next:** 22 — (see ARCHITECTURE.md build order)

### Recent steps

- **21 — society-onboarding-wizard** — DONE (2026-07-12) — `platform.onboarding` feature (deps flats.registry, residents.registry, billing.ledger, branding.core; non-core → plan/L0-gated), perms `platform.onboarding.read/.manage`, nav `setup` (rocket, manage group). Migration `20260712240000_society_onboarding`: `SocietyOnboarding` (societyId PK, currentStep, completedSteps Int[], data Json draft, completedAt, launchedBy) — infra, explicit-scoped. **Orchestration over an already-ACTIVE tenant** (no DRAFT status; launch = completion + one-time opening-balance LOCK). `lib/onboarding`: pure `structure-plan` (multi-block bulk plan, refuses whole run before any write on collision/maxFlats) + `readiness` (rates→block, no-flats→block, no-residents→warn), `service` (draft get/save, profile via raw `prisma.society`, generateStructure, plain-language saveRates→effective-dated RateRules, snapshot, launch+lock), actor/http mirrors billing. API: `/api/onboarding` (GET bootstrap, PATCH draft), `/profile`, `/structure`(?preview=1), `/rates`, `/launch`(GET readiness/POST), `/flats` picker. Reuses existing endpoints for branding/categories/billing/opening-balance/invite/integrations. Full-screen wizard (`app/[locale]/app/setup`) inside app shell: progress rail + live "your society so far" summary + 10 step components (form-kit, Simple-only), save-&-continue. i18n en+ur parity (160 keys). Unit `structure-plan.test`/`readiness.test`; DB-gated `launch.integration.test` (rates gate, opening-balance lock-after-launch, draft survives); e2e `society-onboarding.spec` (9 unauth 401s + page no-500). **Deferred:** CSV imports at steps 3/4/7/8 reuse existing import modals (not embedded); integrations configure lives in Settings.

- **20 — resident-finance** — DONE (2026-07-12) — `billing.resident_portal` feature (deps billing.payments), nav `finance` (wallet, feature-gated NO permission), resident-scoped read/submit over 18–19. `me-finance.ts` gate `assertHoldsFlat`→403 on every flat-scoped read; summary/statement/submitPayment; branded invoice PDF. 7 hard-scoped `/api/me/*` routes. Simple+Pro UI. i18n en+ur. Unit + DB-gated integration + e2e.

- **19 — payments-receipts** — DONE (2026-07-12) — `billing.payments` feature + perms, nav `payments`. Migration `Payment`/`Receipt` + `PaymentSettings`/`ReceiptSequence`. **Money is the ledger** — CLEARED→`LedgerEntry(PAYMENT,CREDIT)`; bounce/void→`REVERSAL,DEBIT`. Receipt PDF worker. API (8), Simple+Pro. 489 unit tests + integration + e2e.

> Full history: PROGRESS-HISTORY.md · Build order: ARCHITECTURE.md
