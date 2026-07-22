> ⚠️ **PUBLIC FILE** — no secrets, hostnames, endpoints or credentials. Placeholders only.

# PROGRESS.md — Marham Patti Build Tracker (SHORT)

> SHORT tracker; full detail in PROGRESS-HISTORY.md (append-only). Keep under 1.5 KB. Each step ends `[CHECKPOINT]`; only stop is `[HUMAN_REQUIRED]`. A completed step is NEVER rebuilt.

## Current Status
- **Project:** Marham Patti — multi-tenant white-label healthcare platform (Ganatra go-live pending).
- **Phase:** 1–97 ✅ (vendor console). Now **PHASE 12 (98–112)** — the Pharmacy module (standalone retail POS + inventory + purchase + customers + returns + day-close). CODEREF 98-112.
- **Last completed:** **112 — realtime-notifications**.
- **Next:** none — no spec beyond 112. Stop with [HUMAN_REQUIRED].
- **Branches:** through feature/97 on staging. Pharmacy on feature/98…112.

### Recent steps
- **112 — realtime-notifications** — DONE (2026-07-22) — per-tenant SSE bus + notification-centre bell with unread state, PWA-push seam, role/actor-routed pharmacy events.
- **111 — keyboard-shortcuts** — DONE (2026-07-22) — per-user key map: conflict-free system defaults + personal rebind/reset, desktop-only runtime that yields to typing, and a "?" help overlay.
- **110 — day-close-and-cashier-reconciliation** — DONE (2026-07-22) — per-counter reconcile: cash/card/credit split, denom count → over/short, close locks the day, whitelabel Z-report.

> Older steps: PROGRESS-HISTORY.md.
