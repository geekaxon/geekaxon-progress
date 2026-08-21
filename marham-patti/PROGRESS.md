> ⚠️ **PUBLIC FILE** — no secrets, hostnames, subdomains, endpoints, credentials or keys. Placeholders only.

# PROGRESS.md — Marham Patti Build Tracker (SHORT)

> SHORT tracker; full detail in PROGRESS-HISTORY.md. Keep under 1.5 KB. A completed step is NEVER rebuilt — a dangling "Next" means [HUMAN_REQUIRED], not an earlier step.

## Current Status
- **Project:** Marham Patti — multi-tenant white-label healthcare platform.
- **Phase:** **PHASE 33** — payment UX, terms, and colour truth.
- **Last completed:** **323 — split-everywhere-and-payment-ux**.
- **Next:** specs/324-supplier-terms-and-ledger-colour.md
- **Group order:** 324 → 325. One step, then stop at the checkpoint.
- **SECOND attempts — evidence before rebuilding:** 324 §2 (319's ledger colours) · 325 §3 (320's totals colours).
- **324 §1 capture:** record how free-text terms parse into a due date BEFORE changing the field.
- **325 §1:** the Returned row MOUNTS on the invoice summary — 304's component, never a second one.
- **PRINCIPLE:** a summary shows every number in its own arithmetic — the calculator test.
- **PRINCIPLE:** a state never recolours money.
- **BRANCH RULE:** create and switch to the spec's own branch BEFORE any commit.
- **Numbering is continuous — never skip a number.**
- **Sequence after this:** returns retest (mobile, waterfalls, window) → Customers → Recent Sales → Settings → Day-close → Accounting → Prints → Dashboard → audits.
- **Held for build:** A4 sale invoice; thermal receipt (awaiting its mockup update).
- **Awaiting hardware:** thermal printing — Goojprt PT-210, 18F0/2AF1, paced writes, FFE0 fallback, 58mm.
- **Before production:** VAPID keys; blank VENDOR_BOOTSTRAP_*; fresh secrets; MFA_STAGING_RELAX=false; SEED_DEMO and SCREENSHOT_TOKEN unset.

### Recent steps
- **325 — purchase-colour-truth-and-polish** — AUTHORED — invoice Returned row, info-blue removed, totals colours (2nd), realtime New Purchase, pull-to-refresh polish.
- **324 — supplier-terms-and-ledger-colour** — AUTHORED — Terms numeric + migration, due-date autofill, ledger colours (2nd), advance block swap, Refund card, phone warning.
- **323 — split-everywhere-and-payment-ux** — DONE (2026-08-21) — one split rule everywhere, tidier refund tabs, no key names on mobile.

> Older steps in PROGRESS-HISTORY.md
