> ⚠️ **PUBLIC FILE** — no secrets, hostnames, endpoints or keys.

# PROGRESS.md — Marham Patti Build Tracker (SHORT)

> SHORT tracker; detail lives in PROGRESS-HISTORY.md. A completed step is NEVER rebuilt; a dangling "Next" means [HUMAN_REQUIRED].

## Current Status
- **Project:** Marham Patti — multi-tenant white-label healthcare platform.
- **Phase:** **PHASE 47** — Fix round after the Phase 46 test (402–408). 402–404, 406 on `fix/*` (`DEPLOY FIX`); 405, 407, 408 on `feature/*` (`DEPLOY FEATURE`). Target branch is still `staging`.
- **Last completed:** **403 — live-sync-and-offline** — DONE (one marker tells the truth about time; the phone gets a banner).
- **Next:** specs/404-mobile-list-paging.md
- **Group order:** 402 → 403 → 404 → 405 → 406 → 407 → 408. One step per session. Never skip a number.
- **MOCKUPS:** `specs/mockups/pharmacy/*.html` is the design target — copy, states, layout.
- **FOUND IN SOURCE:** sale `full` from status not quantities (405 §1); POS preview draws the pre-396 receipt (407 §1).
- **ONE RENDERER:** the print preview draws the renderer's lines; a second drawing fails the spec.
- **ONE PAGING HOOK:** 50 per page, cursor, prefetch — every mobile list reads it (404).
- **ONE MARKER:** four states in `PageLiveSync`; no Offline badge anywhere; mobile has the banner, not the marker (403).
- **After this:** owner's 401 steps (GitHub runner, bot target, dev deploy) → v1.0.0 promote → Pharmacy audit → consistency audit → Lab → Clinic.

### Recent steps
- **403 — live-sync-and-offline** — DONE (2026-09-06) — four states in one marker, the corner badge gone, an offline strip on the phone.
- **402 — release-ops-fixes** — DONE (2026-09-06) — adopt the old processes as blue, prove the slot you built, stricter env check.
- **401 — zero-downtime-release-pipeline** — DONE (2026-09-06).

> Older steps in PROGRESS-HISTORY.md
