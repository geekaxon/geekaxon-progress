> ⚠️ **PUBLIC FILE** — no secrets, hostnames, endpoints or keys.

# PROGRESS.md — Marham Patti Build Tracker (SHORT)

> SHORT tracker; detail lives in PROGRESS-HISTORY.md. A completed step is NEVER rebuilt; a dangling "Next" means [HUMAN_REQUIRED].

## Current Status
- **Project:** Marham Patti — multi-tenant white-label healthcare platform.
- **Phase:** **PHASE 48** — Fix round after the Phase 47 test (409–414). 411–414 are `fix/*` (`DEPLOY FIX`). Target branch is still `staging`.
- **Last completed:** **411 — thermal-receipt-to-golden** — DONE.
- **Next:** **412 — pos-print-dialog** — /specs/412-pos-print-dialog.md
- **Group order:** 409 → 410 → 411 → 412 → 413 → 414. One step per session. Never skip a number.
- **EVIDENCE FIRST (412, 414):** mockup-vs-deployed screenshots under `specs/evidence/`; the after-image is the acceptance.
- **After this:** owner's 401 steps → v1.0.0 promote → Pharmacy audit → consistency audit → Lab → Clinic.

### Recent steps
- **411 — thermal-receipt-to-golden** — DONE (2026-09-06) — the mockup's forty samples are now goldens and the roll matches them row for row.
- **410 — print-helper-installer** — DONE (2026-09-06) — one-click installer, printing by printer name via the spooler, six-digit pairing.
- **409 — ci-runtime** — DONE (2026-09-06) — CI moved to the self-hosted runner with timeouts, a turbo cache and a future-dated-migration gate.

> Older steps in PROGRESS-HISTORY.md
