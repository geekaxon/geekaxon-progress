> ⚠️ **PUBLIC FILE** — no secrets, hostnames, endpoints or keys. Placeholders only.

# PROGRESS.md — Marham Patti Build Tracker (SHORT)

> SHORT tracker; detail in PROGRESS-HISTORY.md. Under 1.5 KB. A completed step is NEVER rebuilt — a dangling "Next" means [HUMAN_REQUIRED].

## Current Status
- **Project:** Marham Patti — multi-tenant white-label healthcare platform.
- **Phase:** **PHASE 36** — Customers and the returns close.
- **Last completed:** **336 — sale-return-waterfall**.
- **Next:** specs/337-returns-mobile-ux.md
- **Group order:** 334 → 335 → 336 → 337 → 338 → 339. The customer entity is extended, never duplicated.
- **337 is phone-only UX** — the waterfall's wording is settled; do not reopen how the money splits.
- **PRINCIPLE:** one net balance — udhaar or advance, never both. Colours by computed style.
- **BRANCH RULE:** switch to the spec's own branch BEFORE any commit. Numbering is continuous.
- **After this:** Recent Sales → Settings → Day-close → Accounting → Prints → Dashboard → audits. Held: A4 invoice, thermal receipt.
- **Awaiting hardware:** thermal printing — Goojprt PT-210, 18F0/2AF1, paced writes, FFE0 fallback, 58mm.
- **Before production:** VAPID keys; blank VENDOR_BOOTSTRAP_*; fresh secrets; MFA_STAGING_RELAX=false; SEED_DEMO and SCREENSHOT_TOKEN unset.

### Recent steps
- **336 — sale-return-waterfall** — DONE (2026-08-22) — a refund clears the udhaar first, and the cashier settles the rest.
- **335 — customers-merge-and-statement** — DONE (2026-08-22) — duplicates join into one, and the account prints.
- **334 — customers-ledger** — DONE (2026-08-22) — the customer's book, and an advance can be handed back.

> Older steps in PROGRESS-HISTORY.md
