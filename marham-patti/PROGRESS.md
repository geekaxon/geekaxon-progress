> ⚠️ **PUBLIC FILE** — no secrets, hostnames, endpoints or credentials. Placeholders only.

# PROGRESS.md — Marham Patti Build Tracker (SHORT)

> SHORT tracker; full detail in PROGRESS-HISTORY.md (append-only). Keep under 1.5 KB. Each step ends `[CHECKPOINT]`; only stop is `[HUMAN_REQUIRED]`. A completed step is NEVER rebuilt.

## Current Status
- **Project:** Marham Patti — multi-tenant white-label healthcare platform (Ganatra go-live pending).
- **Phase:** 1–97 ✅ (vendor console). Now **PHASE 12 (98–112)** — the Pharmacy module (standalone retail POS + inventory + purchase + customers + returns + day-close). CODEREF 98-112.
- **Last completed:** **108 — returns-sale-and-purchase**.
- **Next:** **109 — reports** — /specs/109-reports.md.
- **Branches:** through feature/97 on staging. Pharmacy on feature/98…112.

### Recent steps
- **108 — returns-sale-and-purchase** — DONE (2026-07-22) — sale returns refund + restock the original batch; purchase returns credit the supplier; idempotent + whitelabel prints.
- **107 — customers-credit-and-ledger** — DONE (2026-07-22) — credit-gated customer desk with over-limit gauge, ledger reconciling credit sales + payments, record payment, add/edit; wired the 102 customer link.
- **106 — suppliers-list-and-ledger** — DONE (2026-07-22) — supplier desk with outstanding highlighted + stats, running ledger reconciling purchases + payments, record payment, add/edit.

> Older steps: PROGRESS-HISTORY.md.
