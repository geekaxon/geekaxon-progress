# PROGRESS.md — Rihaish

**Current build-step:** 04 — auth-rbac
**Last completed:** 03 — entitlements-engine — DONE (2026-07-11)
**Next:** 04 — auth-rbac

### Recent steps

- **03 — entitlements-engine** — DONE (2026-07-11) — Feature/Plan/SocietyEntitlement/SocietyPlan models + migration; feature registry (`lib/features.ts`, core.foundation/tenancy/entitlements) validated (missing-edge + cycle) at load; pure DAG (`lib/feature-dag.ts`) + resolver math (`lib/entitlements-compute.ts`: plan∪overrides−disabled−expired+core, enable/disable planning, limits); `lib/entitlements.ts` cached resolver + `requireFeature` (403) + enable/disable (atomic, DAG-guarded, cascade, audit) + registry seeder; probe route `/api/entitlements/check`. All gates green (82 unit tests).
- **02 — tenancy-core** — DONE (2026-07-11) — Society/SocietyDomain/AuditLog models + first migration; AsyncLocalStorage scope + Prisma scoping extension (`lib/db.ts`) with auto societyId injection, soft-delete→AuditLog, `db.unscoped()` boundary (ESLint-enforced); host resolution middleware (platform/society 404 isolation); screenshot token (staging+GET); read-only 423 seam; health tenancy self-test. All gates green.
- **01 — foundation** — DONE (2026-07-11) — Next 15 + React 19 + TS strict scaffold; emerald/gold tokens light+dark; next-intl en/ur + RTL; money/time bigint primitives; `<Icon>`; `/api/health`; CI + deploy.sh + pm2; all gates green.

> Full history: PROGRESS-HISTORY.md · Build order: ARCHITECTURE.md
