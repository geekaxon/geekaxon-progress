> ⚠️ **PUBLIC FILE** — no secrets, hostnames, subdomains, endpoints, credentials or keys. Placeholders only.

# PROGRESS.md — Marham Patti Build Tracker (SHORT)

> SHORT tracker; full detail in PROGRESS-HISTORY.md. Keep under 1.5 KB. A completed step is NEVER rebuilt — a dangling "Next" means [HUMAN_REQUIRED], not an earlier step.

## Current Status
- **Project:** Marham Patti — multi-tenant white-label healthcare platform.
- **Phase:** **PHASE 38** — realtime rule, allocation proof, returns close-out.
- **Last completed:** **343 — realtime-everywhere**.
- **Next:** specs/344-credit-allocation-proof.md
- **Group order:** 343 → 344 → 345 → 346 → 347. One step, then stop at the checkpoint.
- **STANDING RULE (now in ARCHITECTURE.md §2.14):** every screen subscribes on the day it is built; a new screen fails the suite until it does.
- **344 is a SECOND attempt on MONEY** — evidence first; the guard reproduces the owner's two-invoice case; reconciliation reported per supplier.
- **345 §1 items are SECOND attempts** — git-log why 341's build missed them before rebuilding.
- **346:** approval fires the FROZEN posting; rejection never fires it; void reverses by entries; caps count pending.
- **347 RULE:** a document has ONE renderer — mobile PDF == desktop PDF, compared.
- **BRANCH RULE:** create and switch to the spec's own branch BEFORE any commit.
- **Numbering is continuous — never skip a number.**
- **Sequence after this:** Recent Sales testing → Settings testing → Day-close → Accounting → Prints → Dashboard → audits.
- **Held for build:** A4 sale invoice; thermal receipt.
- **Awaiting hardware:** thermal printing — Goojprt PT-210, paced writes, 58mm.
- **Before production:** VAPID keys; blank bootstrap vars; fresh secrets; staging relax off; demo and screenshot tokens unset.

### Recent steps
- **343 — realtime-everywhere** — DONE (2026-08-26) — every screen subscribes, one reader, and the audit is a test now.
- **342 — batch-pricing-and-expiry-choice** — DONE (2026-08-24) — one price for a shop, and it is asked where expired stock goes.
- **341 — returns-page-to-mockup** — DONE (2026-08-24) — the returns page reads like Purchases, and the cards say a number.

> Older steps in PROGRESS-HISTORY.md
