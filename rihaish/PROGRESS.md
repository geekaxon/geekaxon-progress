# PROGRESS.md — Rihaish

**Current build-step:** 77 — deploy-gate — DONE (2026-07-16)
**Last completed:** 77 — deploy-gate — DONE (2026-07-16)
**Next:** 78 (see ARCHITECTURE.md build order)

### Recent steps

- **75 — public-site-redesign** — DONE (2026-07-16) — Pulled the fabricated Rufi testimonial off the marketing site, replaced it with an honest value-prop block, and added a guard test so it can never return.
- **76 — pricing-consolidation** — DONE (2026-07-16) — Collapsed three divergent pricing tables into one, so the website, quote, plan builder and invoice all bill the same number for a given society.
- **77 — deploy-gate** — DONE (2026-07-16) — Made the staging deploy wait for CI to finish green and blocked production from shipping against an un-green staging build, so a red test can no longer reach a server.

> Full history: PROGRESS-HISTORY.md · Build order: ARCHITECTURE.md
