# PROGRESS.md — Rihaish

**Current build-step:** 68 — ci-capacity — DONE (2026-07-14)
**Last completed:** 68 — ci-capacity — DONE (2026-07-14)
**Next:** (see ARCHITECTURE.md build order)

### Recent steps

- **68 — ci-capacity** — DONE (2026-07-14) — Split CI so the cheap gates stay on GitHub and the heavy tests run free on our own box, guarded so they can never touch live data.
- **67 — brand-asset-deployment** — DONE (2026-07-14) — Confirmed the committed brand pack is in place and added a test that fetches each asset and asserts a live 200 with the right type.
- **69 — locale-single-source** — DONE (2026-07-14) — The supported locale list lives in one file; everyone imports it, and a guard test fails the build on any second copy.

> Full history: PROGRESS-HISTORY.md · Build order: ARCHITECTURE.md
