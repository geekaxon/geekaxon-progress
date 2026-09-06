> ⚠️ **PUBLIC FILE** — no secrets, hostnames, endpoints or keys.

# PROGRESS.md — Marham Patti Build Tracker (SHORT)

> SHORT tracker; detail lives in PROGRESS-HISTORY.md. A completed step is NEVER rebuilt; a dangling "Next" means [HUMAN_REQUIRED].

## Current Status
- **Project:** Marham Patti — multi-tenant white-label healthcare platform.
- **Phase:** **PHASE 48** — Fix round after the Phase 47 test (409–414). 411–414 are `fix/*` (`DEPLOY FIX`). Target branch is still `staging`.
- **Last completed:** **410 — print-helper-installer** — DONE.
- **Next:** **411 — thermal-receipt-to-golden** — /specs/411-thermal-receipt-to-golden.md
- **Group order:** 409 → 410 → 411 → 412 → 413 → 414. One step per session. Never skip a number.
- **GOLDENS ARE THE ACCEPTANCE (411):** `specs/411-goldens/*.txt` are extracted from the mockup; the renderer's rows must equal them byte for byte; goldens are never regenerated from the renderer.
- **EVIDENCE FIRST (414, 412):** side-by-side mockup-vs-deployed screenshots committed under `specs/evidence/` before and after; the after-image is the acceptance.
- **ONE PATH CONSTANT PER STREAM (413 §1):** web and API read the same constant.
- **After this:** owner's 401 steps → v1.0.0 promote → Pharmacy audit → consistency audit → Lab → Clinic.

### Recent steps
- **410 — print-helper-installer** — DONE (2026-09-06) — one-click installer, printing by printer name via the spooler, six-digit pairing.
- **409 — ci-runtime** — DONE (2026-09-06) — CI moved to the self-hosted runner with timeouts, a turbo cache and a future-dated-migration gate.
- **411–414** — AUTHORED — receipt to golden · POS print dialog · mobile/list polish · merge rail evidence.

> Older steps in PROGRESS-HISTORY.md
