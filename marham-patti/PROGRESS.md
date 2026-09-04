> ⚠️ **PUBLIC FILE** — no secrets, hostnames, subdomains, endpoints, credentials or keys. Placeholders only.

# PROGRESS.md — Marham Patti Build Tracker (SHORT)

> SHORT tracker; full detail in PROGRESS-HISTORY.md. Keep under 1.5 KB. A completed step is NEVER rebuilt — a dangling "Next" means [HUMAN_REQUIRED], not an earlier step.

## Current Status
- **Project:** Marham Patti — multi-tenant white-label healthcare platform.
- **Phase:** **PHASE 45** — Fix round after the Phase 44 test (385–389). All are FIX specs on `fix/*` branches (`DEPLOY FIX`).
- **Last completed:** **389 — gate-repair-387 (carries 387)** — DONE on `fix/387-customers-fix-round`, ready to merge.
- **Next:** specs/388-rolling-month-window.md
- **Group order:** 388 is the last of the round. One step, then stop at the checkpoint.
- **FOUND IN SOURCE, BUILD FROM THE FINDING:** old `/dashboard` + `/accounts` still in the nav registry; Recent-sales SSE without `withSseHeartbeat`; `GlobalDiscountSection` hook after early return; `belowMinStock` counting zero.
- **THIRD ATTEMPT RULE (387 §1):** "Open the ledger" — capture and name the cause with a line number BEFORE any fix; acceptance is a browser run on staging, never a unit test.
- **NO SECOND STORE OF MONEY:** 387 §2's INFO rows are display only; both instruments must still PASS.
- **Only OTHER requires a note** on stock adjustments; reasons are a tenant list (386 §2).
- **Month window:** open-ended when it ends this month (388) — `Jun – Aug` becomes `Jun – Sept`.
- **BRANCH RULE:** create and switch to the spec's own branch BEFORE any commit (389 is the one exception: it reuses 387's branch).
- **Numbering is continuous — never skip a number.**
- **Sequence after this:** design block (Recent sales, Stock alerts, Customers mobile, Roles/Users, Prints, Notifications, Audit log) → Prints → USB print helper → audits → Lab → Clinic.
- **Production:** live since 1 Sep on `release`; 374–389 promote together after the owner's round is clean; 6+ new migrations ride with it.

### Recent steps
- **389 — gate-repair-387** — DONE (2026-09-04) — four source-substring tests repointed to the text 387 and 386 wrote; carries 387.
- **387 — customers-fix-round** — DONE (2026-09-04) — Open the ledger, stated returns, edit dialog, statement, mobile; merges with 389.
- **386 — inventory-alert-rules** — DONE (2026-09-04) — empty is not low, and the shop keeps its own adjustment reasons.

> Older steps in PROGRESS-HISTORY.md
