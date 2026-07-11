# PROGRESS.md — Rihaish

**Current build-step:** 03 — entitlements-engine
**Last completed:** 02 — tenancy-core — DONE (2026-07-11)
**Next:** 03 — entitlements-engine

### Recent steps

- **02 — tenancy-core** — DONE (2026-07-11) — Society/SocietyDomain/AuditLog models + first migration; AsyncLocalStorage scope + Prisma scoping extension (`lib/db.ts`) with auto societyId injection, soft-delete→AuditLog, `db.unscoped()` boundary (ESLint-enforced); host resolution middleware (platform/society 404 isolation); screenshot token (staging+GET); read-only 423 seam; health tenancy self-test. All gates green.
- **01 — foundation** — DONE (2026-07-11) — Next 15 + React 19 + TS strict scaffold; emerald/gold tokens light+dark; next-intl en/ur + RTL; money/time bigint primitives; `<Icon>`; `/api/health`; CI + deploy.sh + pm2; all gates green.

> Full history: PROGRESS-HISTORY.md · Build order: ARCHITECTURE.md
