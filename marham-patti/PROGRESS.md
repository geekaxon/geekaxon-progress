> ⚠️ **PUBLIC FILE** — no secrets, hostnames, subdomains, endpoints, credentials or keys. Placeholders only.

# PROGRESS.md — Marham Patti Build Tracker (SHORT)

> SHORT tracker; detail in PROGRESS-HISTORY.md. A completed step is NEVER rebuilt — a dangling "Next" means [HUMAN_REQUIRED], not an earlier step.

## Current Status
- **Project:** Marham Patti — multi-tenant white-label healthcare platform.
- **Phase:** **PHASE 34** — pricing on screen, ledger colour close, New Purchase payment.
- **Last completed:** **326 — pos-inline-pricing**.
- **Next:** specs/327-inventory-customer-pays.md
- **Group order:** 327 → 328 → 329. One step, then stop at the checkpoint.
- **328 verifies FIRST:** the owner's report predates 323–325's deploy; a 324-era item still broken is a third attempt, with evidence.
- **329 touches ONLY the payment section** — the rest of the page is context.
- **PRINCIPLE:** direction on amounts, white on balances, success on advances.
- **PRINCIPLE:** money cannot flow from a bucket into itself.
- **BRANCH RULE:** create and switch to the spec's own branch BEFORE any commit.
- **Numbering is continuous — never skip a number.**
- **After this:** returns retest → Customers → Recent Sales → Settings → Day-close → Accounting → Prints → Dashboard.
- **Held for build:** A4 sale invoice; thermal receipt.
- **Awaiting hardware:** thermal — Goojprt PT-210, 18F0/2AF1, paced writes, FFE0 fallback, 58mm.
- **Before production:** VAPID keys; blank bootstrap vars; fresh secrets; staging MFA relax off; demo seed and screenshot token unset.

### Recent steps
- **326 — pos-inline-pricing** — DONE (2026-08-21) — the price a customer pays, shown and printed with its working.
- **325 — purchase-colour-truth-and-polish** — DONE (2026-08-19) — the printed invoice adds up; no state ink on money.
- **324 — supplier-terms-and-due-date** — DONE (2026-08-18) — terms in days, a due date the desk owns.

> Older steps in PROGRESS-HISTORY.md
