> ⚠️ **PUBLIC FILE** — no secrets, hostnames, subdomains, endpoints, credentials or keys. Placeholders only.

# PROGRESS.md — Marham Patti Build Tracker (SHORT)

> SHORT tracker; full detail in PROGRESS-HISTORY.md. Keep under 1.5 KB. A completed step is NEVER rebuilt — a dangling "Next" means [HUMAN_REQUIRED], not an earlier step.

## Current Status
- **Project:** Marham Patti — multi-tenant white-label healthcare platform.
- **Phase:** **PHASE 37** — purchase payment polish, returns to mockup, batch pricing.
- **Last completed:** **340 — new-purchase-payment-polish**.
- **Next:** specs/341-returns-page-to-mockup.md
- **Group order:** 341 → 342. One step, then stop at the checkpoint.
- **342 changes what a customer is charged for tenants who choose Single price** — resolution orders stay locked; one resolver, one branch; FEFO's physical rule untouched.
- **342 default is Per batch** — existing tenants behave exactly as before; verify a split still splits.
- **341 §2:** capture WHY each KPI card returned a dash before fixing; check the same aggregate on Purchases and Suppliers.
- **341 §4 migration:** every tenant's approval behaviour unchanged on upgrade, reported per tenant.
- **PRINCIPLE:** FEFO is a safety rule about stock; batch pricing is a commercial choice — they are not one setting.
- **PRINCIPLE:** a default is a guess unless the tenant was asked.
- **BRANCH RULE:** create and switch to the spec's own branch BEFORE any commit.
- **Numbering is continuous — never skip a number.**
- **Sequence after this:** Recent Sales testing (mount 304's components on sales) → Settings testing → Day-close → Accounting → Prints → Dashboard → audits.
- **Held for build:** A4 sale invoice; thermal receipt (includes 326's line arithmetic).
- **Awaiting hardware:** thermal printing — Goojprt PT-210, 18F0/2AF1, paced writes, FFE0 fallback, 58mm.
- **Before production:** VAPID keys; blank VENDOR_BOOTSTRAP_*; fresh secrets; MFA_STAGING_RELAX=false; SEED_DEMO and SCREENSHOT_TOKEN unset.

### Recent steps
- **342 — batch-pricing-and-expiry-choice** — AUTHORED — Per batch vs Single price; expired handling asked once; quarantine view.
- **341 — returns-page-to-mockup** — AUTHORED — Purchases pattern, KPI cards compute, drawer money consequence, approval permission.
- **340 — new-purchase-payment-polish** — DONE (2026-08-24) — Udhaar/Exact pills off the total, one-line account, udhaar shown, mobile Save PDF.

> Older steps in PROGRESS-HISTORY.md
