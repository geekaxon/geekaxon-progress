> ⚠️ **PUBLIC FILE** — no secrets, hostnames, subdomains, endpoints, credentials or keys. Placeholders only.

# PROGRESS.md — Marham Patti Build Tracker (SHORT)

> SHORT tracker; full detail in PROGRESS-HISTORY.md (append-only). Keep under 1.5 KB. Each step ends `[CHECKPOINT]`; only stop is `[HUMAN_REQUIRED]`. A completed step is NEVER rebuilt — an empty or dangling "Next" means [HUMAN_REQUIRED], not an earlier step.

## Current Status
- **Project:** Marham Patti — multi-tenant white-label healthcare platform (Ganatra go-live pending owner approval).
- **Phase:** 1–112 ✅ (vendor console + pharmacy on staging). Now **PHASE 13 (113–121)** — design implementation & fixes, **no business-logic change**. Companion: CODEREF 113-121.
- **Last completed:** **115 — apex-public-page**.
- **Next:** **116 — pos-to-mockup** — /specs/116-pos-to-mockup.md
- **Branches:** through feature/112 on staging. Phase 13 builds on feature/113…121.

### Recent steps
- **115 — apex-public-page** — DONE (2026-07-23) — apex root is a public marketing page; no auth, no entrance chooser; contact CTA from config.
- **114 — tenant-whitelabelling** — DONE (2026-07-23) — server-resolved brand per host; no login flash, no clinic field, one sidebar mark.
- **113 — app-shell-layout-foundation** — DONE (2026-07-22) — shell owns the content width, sidebar user card + live counts, one POS entry.

> Older steps in PROGRESS-HISTORY.md.