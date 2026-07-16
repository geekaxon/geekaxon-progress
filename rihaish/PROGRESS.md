# PROGRESS.md — Rihaish

**Current build-step:** 78 — webhook-idempotency — DONE (2026-07-16)
**Last completed:** 78 — webhook-idempotency — DONE (2026-07-16)
**Next:** 79 (see ARCHITECTURE.md build order)

### Recent steps

- **76 — pricing-consolidation** — DONE (2026-07-16) — Collapsed three divergent pricing tables into one, so the website, quote, plan builder and invoice all bill the same number for a given society.
- **77 — deploy-gate** — DONE (2026-07-16) — Made the staging deploy wait for CI to finish green and blocked production from shipping against an un-green staging build, so a red test can no longer reach a server.
- **78 — webhook-idempotency** — DONE (2026-07-16) — Made the SafePay webhook credit a society exactly once even under a retry or a race, so a repeated payment notification can no longer double-charge.

> Full history: PROGRESS-HISTORY.md · Build order: ARCHITECTURE.md
