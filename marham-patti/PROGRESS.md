> ⚠️ **PUBLIC FILE** — pushed to a separate public progress repo. **NO secrets, hostnames, real subdomains, API endpoints, credentials, or keys — ever.** Placeholders only.

# PROGRESS.md — Marham Patti

**Current build-step:** Phase 6 complete (63 done). Phase 7 authored (64–69).
**Last completed:** 63 — design-system-v3 — DONE (2026-07-14)
**Next-step pointer:** → **64 — `specs/64-vendor-redesign-foundation.md`** (companion `specs/64-69-CODEREF.md`; visual target `specs/mockups/`). Build 64→69 in order, one `[CHECKPOINT]` each.

**Branch state:** `feat/01`…`feat/39` + `feature/40`…`feature/63` merged to `staging` (deploy green; billing+privacy migrations applied; demo seed idempotent). Phase 7 stacks `feature/64-vendor-redesign-foundation` onward.

### Recent steps

- **fix — demo seed cross-day token re-run** ✅ (2026-07-15) — floating demo dates drift the daily queue-number key by a day on a later run; the seed now clears its own tokens and rebuilds fresh, with a cross-day test.
- **fix — demo seed token re-run** ✅ (2026-07-15) — second demo seed hit P2002 on the daily queue-number; now upsert tokens on the real per-doctor-per-day composite, with a twice-run test that seeds a clashing app-style token.
- **63 — design-system-v3** ✅ (2026-07-14) — one control anatomy, password show/hide on every login, logins on a neutral page, drift guard in the lint gate.

> Full history: PROGRESS-HISTORY.md · Build order: ARCHITECTURE.md