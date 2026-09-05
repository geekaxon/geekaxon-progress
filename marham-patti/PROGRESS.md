> ⚠️ **PUBLIC FILE** — no secrets, hostnames, subdomains, endpoints, credentials or keys. Placeholders only.

# PROGRESS.md — Marham Patti Build Tracker (SHORT)

> SHORT tracker; full detail in PROGRESS-HISTORY.md. Keep under 1.5 KB. A completed step is NEVER rebuilt — a dangling "Next" means [HUMAN_REQUIRED], not an earlier step.

## Current Status
- **Project:** Marham Patti — multi-tenant white-label healthcare platform.
- **Phase:** **PHASE 46** — Design block after the Phase 45 test (390–401). 390 is `fix/*` (`DEPLOY FIX`); 391–401 are `feature/*` (`DEPLOY FEATURE`).
- **Last completed:** **390 — leftovers-and-live-sync** — DONE.
- **Next:** **391 — recent-sales-desktop-to-mockup** — specs/391-recent-sales-desktop-to-mockup.md
- **Group order:** 390 → 391 → 392 → 393 → 394 → 395 → 396 → 397 → 398 → 399 → 400 → 401. One step, then stop at the checkpoint.
- **MOCKUPS:** `specs/mockups/pharmacy/*.html` — **HTML is the design target** (no PNGs from this block). Read copy, states and behaviour from the file; match its layout.
- **ASSEMBLED, NOT BUILT:** every screen mounts kits that exist (KPI, chips, tables, drawer, payment dialog, export, sheet kit, `@mp/escpos`). A component written fresh where one exists fails the spec.
- **ONE LIVE LABEL:** every bus-subscribed page mounts `PageLiveSync`; a second live chip fails the spec.
- **ONE RENDERER (396–397):** goldens re-baselined from the mockup's §8, never from the code under test; thermal and A4 share one document model.
- **401 needs the owner:** Actions billing cleared + self-hosted runner registered before its merge; v1.0.0 = the 374–401 promote.
- **BRANCH RULE:** create and switch to the spec's own branch BEFORE any commit.
- **Numbering is continuous — never skip a number.**
- **Sequence after this:** final Pharmacy audit → consistency audit → Lab → Clinic.
- **Production:** live since 1 Sep on `release`; releases follow docs/RELEASE-RUNBOOK.md (rewritten by 401).

### Recent steps
- **390 — leftovers-and-live-sync** — DONE (2026-09-05) — Owner holds every key in code; three routes retired; one live marker per page.
- **389 — gate-repair-387** — DONE (carried 387).
- **388 — rolling-month-window** — DONE.

> Older steps in PROGRESS-HISTORY.md
