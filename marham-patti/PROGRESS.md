> ⚠️ **PUBLIC FILE** — pushed to a separate public progress repo (`projects-abovenext/marham-patti-progress`). **NO secrets, hostnames, real subdomains, API endpoints, credentials, or keys — ever.** Placeholders only.

# PROGRESS.md — Marham Patti Build Tracker (SHORT)

> **This is the SHORT working tracker the build agent reads at the start of every step.** It holds only: current state, the next-step pointer, the checkpoint rule, branch state, and the last 3 steps as one-liners. The COMPLETE detail is in **PROGRESS-HISTORY.md** (append-only archive).
>
> **At the END of the step you just finished (before `[CHECKPOINT]`, as part of finishing the spec — never deferred):** (1) mark the finished step DONE in **Current Status**, (2) prepend a ONE-LINE entry to **Recent steps**, (3) drop the now-4th one-liner (keep exactly 3), and (4) append the FULL entry to **PROGRESS-HISTORY.md**. Tied to the SPEC being done, NOT to any deploy/merge. Never let this file grow long.
>
> **Numbering:** canonical per ARCHITECTURE.md §14 (continuous 1→N; phase = label).
>
> **Checkpoint rule (continuous mode):** end every step with `[CHECKPOINT]` on its own final line — the controller records PROGRESS, then SILENTLY merges the branch to staging + pushes (deploy runs in background; check with `DEPLOY STATUS`) and continues. Do NOT use `[FIXED_CHECKPOINT]` — even data-model / RLS / auth / flags / accounts / payments changes just end with `[CHECKPOINT]`. Stop ONLY by ending with `[HUMAN_REQUIRED]` when the step's spec is missing/empty or an infra problem is not code-fixable.

## Current Status
- **Project:** Marham Patti (AI-powered, multi-tenant, white-label, vertical-configurable healthcare OS · first tenant Ganatra Clinic = Full suite · built by AboveNext)
- **Phase:** **NOT STARTED.** Launch scope = build-steps **1–39** (Foundation 1–13, Phase 1 core 14–24, Phase 2 ops 25–30, Phase 3 apps 31–34, AI Suite 35–39). Deferred (stub specs only): 40–45 (white-label admin UI, SaaS admin console, subscription billing, CRM, telemedicine, marketplace — marketplace 45 & billing 42 depend on the SaaS console 41).
- **SaaS model:** every capability is a feature flag (sub-feature granularity); flags bundle into **vertical presets** (Pharmacy / Lab / Clinic / Clinic+Pharmacy / Full) with per-tenant overrides. Always-on infra (auth, RLS isolation, audit, offline, flag engine, branding, i18n) is never a flag.
- **Last completed build-step:** **1 — Foundation: Monorepo + DB + Prisma base + health endpoint + deploy.sh + CI.** ✅ DONE/APPROVED (2026-07-03) — pnpm/Turbo monorepo + 7 `@mp/*` packages + api(/health, no `/api` prefix)/web(PWA placeholder)/worker + empty Prisma baseline + deploy.sh(§15) + CI; gates green (typecheck 16/16, lint 0/0, build 10/10, jest 2/2).
- **Current build-step:** **2 — Tenancy + RLS.** Read `specs/02-tenancy-rls.md` (authoritative). If missing/empty, STOP with `[HUMAN_REQUIRED]`.
- **Next-step pointer:** → `specs/02-tenancy-rls.md`
- **Branch state:** `feat/01-foundation-monorepo` complete (awaiting controller merge to `staging`). Step 2 branch `feat/02-tenancy-rls` stacks on it if not yet merged. (Repo must be cloned onto the server ONCE before the first deploy — see ARCHITECTURE.md §15.)

### Recent steps (one-liners — FULL detail in PROGRESS-HISTORY.md)
- **01 — Foundation monorepo** ✅ 2026-07-03 — pnpm+Turbo monorepo, `@mp/{config,db,shared,flags,brand,ui,ai}`, NestJS `/health` (no `/api` prefix), Next.js 15 PWA placeholder, worker, empty Prisma baseline, `deploy.sh`(§15)+`ecosystem.config.js`+CI. Gates: typecheck 16/16, lint 0/0, build 10/10, jest 2/2 (+2). Isolation/flags/perf/i18n ➖ (foundation).

---

> Older steps are archived in **PROGRESS-HISTORY.md**. Read that file only when recovering detail about an older step — never cover-to-cover each step.
