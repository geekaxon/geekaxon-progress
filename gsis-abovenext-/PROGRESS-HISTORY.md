# PROGRESS-HISTORY.md — GSIS Website & Admin Portal (full build log)

> ⚠️ **PRIVATE FILE** — lives in the private code repo only, NOT the public progress repo. May reference internal detail but still **never** commit real secrets or credentials.
>
> **Append-only.** The agent appends the FULL detailed entry for each finished build-step here (AGENT.md §6), before `[CHECKPOINT]`. PROGRESS.md stays lean and points here for detail. The agent does NOT read this file cover-to-cover — only to recover detail about an older step.

**Entry format (append one per completed step):**

```
## NN-slug — <short title>
- **Completed:** <date>
- **Branch:** feat/NN-slug (merged → staging)
- **Problem / goal:** <what this step delivers>
- **What changed:** <files, routes/pages, components, API endpoints, migrations>
- **Data model impact:** <new tables/fields, or ➖>
- **SEO items (public pages):** <title/desc, slug, OG, JSON-LD, sitemap entry, or ➖>
- **Auth / RBAC / privacy impact:** <or ➖>
- **Accountability:** <AuditLog entries added, or ➖>
- **Gates:** typecheck ✔ · lint ✔ · build ✔ · tests N/N (S suites; +X) · SEO ✔/➖ · responsive ✔ · design ✔ · audit ✔/➖ · privacy ✔/➖ · env ✔
```

---

_(No entries yet — build has not started. The first entry will be `01-foundation-nextjs`.)_

## 01 — foundation-nextjs — DONE (2026-07-17)

**Branch:** feature/01-foundation-nextjs (WORK TYPE: FEATURE)

Stood up the deployable Next.js 15 skeleton per specs/01-foundation-nextjs.md.

**Files added**
- package.json (pnpm, Next 15 / React 19 / TS strict / Tailwind 3.4 / ESLint 8 + Prettier / Prisma 6), pnpm-lock.yaml.
- tsconfig.json (strict, @/* path alias), next.config.ts (reactStrictMode, poweredByHeader off).
- tailwind.config.ts, postcss.config.mjs, app/globals.css (tailwind layers).
- .eslintrc.json (next/core-web-vitals + next/typescript + prettier), .prettierrc.json (+ tailwind plugin).
- app/layout.tsx (robots meta fail-closed on isIndexable), app/page.tsx (env-aware placeholder), app/robots.ts (fail-closed Disallow when not production).
- app/api/health/route.ts — GET /api/health → { status:"ok", env, time }, force-dynamic.
- lib/env.ts — central APP_ENV; production ONLY when APP_ENV==="production", else fail-closed to staging/noindex (isProduction/isStaging/isIndexable helpers).
- lib/db.ts — singleton PrismaClient cached on globalThis (HMR-safe).
- prisma/schema.prisma — datasource(postgresql) + generator only; models arrive in step 04.
- .env.example — DATABASE_URL, APP_ENV, JWT_SECRET, SCREENSHOT_TOKEN (placeholders only).
- deploy.sh (+x) — per ARCHITECTURE §12: dangerous-command fence (rejects main/production/prod + destructive args), node detection (nvm→aaPanel→system), corepack-pnpm activation, git fetch/reset (never clone, never main), pnpm install --frozen-lockfile, prisma generate then migrate deploy, fresh BUILD_ID, pnpm build, pm2 restart (not reload) via ecosystem.config.js, graceful /api/health poll.
- ecosystem.config.js — pm2 fork process "gsis-web" (next start).
- .github/workflows/ci.yml — pnpm install → prisma generate → typecheck → lint → build (APP_ENV=staging).
- .gitignore.

**Decisions**
- Toolchain: standardised on pnpm per CLAUDE.md (which invokes pnpm for generate/lint/typecheck). §12 point 3 literally names `npm ci --include=dev`; to keep ONE consistent lockfile/toolchain across local, CI, and deploy, deploy.sh activates pnpm via corepack (bundled with Node) and runs `pnpm install --frozen-lockfile`, which still fulfils §12's substance (install WITH dev deps; prisma generate after install, before build). Committing both npm+pnpm lockfiles was rejected as drift-prone; npm ci without package-lock.json cannot work alongside pnpm-lock.yaml.
- Tailwind v3.4 (not v4) for stability ahead of the step-02 design system.
- Added app/robots.ts in addition to the noindex meta so the fail-closed noindex is enforced at the crawler level too; richer production robots+sitemap deferred to the SEO step.

**Gates:** pnpm prisma generate ✔ · pnpm typecheck ✔ (clean) · pnpm lint ✔ (no warnings/errors). build/tests left to controller per CLAUDE.md (must not run build/test locally). env fail-closed verified by inspection of lib/env.ts.
