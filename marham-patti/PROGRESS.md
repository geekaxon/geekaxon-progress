> ⚠️ **PUBLIC FILE** — no secrets, hostnames, subdomains, endpoints, credentials or keys. Placeholders only.

# PROGRESS.md — Marham Patti Build Tracker (SHORT)

> SHORT tracker; full detail in PROGRESS-HISTORY.md. Keep under 1.5 KB. A completed step is NEVER rebuilt — a dangling "Next" means [HUMAN_REQUIRED], not an earlier step.

## Current Status
- **Project:** Marham Patti — multi-tenant white-label healthcare platform.
- **Phase:** **PHASE 21** — production LIVE; Purchases and Suppliers.
- **Last completed:** **239 — dialog-close-and-amount-fields**.
- **Next:** specs/240-purchases-suppliers-desktop.md
- **Group order:** 240 → 241. Build one step only, then stop at the checkpoint.
- **Numbering is continuous — never skip a number.**
- **Production note:** deploys change CODE only; no operational data written on boot. Fix on staging, verify, then promote.
- **Sequence after this round:** thermal printing (raw ESC/POS) → Settings finalisation → Customers, Returns, Day-close, Prints.
- **Parked:** thermal printing transport; per-page menu permissions; deferred shortcut bindings.

### Recent steps
- **241 — purchases-suppliers-mobile** — AUTHORED — card lists, per-line entry cards, ledger sheet, inherited sheet behaviour.
- **240 — purchases-suppliers-desktop** — AUTHORED — lists, unit-aware entry with conversion line, supplier ledger, A4 invoice.
- **239 — dialog-close-and-amount-fields** — DONE (2026-08-10) — one dismissal for every ✕; money fields take decimals and step on the arrows.

> Older steps in PROGRESS-HISTORY.md
