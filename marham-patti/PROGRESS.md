> ⚠️ **PUBLIC FILE** — no secrets, hostnames, subdomains, endpoints, credentials or keys. Placeholders only.

# PROGRESS.md — Marham Patti Build Tracker (SHORT)

> SHORT tracker; full detail in PROGRESS-HISTORY.md. Keep under 1.5 KB. A completed step is NEVER rebuilt — a dangling "Next" means [HUMAN_REQUIRED], not an earlier step.

## Current Status
- **Project:** Marham Patti — multi-tenant white-label healthcare platform.
- **Phase:** Vendor signed off at 151. **PHASE 19** — tenant finalisation against the mockups. Group 152→154: schema and deploy hardening, the English-only sweep, then the POS visual match. Standing rules: tenant app English only; identity via @mp/brand; business logic frozen; Phase 12 suites pass unchanged; verification is visual.
- **Last completed:** **153 — tenant-english-only-sweep**.
- **Next:** **154 — pos-visual-match** — /specs/154-pos-visual-match.md
- **Branches:** through 152 on staging. 153 on fix/153-tenant-english-only-sweep.
- **Note:** spec 137 is retired — superseded by 154. Never build it.

### Recent steps
- **153 — tenant-english-only-sweep** — DONE (2026-07-30) — staff screens resolve English through the framework; no dir flip; lint guard bites.
- **152 — schema-and-deploy-hardening** — DONE (2026-07-30) — consent enum-type migration, deploy drift gate, --container token.
- **151 — auth-routing-and-avatar** — DONE (2026-07-29) — server-side auth redirects, avatar object-fit cover.

> Older steps in PROGRESS-HISTORY.md
