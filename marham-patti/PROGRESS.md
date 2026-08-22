> ⚠️ **PUBLIC FILE** — no secrets, hostnames, endpoints or keys. Placeholders only.

# PROGRESS.md — Marham Patti Build Tracker (SHORT)

> SHORT tracker; detail in PROGRESS-HISTORY.md. Under 1.5 KB. A completed step is NEVER rebuilt — a dangling "Next" means [HUMAN_REQUIRED].

## Current Status
- **Project:** Marham Patti — multi-tenant white-label healthcare platform.
- **Phase:** **PHASE 36** — Customers and the returns close.
- **Last completed:** **337 — returns-mobile-ux**.
- **Next:** specs/338-returns-scope-and-copy.md
- **Group order:** 338 → 339. **338 is scope and copy** — how the money splits is settled.
- **PRINCIPLE:** one net balance — udhaar or advance, never both; colours by computed style. The customer is extended, never duplicated.
- **BRANCH RULE:** switch to the spec's own branch BEFORE any commit. Numbering is continuous.
- **After this:** Recent Sales → Settings → Day-close → Accounting → Prints → Dashboard → audits. Held: A4 invoice, receipt.
- **Awaiting hardware:** thermal printing — Goojprt PT-210, 18F0/2AF1, paced writes, FFE0 fallback, 58mm.
- **Before production:** VAPID keys; blank VENDOR_BOOTSTRAP_*; fresh secrets; MFA_STAGING_RELAX=false; SEED_DEMO and SCREENSHOT_TOKEN unset.

### Recent steps
- **337 — returns-mobile-ux** — DONE (2026-08-22) — back walks the steps, and the phone has one picker.
- **336 — sale-return-waterfall** — DONE (2026-08-22) — a refund clears the udhaar first, the cashier settles the rest.
- **335 — customers-merge-and-statement** — DONE (2026-08-22) — duplicates join into one, and the account prints.

> Older steps in PROGRESS-HISTORY.md
