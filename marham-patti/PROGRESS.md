> ⚠️ **PUBLIC FILE** — no secrets, hostnames, subdomains, endpoints, credentials or keys. Placeholders only.

# PROGRESS.md — Marham Patti Build Tracker (SHORT)

> SHORT tracker; full detail in PROGRESS-HISTORY.md (append-only). Keep under 1.5 KB. Each step ends `[CHECKPOINT]`; only stop is `[HUMAN_REQUIRED]`. A completed step is NEVER rebuilt — an empty or dangling "Next" means [HUMAN_REQUIRED], not an earlier step.

## Current Status
- **Project:** Marham Patti — multi-tenant white-label healthcare platform (Ganatra go-live pending owner approval).
- **Phase:** 1–95 ✅. Now **PHASE 11 (96–97)** — host-aware surface isolation + tenant branding, then vendor UI fixes round 3 (CODEREF 96-97).
- **Last completed:** **96 — host-aware-routing-and-tenant-branding** (2026-07-20) — the host surface governs every path (cross-surface → clean 404), tenant branding on staff auth, relative API base enforced in code.
- **Next:** **97 — vendor-ui-fixes-round-3** — /specs/97-vendor-ui-fixes-round-3.md.
- **Branches:** through feature/95 merged to staging (live on the real domain; deploy manual).

### Recent steps
- **96 — host-aware-routing-and-tenant-branding** — DONE (2026-07-20) — surface boundary on every path, tenant logo/name/palette on staff auth screens, browser API base forced relative in code.
- **95 — ai-command-centre-and-subscription** — DONE (2026-07-19) — AI-key verify + model cascade, disabled keys out of routing, task purposes, auto-renew on confirmed payment, optional 2FA.
- **94 — global-ui-completions** — DONE (2026-07-19) — rows-per-page → searchable select, list view/size/columns persist per list, dashboard activity capped at 10.

> Older steps in PROGRESS-HISTORY.md.
