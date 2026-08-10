> ⚠️ **PUBLIC FILE** — no secrets, hostnames, subdomains, endpoints, credentials or keys. Placeholders only.

# PROGRESS.md — Marham Patti Build Tracker (SHORT)

> SHORT tracker; detail in PROGRESS-HISTORY.md. Under 1.5 KB. A completed step is NEVER rebuilt — a dangling "Next" means [HUMAN_REQUIRED], not an earlier step.

## Current Status
- **Project:** Marham Patti — multi-tenant white-label healthcare platform.
- **Phase:** **PHASE 21** — production LIVE; Purchases and Suppliers testing.
- **Last completed:** **246 — demo-seed-repair-and-repeat-failures**.
- **Next:** specs/247-suppliers-card-view-and-desktop-r3.md
- **Group order:** 246 → 247 → 248. One step only, then stop at the checkpoint.
- **BRANCH RULE:** switch to the spec's own branch BEFORE any commit.
- **VERIFICATION RULE:** a fix is accepted on the DEPLOYED page, never on a unit test.
- **Numbering is continuous — never skip a number.**
- **Production note:** deploys change CODE only; no operational data on boot.
- **Sequence after this round:** thermal printing → shortcuts → consistency audit → Customers → Returns → Recent Sales → Settings → Day-close → Prints → Dashboard.
- **Parked:** purchases import; server-side export; per-page menu permissions.

### Recent steps
- **248 — purchases-suppliers-mobile-r3** — AUTHORED — chrome overlap, .mfiltrow/.segctl row, full-file diff.
- **247 — suppliers-card-view-and-desktop-r3** — AUTHORED — new .scard card view, full-file diff.
- **246 — demo-seed-repair-and-repeat-failures** — DONE (2026-08-10) — seed numbers from the till counter; band, retry, picker, import.

> Older steps in PROGRESS-HISTORY.md
