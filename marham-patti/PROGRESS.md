> ⚠️ **PUBLIC FILE** — no secrets, hostnames, subdomains, endpoints, credentials or keys. Placeholders only.

# PROGRESS.md — Marham Patti Build Tracker (SHORT)

> SHORT tracker; full detail in PROGRESS-HISTORY.md. Keep under 1.5 KB. A completed step is NEVER rebuilt — a dangling "Next" means [HUMAN_REQUIRED], not an earlier step.

## Current Status
- **Project:** Marham Patti — multi-tenant white-label healthcare platform.
- **Phase:** **PHASE 31** — ledger truth, payments and personalization.
- **Last completed:** **311 — names-and-identity**.
- **Next:** specs/312-realtime-and-interaction.md
- **Group order:** 312 → 313 → 314 → 315 → 316. One step, then stop at the checkpoint.
- **SECOND/THIRD attempts — capture before fixing:** 312 §1.
- **Awaiting the owner** — run docs/311-catalogue-strength-report.sql (never edit catalogue data) and docs/310-supplier-advance-verification.sql (never delete a row).
- **Mockups are the latest committed standalone files** — badge/pills/ledger/margin work is in them.
- **304's components are MOUNTED in 315, never rebuilt.**
- **BRANCH RULE:** create and switch to the spec's own branch BEFORE any commit.
- **PRINCIPLE:** when a rule creates a new state, audit every surface rendering the old states.
- **Numbering is continuous — never skip a number.**
- **Sequence after this:** returns retest round (mobile, waterfalls, window) → Customers → Recent Sales → Settings → Day-close → Accounting → Prints → Dashboard → audits.
- **Held for build:** A4 sale invoice; thermal receipt (awaiting its mockup update).
- **Awaiting hardware:** thermal printing — Goojprt PT-210, 18F0/2AF1, paced writes, FFE0 fallback, 58mm.
- **Before production:** VAPID keys; blank VENDOR_BOOTSTRAP_*; fresh secrets; MFA_STAGING_RELAX=false; SEED_DEMO and SCREENSHOT_TOKEN unset.

### Recent steps
- **311 — names-and-identity** — DONE (2026-08-19) — one composer names every item, one component credits every person.
- **310 — ledger-truth** — DONE (2026-08-19) — advance instead of a floored zero, all returns, real counts.
- **309 — returns-mobile-and-copy** — DONE (2026-08-19) — the whole card ticks its box, blank band measured away.

> Older steps in PROGRESS-HISTORY.md
