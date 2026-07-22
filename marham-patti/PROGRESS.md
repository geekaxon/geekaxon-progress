> ⚠️ **PUBLIC FILE** — no secrets, hostnames, endpoints or credentials. Placeholders only.

# PROGRESS.md — Marham Patti Build Tracker (SHORT)

> SHORT tracker; full detail in PROGRESS-HISTORY.md (append-only). Keep under 1.5 KB. Each step ends `[CHECKPOINT]`; only stop is `[HUMAN_REQUIRED]`. A completed step is NEVER rebuilt.

## Current Status
- **Project:** Marham Patti — multi-tenant white-label healthcare platform (Ganatra go-live pending).
- **Phase:** 1–97 ✅ (vendor console). Now **PHASE 12 (98–112)** — the Pharmacy module (standalone retail POS + inventory + purchase + customers + returns + day-close). CODEREF 98-112.
- **Last completed:** **100 — module-aware-app-shell-and-counter-selection**.
- **Next:** **101 — pos-cart-and-checkout** — /specs/101-pos-cart-and-checkout.md.
- **Branches:** through feature/97 on staging. Pharmacy on feature/98…112.

### Recent steps
- **100 — module-aware-app-shell-and-counter-selection** — DONE (2026-07-22) — counter/till picker bound to the session, shown in shell, carried downstream; nav stays module-aware by flags.
- **99 — pharmacy-settings-and-configuration** — DONE (2026-07-21) — resolver + admin-gated settings screen for tax, credit, prescriptions, counters, restocking fee, with zero-config defaults.
- **98 — pharmacy-data-models-and-migrations** — DONE (2026-07-21) — additive retail schema: customers+credit, returns, counters, day-close, settings — all tenant-scoped under RLS.

> Older steps: PROGRESS-HISTORY.md.
