> ⚠️ **PUBLIC FILE** — no secrets, hostnames, real subdomains, endpoints, or keys. Placeholders only.

# PROGRESS.md — Marham Patti Build Tracker (SHORT)

> SHORT tracker. Full detail in PROGRESS-HISTORY.md (append-only). Every step ends `[CHECKPOINT]`; only stop is `[HUMAN_REQUIRED]`.

## Current Status
- **Project:** Marham Patti (multi-tenant white-label healthcare platform · Ganatra launch 1 Aug 2026)
- **Phase:** 1–69 ✅. Now **PHASE 8 (70–84) — vendor completion + Ganatra launch**; launch-critical first.
- **Last completed build-step:** **75 — subscription-module** — DONE (2026-07-17).
- **Next:** **76 — trial-and-demo** — /specs/76-trial-and-demo.md (companion /specs/70-84-CODEREF.md).
- **Branch state:** through feature/69 merged to staging (live; deploy manual). Phase 8 stacks feature/70-global-foundation onward.

### Recent steps
- **75 — subscription-module** — DONE (2026-07-17) — subscription lifecycle with a guarded state machine, a login access gate for suspended clinics, and a per-tenant billing view.
- **74 — rbac-users-management** — DONE (2026-07-17) — console screens to manage vendor and each tenant's staff and custom roles, with invites/resets, tenant-scoped and audited.
- **73 — rbac-core** — DONE (2026-07-17) — custom roles + permission engine on both sides; clinic and console roles union into a user's effective set, enforced on the server, gated in the UI.

> Older steps in PROGRESS-HISTORY.md.