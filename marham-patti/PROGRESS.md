> ⚠️ **PUBLIC FILE** — no secrets, hostnames, subdomains, endpoints, credentials or keys. Placeholders only.

# PROGRESS.md — Marham Patti Build Tracker (SHORT)

> SHORT tracker; full detail in PROGRESS-HISTORY.md. Keep under 1.5 KB. A completed step is NEVER rebuilt — a dangling "Next" means [HUMAN_REQUIRED], not an earlier step.

## Current Status
- **Project:** Marham Patti — multi-tenant white-label healthcare platform.
- **Phase:** **PHASE 37** — purchase payment polish, returns to mockup, batch pricing.
- **Last completed:** **341 — returns-page-to-mockup**.
- **Next:** specs/342-batch-pricing-and-expiry-choice.md
- **342:** Single price changes what a customer is charged; one resolver, one branch; default stays Per batch, so a split still splits.
- **341 left a migration to run on upgrade** — the return-approval grant, per tenant; dry-run first.
- **PRINCIPLE:** FEFO is a safety rule, batch pricing a commercial choice — not one setting; a default is a guess unless the tenant was asked.
- **BRANCH RULE:** create and switch to the spec's own branch BEFORE any commit.
- **Numbering is continuous — never skip a number.**
- **Sequence after this:** Recent Sales testing → Settings → Day-close → Accounting → Prints → Dashboard → audits.
- **Held for build:** A4 sale invoice; thermal receipt (includes 326's line arithmetic).
- **Awaiting hardware:** thermal printing — Goojprt PT-210, paced writes, FFE0 fallback, 58mm.
- **Before production:** VAPID keys; blank VENDOR_BOOTSTRAP_*; fresh secrets; MFA_STAGING_RELAX=false; SEED_DEMO and SCREENSHOT_TOKEN unset.

### Recent steps
- **342 — batch-pricing-and-expiry-choice** — AUTHORED — Per batch vs Single price; expired handling asked once; quarantine view.
- **341 — returns-page-to-mockup** — DONE (2026-08-24) — Month filter and shared icons, cards that compute, drawer money, approval as a permission.
- **340 — new-purchase-payment-polish** — DONE (2026-08-24) — Udhaar/Exact pills off the total, one-line account, udhaar shown, mobile Save PDF.

> Older steps in PROGRESS-HISTORY.md
