# PROGRESS.md — Rihaish

**Current build-step:** 70 — design-token bridge — DONE (2026-07-16)
**Last completed:** 70 — design-token bridge — DONE (2026-07-16)
**Next:** 71 (see ARCHITECTURE.md build order)

### Recent steps

- **68 — ci-capacity** — DONE (2026-07-14) — Split CI so the cheap gates stay on GitHub and the heavy tests run free on our own box, guarded so they can never touch live data.
- **69 — locale-single-source** — DONE (2026-07-14) — The supported locale list lives in one file; everyone imports it, and a guard test fails the build on any second copy.
- **70 — design-token bridge** — DONE (2026-07-16) — Adopted the new design system's colours to drive every existing component through one token layer, plus a guard that blocks ad-hoc colours.

> Full history: PROGRESS-HISTORY.md · Build order: ARCHITECTURE.md
