> ⚠️ **PUBLIC FILE** — no secrets, hosts, endpoints, credentials or keys. Placeholders only.

# PROGRESS.md — Marham Patti Build Tracker (SHORT)

> SHORT tracker; detail in PROGRESS-HISTORY.md. Keep under 1.5 KB. A done step is NEVER rebuilt; a dangling "Next" means [HUMAN_REQUIRED].

## Current Status
- **Project:** Marham Patti — multi-tenant white-label healthcare platform.
- **Phase:** **PHASE 43** — the walk, the runbook, returns and customers close.
- **Last completed:** **369 — release-runbook**.
- **Next:** **370 — returns-date-and-slip** — specs/370-returns-date-and-slip.md
- **Group order:** 370 → 371 → 372 → 373. One step, then stop. Production is LIVE.
- **OWNER ACTION:** reconcile with apply=true on staging, then the instrument; the reseed waits until that reads clean.
- **RULES:** branch before any commit; numbering is continuous.
- **Sequence:** reseed → Settings testing → Day-close → Accounting → Prints → Dashboard → audits.
- **Before production:** blank the vendor bootstrap vars after first login + restart. Held: A4 invoice, thermal receipt + statement.

### Recent steps
- **369 — release-runbook** — DONE (2026-09-03) — the release is a checklist now: an env gate, and a rehearsal on a copy.
- **368 — allocation-walk-source** — DONE (2026-09-03) — a new delivery now runs the walk; an opening balance is not drift.
- **367 — recent-sales-close** — DONE (2026-09-02) — the standing rules, walked over the sales register.

> Older steps in PROGRESS-HISTORY.md
