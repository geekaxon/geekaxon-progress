> ⚠️ **PUBLIC FILE** — no secrets, hostnames, subdomains, endpoints, credentials or keys. Placeholders only.

# PROGRESS.md — Marham Patti Build Tracker (SHORT)

> SHORT tracker; full detail in PROGRESS-HISTORY.md. Keep under 1.5 KB. A completed step is NEVER rebuilt — a dangling "Next" means [HUMAN_REQUIRED], not an earlier step.

## Current Status
- **Project:** Marham Patti — multi-tenant white-label healthcare platform.
- **Phase:** **PHASE 38** — realtime rule, allocation proof, returns close-out.
- **Last completed:** **344 — credit-allocation-proof**.
- **Next:** specs/345-returns-page-close.md
- **Group order:** 343 → 344 → 345 → 346 → 347. One step, then stop.
- **RULE:** every screen subscribes on the day it is built (ARCHITECTURE.md §2.14).
- **RULE:** anything stating one invoice's balance reads that supplier's whole book; only a list page stays a page.
- **345 §1 items are SECOND attempts** — git-log why 341 missed them first.
- **346:** approval fires the FROZEN posting; rejection never does; void reverses by entries.
- **347 RULE:** a document has ONE renderer — mobile PDF == desktop PDF, compared.
- **BRANCH RULE:** switch to the spec's own branch BEFORE any commit.
- **Numbering is continuous — never skip a number.**
- **Then:** Recent Sales → Settings → Day-close → Accounting → Prints → Dashboard → audits.
- **Held:** A4 sale invoice; thermal receipt (Goojprt PT-210, 58mm, paced writes).
- **Before production:** VAPID keys; fresh secrets; staging relax off; demo and screenshot tokens unset.

### Recent steps
- **344 — credit-allocation-proof** — DONE (2026-08-26) — the credit walk read a page, not a book; the older invoice moves now.
- **343 — realtime-everywhere** — DONE (2026-08-26) — every screen subscribes, one reader, and the audit is a test now.
- **342 — batch-pricing-and-expiry-choice** — DONE (2026-08-24) — one price for a shop, and it is asked where expired stock goes.

> Older steps in PROGRESS-HISTORY.md
