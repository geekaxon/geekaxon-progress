> ⚠️ **PUBLIC FILE** — no secrets, hostnames, endpoints or keys.

# PROGRESS.md — Marham Patti Build Tracker (SHORT)

> SHORT tracker; detail lives in PROGRESS-HISTORY.md. A completed step is NEVER rebuilt; a dangling "Next" means [HUMAN_REQUIRED].

## Current Status
- **Project:** Marham Patti — multi-tenant white-label healthcare platform.
- **Phase:** **PHASE 48** — Fix round after the Phase 47 test (409–414). 410 is `feature/*` (`DEPLOY FEATURE`); the rest `fix/*` (`DEPLOY FIX`). Target branch is still `staging`.
- **Last completed:** **409 — ci-runtime** — DONE.
- **Next:** **410 — print-helper-installer** — /specs/410-print-helper-installer.md
- **Group order:** 409 → 410 → 411 → 412 → 413 → 414. One step per session. Never skip a number.
- **GOLDENS ARE THE ACCEPTANCE (411):** `specs/411-goldens/*.txt` are extracted from the mockup; the renderer's rows must equal them byte for byte; goldens are never regenerated from the renderer.
- **EVIDENCE FIRST (414, 412):** side-by-side mockup-vs-deployed screenshots committed under `specs/evidence/` before and after; the after-image is the acceptance.
- **PRINT BY NAME (410):** Windows printers by name through the spooler; a non-device name is refused; "sent" only when the spooler accepted.
- **ONE PATH CONSTANT PER STREAM (413 §1):** web and API read the same constant.
- **After this:** owner's 401 steps → v1.0.0 promote → Pharmacy audit → consistency audit → Lab → Clinic.

### Recent steps
- **409 — ci-runtime** — DONE (2026-09-06) — CI moved to the self-hosted runner with timeouts, a turbo cache and a future-dated-migration gate.
- **410–414** — AUTHORED — print-helper installer · receipt to golden · POS print dialog · mobile/list polish · merge rail evidence.
- **402–408** — DONE (2026-09-06).

> Older steps in PROGRESS-HISTORY.md
