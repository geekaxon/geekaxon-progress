> ⚠️ **PUBLIC FILE** — no secrets, hostnames, endpoints or credentials. Placeholders only.

# PROGRESS.md — Marham Patti Build Tracker (SHORT)

> SHORT tracker; full detail in PROGRESS-HISTORY.md (append-only). Keep under 1.5 KB. Each step ends `[CHECKPOINT]`; only stop is `[HUMAN_REQUIRED]`. A completed step is NEVER rebuilt.

## Current Status
- **Project:** Marham Patti — multi-tenant white-label healthcare platform (Ganatra go-live pending).
- **Phase:** 1–97 ✅ (vendor console). Now **PHASE 12 (98–112)** — the Pharmacy module (standalone retail POS + inventory + purchase + customers + returns + day-close). CODEREF 98-112.
- **Last completed:** **105 — purchase-entry-and-stock-in**.
- **Next:** **106 — suppliers-list-and-ledger** — /specs/106-suppliers-list-and-ledger.md.
- **Branches:** through feature/97 on staging. Pharmacy on feature/98…112.

### Recent steps
- **105 — purchase-entry-and-stock-in** — DONE (2026-07-22) — record a supplier delivery → batches + FEFO + latest cost, paid/partial/credit payables, whitelabel A4 invoice.
- **104 — inventory-stock-alerts-and-adjustments** — DONE (2026-07-22) — low-stock + near-expiry working views, audited stock adjustments (never negative), alert events raised for 112.
- **103 — inventory-list-and-medicine-detail** — DONE (2026-07-22) — stat strip, low-stock + near-expiry signals, medicine detail + add/edit.

> Older steps: PROGRESS-HISTORY.md.
