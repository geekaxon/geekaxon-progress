> ⚠️ **PUBLIC FILE** — no secrets, hostnames, endpoints or credentials. Placeholders only.

# PROGRESS.md — Marham Patti Build Tracker (SHORT)

> SHORT tracker; full detail in PROGRESS-HISTORY.md (append-only). Keep under 1.5 KB. Each step ends `[CHECKPOINT]`; only stop is `[HUMAN_REQUIRED]`. A completed step is NEVER rebuilt.

## Current Status
- **Project:** Marham Patti — multi-tenant white-label healthcare platform (Ganatra go-live pending).
- **Phase:** 1–97 ✅ (vendor console). Now **PHASE 12 (98–112)** — the Pharmacy module (standalone retail POS + inventory + purchase + customers + returns + day-close). CODEREF 98-112.
- **Last completed:** **102 — pos-payment-receipt-and-stock-decrement**.
- **Next:** **103 — inventory-list-and-medicine-detail** — /specs/103-inventory-list-and-medicine-detail.md.
- **Branches:** through feature/97 on staging. Pharmacy on feature/98…112.

### Recent steps
- **102 — pos-payment-receipt-and-stock-decrement** — DONE (2026-07-22) — cash/card/credit/split payment, idempotent FEFO stock decrement, rx confirm+log, tenant-branded 58/80mm receipt.
- **101 — pos-cart-and-checkout** — DONE (2026-07-22) — POS cart: scan/search, FEFO pick, per-line + cart discount, per-medicine tax, optional customer, hold/resume.
- **100 — module-aware-app-shell-and-counter-selection** — DONE (2026-07-22) — counter/till picker bound to the session, shown in shell, carried downstream; nav stays module-aware by flags.

> Older steps: PROGRESS-HISTORY.md.
