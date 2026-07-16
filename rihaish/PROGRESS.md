# PROGRESS.md — Rihaish

**Current build-step:** 79 — p2-cleanup — DONE (2026-07-16)
**Last completed:** 79 — p2-cleanup — DONE (2026-07-16)
**Next:** 80 (see ARCHITECTURE.md build order)

### Recent steps

- **77 — deploy-gate** — DONE (2026-07-16) — Made the staging deploy wait for CI to finish green and blocked production from shipping against an un-green staging build, so a red test can no longer reach a server.
- **78 — webhook-idempotency** — DONE (2026-07-16) — Made the SafePay webhook credit a society exactly once even under a retry or a race, so a repeated payment notification can no longer double-charge.
- **79 — p2-cleanup** — DONE (2026-07-16) — Cleared six confirmed review defects: dashboard revenue now reads as real currency, meter bills compute exactly, one two-factor sign-up screen, and CI runs only the critical checks fast.

> Full history: PROGRESS-HISTORY.md · Build order: ARCHITECTURE.md
</content>
</invoke>
