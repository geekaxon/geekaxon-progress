> ⚠️ **PUBLIC FILE** — pushed to the public progress repo (`projects-abovenext/marham-patti-progress`). **NO secrets, hostnames, real subdomains, API endpoints, credentials, or keys — ever.** Placeholders only.

# PROGRESS-HISTORY.md — Marham Patti (append-only archive)

> **This is the COMPLETE, append-only archive** of every finished build-step, newest at the top. The agent appends the FULL entry here as part of finishing each step (see PROGRESS.md recording protocol). Read this file **only when recovering an older step's detail** — never cover-to-cover. The SHORT rolling tracker is **PROGRESS.md**.
>
> **Entry format (per step):** `## Build-step NN — <title>` then: work-type (FEATURE/FIX), phase label, branch, dependency note, schema/migration/RLS/auth/permission/**feature-flag** impact, problem, what changed (data model, endpoints, UI Simple/Pro, **flags exposed**, i18n keys), and ALL verification-gate numbers (typecheck, lint, turbo build, jest `N/N (S suites; +X)`, i18n parity, isolation, **flag/vertical smoke**, **performance/Lighthouse**, white-label).
>
> **Numbering:** canonical per ARCHITECTURE.md §14 (continuous 1→N; phase = label).

---

## Build-step 01 — Foundation: Monorepo + DB + Prisma base + health endpoint + deploy.sh + CI

- **Work-type:** FEATURE (greenfield platform base). **Phase:** Foundation (item 1 of 39). **Branch:** `feat/01-foundation-monorepo` (off the initial commit; controller merges to `staging` at checkpoint).
- **Depends on:** nothing (first step). **Date:** 2026-07-03.
- **Schema/migration:** empty Prisma baseline (`packages/db/prisma/schema.prisma` — datasource postgresql + generator only, NO business models). Baseline migration `00000000000000_baseline` (no-op `SELECT 1;`) + `migration_lock.toml (postgresql)`. **RLS:** ➖ (no business tables — begins in 02). **Auth/permission:** ➖ (begins 03/04). **Feature-flags:** ➖ (engine scaffolded only; real engine in 05).

### Problem
There was no codebase. Every later step assumes a pnpm/Turborepo monorepo, Prisma+Postgres, a health endpoint, the `@mp/*` shared packages (incl. `@mp/flags`, `@mp/brand`), a working `deploy.sh` (with the learned hard rules + Lighthouse-gate hook), and CI. Built once, correctly.

### What changed
- **Monorepo:** pnpm workspace (`pnpm-workspace.yaml`) + Turborepo (`turbo.json`: build/typecheck/lint/test/dev; `^build` ordering so packages build before apps). Root `tsconfig.base.json` (CommonJS/node to avoid ESM↔CJS friction with NestJS decorators), flat ESLint (`eslint.config.mjs`, lenient at foundation), Prettier, `.gitignore`, `.npmrc`, `.env.example`.
- **Packages (`packages/*`):**
  - `@mp/config` — zod-validated env; **lazy** `loadConfig()` (throws on boot if invalid, never on import so build/typecheck don't trip); `isStaging()` fail-closed helper; vars incl. `APP_ENV`.
  - `@mp/db` — shared Prisma singleton (hot-reload-guarded) + **frozen `runWithTenant(tenantId, fn)` seam** (placeholder; real `SET LOCAL app.tenant_id` RLS wiring lands in 02) + `disconnect()`.
  - `@mp/shared` — `Result`/`ok`/`err`, `nowIso`, `isNonEmptyString`, `APP_NAME`.
  - `@mp/flags` — capability-engine SCAFFOLD: `FlagKey`, five `VERTICAL_PRESETS`, `CapabilityContext`, `isFeatureEnabled` (fail-closed). Real engine 05.
  - `@mp/brand` — white-label SCAFFOLD: canonical DEFAULT theme = ARCHITECTURE.md §11 tokens (Brand Teal `#06888D` … Alert `#B4453C`), identity (name/logo/sender), fonts (display/body/urdu), `brandCssVars()`/`brandCssVarString()`.
  - `@mp/ui` — design-system SCAFFOLD: `cn()`, `themeCssVars()` re-export. Real shadcn/ui system 13.
  - `@mp/ai` — provider-agnostic gateway SCAFFOLD: `createAiGateway` (`isConfigured()=false`), `AI_ASSIST_ONLY` guarantee. Real suite 35–39.
- **Apps (`apps/*`):**
  - `api` — NestJS 11; `GET /health` → `{status:'ok', buildId, ts}` mounted at ROOT (**no `/api` prefix**; verified `/api/health` 404s); `main.ts` validates env via `loadConfig` on boot; Jest (`ts-jest`) health spec.
  - `web` — Next.js 15 App Router PWA; tokenized placeholder home rendered entirely from `@mp/brand` tokens (white-label honored, no hardcoded brand); `manifest.webmanifest` + `sw.js` app-shell + client SW registration; `icon.svg`; brand CSS vars injected on `<html>`. Standalone tsconfig (bundler/esnext/jsx-preserve); committed 2-line `next-env.d.ts`.
  - `worker` — placeholder job runner (config-validate + heartbeat + graceful SIGTERM/SIGINT).
- **Deploy/infra:** `deploy.sh` per §15 — Node detection (nvm → aaPanel `/www/server/nodejs/*` → system), corepack/pnpm, source server `./.env`, pull/reset existing clone (never clone, **branch fence blocks main/master/production/prod** + dangerous-arg fence), `pnpm install --prod=false --frozen-lockfile`, `prisma generate` → `migrate deploy`, fresh `BUILD_ID`, `turbo build`, **`run_lighthouse_gate()` placeholder hook** (real gate 13), **`pm2 restart` (not reload)** via `ecosystem.config.js` (`mp-api`/`mp-web`/`mp-worker`), graceful `/health` retry loop. `ecosystem.config.js` PM2 defs. `.github/workflows/ci.yml` mirrors deploy ordering (install → prisma generate → typecheck → lint → build → test) + Lighthouse stub job. Existing `deploy-staging.yml`/`deploy-production.yml` left intact.

### Verification gates
1. **typecheck:** green — 16/16 turbo tasks. Clean-clone web typecheck (no `.next`) verified exit 0 with committed 2-line `next-env.d.ts`.
2. **lint:** clean — 0 errors, 0 warnings (removed stray `eslint-disable` directives).
3. **turbo build:** green — 10/10 (web `next build` static; api/worker/packages `tsc`).
4. **jest:** 2/2 (1 suite; +2 new) — HealthController.
5. **i18n:** ➖ (no user-facing UI strings yet; two-tier i18n base lands in 08).
6. **Isolation/RLS:** ➖ (no business tables; `runWithTenant` seam frozen for 02).
7. **Feature-flag / vertical smoke:** ➖ (no gated capability yet; flag engine scaffolded, real in 05).
8. **Performance/Lighthouse:** ➖ (no real public surface yet; placeholder web First Load JS ~102 kB. Perf foundation + Lighthouse gate land in 13).
9. **Design self-check:** placeholder is tokenized (brand palette swatches, serif display, light/dark via `color-scheme`), not raw HTML; full design system 13.
10. **White-label:** ✓ web reads brand tokens/identity from `@mp/brand`; no hardcoded logo/color/name.
11. **Accountability:** ➖ (no state changes yet; AuditLog begins 06).
12. **Offline:** SW shell installs + network-first navigation fallback (real offline core 10).
13. **.env.example:** ✓ every var a placeholder incl. `APP_ENV`; **no secrets/real hosts**.

### Notes / manual-run caveats
- `prisma migrate deploy` and a full end-to-end `deploy.sh` run require a live Postgres 16 + the aaPanel server; not runnable in this build sandbox (no Postgres/PM2). Schema validated (`prisma validate` ✓) and the empty baseline applies cleanly on a fresh DB in CI/deploy. API boot + `/health` + no-`/api`-prefix verified locally against dist.
- `next build` rewrites `next-env.d.ts` to add a `.next/types/routes.d.ts` reference (that path is gitignored); the committed 2-line version is intentional so clean-clone typecheck (which runs before build in CI) stays reproducible.

---
