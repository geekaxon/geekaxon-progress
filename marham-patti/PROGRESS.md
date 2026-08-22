> ⚠️ **PUBLIC FILE** — no secrets, hostnames, subdomains, endpoints, credentials or keys. Placeholders only.

# PROGRESS.md — Marham Patti Build Tracker (SHORT)

> SHORT tracker; full detail in PROGRESS-HISTORY.md. Keep under 1.5 KB. A completed step is NEVER rebuilt — a dangling "Next" means [HUMAN_REQUIRED], not an earlier step.

## Current Status
- **Project:** Marham Patti — multi-tenant white-label healthcare platform.
- **Phase:** **PHASE 36** — Customers and the returns close.
- **Last completed:** **332 — new-purchase-payment-behaviour**.
- **Next:** specs/333-customers-core.md
- **Group order:** 333 → 334 → 335 → 336 → 337 → 338 → 339. 334 before 336; a customer entity exists — extend it, never duplicate.
- **336 moves money** — one-net-balance guard every case; re-read the supplier desk's advance draw-down (332 dropped it on purchases).
- **PRINCIPLE:** one net balance — udhaar or advance, never both. The payment box passes the calculator test in every state. Colours by computed style.
- **BRANCH RULE:** create and switch to the spec's own branch BEFORE any commit. Numbering is continuous.
- **Sequence after this:** Recent Sales → Settings → Day-close → Accounting → Prints → Dashboard → audits. Held: A4 invoice, thermal receipt.
- **Awaiting hardware:** thermal printing — Goojprt PT-210, 18F0/2AF1, paced writes, FFE0 fallback, 58mm.
- **Before production:** VAPID keys; blank VENDOR_BOOTSTRAP_*; fresh secrets; MFA_STAGING_RELAX=false; SEED_DEMO and SCREENSHOT_TOKEN unset.

### Recent steps
- **332 — new-purchase-payment-behaviour** — DONE (2026-08-22) — the payment box fills itself, caps itself and says where a difference goes.
- **331 — polish-and-new-purchase-payment** — DONE.
- **330 — ledger-colour-source** — DONE.

> Older steps in PROGRESS-HISTORY.md
