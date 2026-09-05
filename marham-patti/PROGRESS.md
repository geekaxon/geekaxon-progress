> ⚠️ **PUBLIC FILE** — no secrets, hostnames, endpoints or keys.

# PROGRESS.md — Marham Patti Build Tracker (SHORT)

> SHORT tracker; detail lives in PROGRESS-HISTORY.md. A completed step is NEVER rebuilt; a dangling "Next" means [HUMAN_REQUIRED].

## Current Status
- **Project:** Marham Patti — multi-tenant white-label healthcare platform.
- **Phase:** **PHASE 46** — Design block (390–401) complete. Released as v1.0.0.
- **Last completed:** **401 — zero-downtime-release-pipeline** — DONE (v1.0.0, three environments).
- **Next:** none — build order complete
- **OWNER FIRST:** Actions billing, a self-hosted runner, the `develop` branch and the slot ports in each `.env`.
- **MOCKUPS:** `specs/mockups/pharmacy/*.html` is the design target.
- **ASSEMBLED, NOT BUILT:** mount the kits that exist; a fresh copy fails the spec.
- **ONE LIVE LABEL:** bus-subscribed pages mount `PageLiveSync`; a second live chip fails.
- **After this:** Pharmacy final audit, consistency audit, Lab, Clinic.

### Recent steps
- **401 — zero-downtime-release-pipeline** — DONE (2026-09-05) — The new build starts beside the old one, proves itself, then takes over.
- **400 — audit-log-to-mockup** — DONE (2026-09-05) — The trail reads like a page now: who did what, from which counter, and what moved.
- **399 — notifications-centre** — DONE (2026-09-05) — The bell finally has a page, and marking things read agrees everywhere at once.

> Older steps in PROGRESS-HISTORY.md
