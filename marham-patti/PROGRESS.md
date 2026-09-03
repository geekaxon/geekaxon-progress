> ⚠️ **PUBLIC FILE** — no secrets, hosts, endpoints, credentials or keys. Placeholders only.

# PROGRESS.md — Marham Patti Build Tracker (SHORT)

> SHORT tracker; detail in PROGRESS-HISTORY.md. Keep under 1.5 KB. A done step is NEVER rebuilt; a dangling "Next" means [HUMAN_REQUIRED].

## Current Status
- **Project:** Marham Patti — multi-tenant white-label healthcare platform.
- **Phase:** **PHASE 43** — the walk, the runbook, returns and customers close.
- **Last completed:** **371 — customers-desktop-close**.
- **Next:** **372 — customers-mobile-close** — specs/372-customers-mobile-close.md
- **Group order:** 372 → 373. One step, then stop. Production is LIVE.
- **OWNER ACTION:** reconcile with apply=true on staging, then the instrument; the reseed waits on it reading clean.
- **RULES:** branch before any commit; numbering is continuous.
- **Sequence:** reseed → Settings → Day-close → Accounting → Prints → Dashboard → audits.
- **Before production:** blank the vendor bootstrap vars after first login + restart. Held: A4 invoice, thermal receipt.

### Recent steps
- **371 — customers-desktop-close** — DONE (2026-09-03) — a merged customer cannot be edited or take money.
- **370 — returns-date-and-slip** — DONE (2026-09-03) — the month filter rolls with today, approvals ignore it, the slip prints.
- **369 — release-runbook** — DONE (2026-09-03) — the release is a checklist now: an env gate, and a rehearsal on a copy.

> Older steps in PROGRESS-HISTORY.md
