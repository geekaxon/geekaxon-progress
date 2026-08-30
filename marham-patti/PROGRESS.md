> ⚠️ **PUBLIC FILE** — no secrets, hostnames, subdomains, endpoints, credentials or keys. Placeholders only.

# PROGRESS.md — Marham Patti Build Tracker (SHORT)

> SHORT tracker; full detail in PROGRESS-HISTORY.md. Keep under 1.5 KB. A completed step is NEVER rebuilt — a dangling "Next" means [HUMAN_REQUIRED], not an earlier step.

## Current Status
- **Project:** Marham Patti — multi-tenant white-label healthcare platform.
- **Phase:** **PHASE 41** — allocation proof, the final drawfacts, and the reseed.
- **Last completed:** **357 — allocation-itemized-and-verified**.
- **Next:** specs/358-drawfacts-final-and-formats.md
- **Group order:** 358 → 359 → 360. One step, then stop at the checkpoint.
- **357's CHECK IS A SCRIPT, UNRUN:** no database is reachable from the build agent, so the over-allocation check ships read-only and exits non-zero; run it before trusting the figures.
- **358 §1 is the FIFTH report, resolved by MOUNTING:** delete .drawfacts; the return drawer mounts the Purchase drawer's facts component. No more styling.
- **360 is DESTRUCTIVE AND GATED:** the reseed script merges UNRUN; the owner runs it after reading the report; production structurally unreachable; masters kept.
- **RULE:** every derived figure lists its constituents — the calculator test extends to history.
- **RULE:** the pulse fires only for events received in the open session.
- **RULE:** every data column sorts; ACTIONS never does.
- **BRANCH RULE:** create and switch to the spec's own branch BEFORE any commit.
- **Numbering is continuous — never skip a number.**
- **Sequence after this:** Recent Sales testing → Settings testing → Day-close → Accounting → Prints → Dashboard → audits.
- **Held for build:** A4 sale invoice; thermal receipt (includes 326's line arithmetic).
- **Awaiting hardware:** thermal printing — Goojprt PT-210, 18F0/2AF1, paced writes, FFE0 fallback, 58mm.
- **Before production:** VAPID keys; blank VENDOR_BOOTSTRAP_*; fresh secrets; MFA_STAGING_RELAX=false; SEED_DEMO and SCREENSHOT_TOKEN unset; reseed staging guard re-verified.

### Recent steps
- **360 — staging-reseed** — AUTHORED — gated wipe+seed, masters kept, invariants printed.
- **359 — pulse-sort-and-print-parity** — AUTHORED — session-only pulse, sort-everywhere rule, draft conditions, CN spacing to invoice.
- **358 — drawfacts-final-and-formats** — AUTHORED — mount the purchase facts component; phone format app-wide; owner one-liners.
- **357 — allocation-itemized-and-verified** — DONE (2026-08-30) — each applied payment on its own line, dated and clickable to the ledger.

> Older steps in PROGRESS-HISTORY.md
