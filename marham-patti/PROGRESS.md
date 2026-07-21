> ⚠️ **PUBLIC FILE** — no secrets, hostnames, endpoints or credentials. Placeholders only.

# PROGRESS.md — Marham Patti Build Tracker (SHORT)

> SHORT tracker; full detail in PROGRESS-HISTORY.md (append-only). Keep under 1.5 KB. Each step ends `[CHECKPOINT]`; only stop is `[HUMAN_REQUIRED]`. A completed step is NEVER rebuilt.

## Current Status
- **Project:** Marham Patti — multi-tenant white-label healthcare platform (Ganatra go-live pending).
- **Phase:** 1–97 ✅ (vendor console, under owner testing). Now **PHASE 12 (98–112)** — the Pharmacy module: first tenant operational module (standalone retail POS + inventory + purchase + customers + returns + day-close). CODEREF 98-112.
- **Last completed:** **98 — pharmacy-data-models-and-migrations**.
- **Next:** **99 — pharmacy-settings-and-configuration** — /specs/99-pharmacy-settings-and-configuration.md.
- **Branches:** through feature/97 on staging. Pharmacy on feature/98…112.

### Recent steps
- **98 — pharmacy-data-models-and-migrations** — DONE (2026-07-21) — additive retail schema: customers+credit, returns, counters, day-close, settings — all tenant-scoped under RLS.
- **97 — vendor-ui-fixes-round-3** — DONE — tenant logo/branding fixes, palette colours, dark breadcrumbs, vendor PWA icon, blended AI rate, plain-English AI task copy.
- **96 — host-aware-routing-and-tenant-branding** — DONE — host×path routing matrix, tenant branding on tenant surfaces, relative API base URL.

> Older steps: PROGRESS-HISTORY.md.
