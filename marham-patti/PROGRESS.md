> ⚠️ **PUBLIC FILE** — no secrets, hostnames, subdomains, endpoints, credentials or keys. Placeholders only.

# PROGRESS.md — Marham Patti Build Tracker (SHORT)

> SHORT tracker; full detail in PROGRESS-HISTORY.md. Keep under 1.5 KB. A completed step is NEVER rebuilt — a dangling "Next" means [HUMAN_REQUIRED], not an earlier step.

## Current Status
- **Project:** Marham Patti — multi-tenant white-label healthcare platform.
- **Phase:** **PHASE 19** — tenant finalisation (174→178) complete. Tenant only; vendor unchanged.
- **Last completed:** **178 — remember-me-and-pwa-pin** — long-lived "remember me"/PWA sessions with a 4-digit POS identity-PIN gate.
- **Next:** none — awaiting next spec block
- **Branches:** through 173 on staging. 178 on feature/178-remember-me-and-pwa-pin.
- **Note:** Inventory waits on the retail-catalog scoping in ARCHITECTURE.

### Recent steps
- **178 — remember-me-and-pwa-pin** — DONE (2026-08-02) — remember-me/PWA long-lived sessions behind a 4-digit POS identity-PIN gate; 5-strike lockout; all events audited.
- **177 — tenant-global-search** — DONE (2026-08-02) — `/` overlay over own products/customers/suppliers; plan+permission gated, RLS-scoped, rate-limited.
- **176 — topbar-to-mockup** — DONE (2026-08-02) — `.ptop` title/subtitle/tag from registry; keyboard/bell/theme/user acts; install + sign-out in usermenu.

> Older steps in PROGRESS-HISTORY.md
