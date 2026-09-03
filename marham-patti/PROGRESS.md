> ⚠️ **PUBLIC FILE** — no secrets, hostnames, subdomains, endpoints, credentials or keys. Placeholders only.

# PROGRESS.md — Marham Patti Build Tracker (SHORT)

> SHORT tracker; full detail in PROGRESS-HISTORY.md. Keep under 1.5 KB. A completed step is NEVER rebuilt — a dangling "Next" means [HUMAN_REQUIRED], not an earlier step.

## Current Status
- **Project:** Marham Patti — multi-tenant white-label healthcare platform.
- **Phase:** **PHASE 43** — the walk, the runbook, returns and customers close.
- **Last completed:** **368 — allocation-walk-source**.
- **Next:** **369 — release-runbook** — specs/369-release-runbook.md
- **Group order:** 369 → 370 → 371 → 372 → 373. One step, then stop. Production is LIVE on `release`.
- **OWNER ACTION before the reseed:** run the reconciliation with apply=true on staging, then the instrument; RESEED (360) stays unrun until it reads clean.
- **RULES:** branch first, before any commit; numbering is continuous.
- **Sequence after this:** reseed → Settings testing → Day-close → Accounting → Prints → Dashboard → audits.
- **Before production:** blank VENDOR_BOOTSTRAP_* after first vendor login + restart. Held for build: A4 invoice, thermal receipt + statement; hardware pending.

### Recent steps
- **368 — allocation-walk-source** — DONE (2026-09-03) — a new delivery now runs the walk; an opening balance is not drift.
- **367 — recent-sales-close** — DONE (2026-09-02) — the standing rules, walked over the sales register.
- **366 — sales-register** — DONE (2026-09-02) — the sales register is a document, assembled from parts.

> Older steps in PROGRESS-HISTORY.md
