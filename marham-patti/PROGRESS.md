> ⚠️ **PUBLIC FILE** — no secrets, hostnames, subdomains, endpoints, credentials or keys. Placeholders only.

# PROGRESS.md — Marham Patti Build Tracker (SHORT)

> SHORT tracker; full detail in PROGRESS-HISTORY.md (append-only). Keep under 1.5 KB. Each step ends `[CHECKPOINT]`; only stop is `[HUMAN_REQUIRED]`. A completed step is NEVER rebuilt — an empty or dangling "Next" means [HUMAN_REQUIRED], not an earlier step.

## Current Status
- **Project:** Marham Patti — multi-tenant white-label healthcare platform (Ganatra go-live pending owner approval).
- **Phase:** 1–97 ✅ (vendor console complete, under owner testing). Now **PHASE 12 (98–112)** — the Pharmacy module (standalone-capable retail POS + inventory + purchase + suppliers + customers + returns + reports + day-close + shortcuts + notifications). Companion: CODEREF 98-112.
- **Last completed:** **98 — pharmacy-data-models-and-migrations** (2026-07-21) — additive models (Customer, CustomerPayment, PurchaseItem, SaleReturn, PurchaseReturn, ReturnItem, Counter, DayClose, CashCount, KeyboardShortcut, PharmacySettings) + enums + Medicine columns, all tenant-scoped under RLS.
- **Next:** **99 — pharmacy-settings-and-configuration** — /specs/99-pharmacy-settings-and-configuration.md.
- **Branches:** through feature/97 (vendor) merged to staging; feature/98 built. Pharmacy block continues 99…112.

### Recent steps
- **98 — pharmacy-data-models-and-migrations** — DONE (2026-07-21) — one additive migration: new pharmacy models/enums + Medicine columns (taxRate, minStock, shelfLocation, active), all RLS-forced and tenant-scoped.
- **97 — vendor-ui-fixes-round-3** — DONE — tenant logo in list, branding upload persistence, single role, palette colours, dark breadcrumbs, rows-per-page width, vendor manifest/PWA icon, blended AI rate, plain-English AI task copy.
- **96 — host-aware-routing-and-tenant-branding** — DONE — full host×path routing matrix, tenant branding on staff auth, relative API base enforced in code.

> Older steps in PROGRESS-HISTORY.md.
