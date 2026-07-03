# AGENT.md — Marham Patti (autonomous build agent instructions)

> You are the autonomous build agent (Claude Code on the VPS, driven via Telegram). You build **Marham Patti** one build-step at a time, in the canonical order in **ARCHITECTURE.md §14**. At the start of EVERY session read **ARCHITECTURE.md** (what to build), **this file** (how), **PROGRESS.md** (state + next step), and the current step's **`specs/NN-slug.md`** (authoritative). Do NOT read PROGRESS-HISTORY.md cover-to-cover — only to recover an older step's detail.

---

## 0. Golden rules (every session)

1. **One step at a time, in order.** Build only the current build-step from PROGRESS.md's next-step pointer. Never skip or batch.
2. **The spec is authoritative.** If `specs/NN-slug.md` is **missing/empty**, STOP — end with `[HUMAN_REQUIRED]`, nothing else.
3. **Foundation/security first** — §14 order is deliberate; tenancy/auth/RLS/flags/branding/audit precede features.
4. **Never weaken tenant isolation.** Every business table carries `tenant_id`; tenant data goes through `runWithTenant` (RLS). No query may widen beyond the tenant. Isolation is always-on, never a flag.
5. **Respect feature flags + precedence.** Gate every capability **flag-first, then permission** (§ below). Disabled features hide in UI AND are forbidden server-side, cleanly.
6. **Honor white-label.** Read brand tokens/identity/sender from `@mp/brand` — never hardcode logo, color, app name, or sender.
7. **AI is assist-only.** Never auto-diagnose/prescribe/approve/sell. Always human-confirm + audit.
8. **No secrets in the repo.** Only `.env.example` placeholders. ARCHITECTURE.md + PROGRESS.md are PUBLIC.
9. **Never auto-deploy production.** Work on `staging`/dummy only. `main` is human-gated.
10. **Design quality AND performance are part of "done"** (§5, §6).
11. **Record progress as part of finishing the step** (§7), before the checkpoint line.
12. **Continuous mode:** end every step with `[CHECKPOINT]`. Stop only with `[HUMAN_REQUIRED]` (missing spec or non-code-fixable infra).

## 1. Per-step workflow

1. **Read** ARCHITECTURE.md (esp. §2 principles, §4 flags/verticals, §11 tokens, §12 gates, §13 perf, §14 order, §15 deploy), this file, PROGRESS.md, `specs/NN-slug.md`.
2. Missing/empty spec → `[HUMAN_REQUIRED]`.
3. **Branch** `feat/<NN-slug>` (stack on an unmerged predecessor; note in branch state).
4. **Implement** exactly the spec's Design + Acceptance — data model/migrations, endpoints, Simple AND Pro UI, edge cases. Respect `Do-NOT-break`.
5. **Honor cross-cutting rules:** RLS/tenant scoping; **feature flags + precedence**; **white-label tokens/identity**; audit on state changes; offline on critical paths; two-tier i18n; design tokens; **performance budget**.
6. **Run all verification gates** (§4). Add new tests. Everything green.
7. **Self-check** design, performance, flags/verticals, white-label, accountability, offline, isolation (§5/§6).
8. **Record progress** (§7): update PROGRESS.md + append full entry to PROGRESS-HISTORY.md.
9. **End with `[CHECKPOINT]`** on its own final line.

## 2. Checkpoint markers (continuous mode)

- **`[CHECKPOINT]`** — end EVERY completed step. Soft; controller auto-records + merges to staging + continues. **No `[FIXED_CHECKPOINT]`** — even data-model/RLS/auth/accounts/payments end with a soft `[CHECKPOINT]` (human gate is at production deploy).
- **`[HUMAN_REQUIRED]`** — end with this (nothing after) ONLY when: the current `specs/NN-*.md` is missing/empty, OR a deploy/infra failure is **not code-fixable** (server down, SSH/secret/auth, DNS, disk, telephony/printer hardware). Never write anything after the marker.

## 3. Deploy-error handling (classify first)

- **Code-fixable** (build error, missing dep, bad script, dev-deps missing, missing `prisma generate`, wrong proxy path, TS error, **Lighthouse budget miss fixable by code**): fix — **max 2 attempts** — then re-run gates + re-`[CHECKPOINT]`.
- **NOT code-fixable** (server down, SSH/secret/auth, DNS, disk, infra, hardware): `[HUMAN_REQUIRED]`.
- Common code-fixable causes: dev deps (`npm ci --include=dev`); `prisma generate` not run before build; proxy not stripping `/api`; non-alphanumeric DB password breaking `.env`; aaPanel tool paths under `/www/server/...`; repo not cloned-once.

## 4. Verification gates (DONE only when ALL pass; numbers go in the history entry)

1. `typecheck` green. 2. `lint` clean. 3. `turbo build` green. 4. Jest green + **new tests**; `N/N (S suites; +X)`.
5. **i18n:** new UI keys EN+UR parity green; AI steps EN+UR+Roman tested.
6. **Isolation:** tenant scoping/RLS correct; no widening; `➖` if pure presentation.
7. **Feature-flag gate:** capability checked **server-side** + hidden in UI when off; **vertical smoke tests** green — module behaves under **Pharmacy / Lab / Clinic / Clinic+Pharmacy / Full** flag sets (no orphan routes/nav; clean off-state). Precedence correct (flag→permission).
8. **Performance gate:** public/patient-facing surfaces **Lighthouse mobile ≥ 90**; internal tools meet the fast-&-smooth budget (§13). Report numbers; if a step ships no public surface, assert `➖` with reason.
9. **Design self-check:** polished shadcn/ui + tokens, light/dark, skeletons, RTL — not raw HTML.
10. **White-label self-check:** brand tokens/identity/sender read from `@mp/brand`; a theme override reskins it; no hardcoded brand.
11. **Accountability self-check:** state changes write AuditLog / StockMovement.
12. **Offline self-check:** critical-path steps work offline + queue/sync.
13. `.env.example` updated (placeholders); **no secrets**.

Fix code-fixable failures and re-run; non-code-fixable → `[HUMAN_REQUIRED]`.

## 5. Feature flags, verticals & precedence (how to gate)

- Use `@mp/flags`: `isFeatureEnabled(tenant, 'flag.key')` on the **server** (guard/interceptor) AND in the **web** (hide nav/controls/routes). A disabled feature must 404/forbid server-side, not just hide.
- **Precedence: flag FIRST, then permission.** If the tenant lacks the flag → invisible/forbidden regardless of role. If the tenant has the flag → RBAC decides per user.
- **Sub-feature granularity:** check the specific flag the spec names (e.g. `pharmacy.online`, not a blanket `pharmacy`).
- **Vertical smoke tests:** every Phase-1+ step must prove it works with its flags ON and degrades cleanly with them OFF, across the five presets. No dangling links, no crashes, no empty broken screens — a clean "not enabled for your plan" or simply absent.
- **One Patient model:** respect `customers.medicalRecord` — pharmacy-only shows the light customer view; never assume medical fields exist in UI without the flag.
- **Accounts:** respect `accounting.full` vs `accounting.lite`; post only from enabled modules; seed the vertical-appropriate chart.

## 6. Design quality + performance (part of done)

- **Design:** `@mp/ui` (Tailwind + shadcn/ui) + §11 tokens; light/dark; motion; **animated charts**; **skeletons**; large touch targets; icon+label; voice where useful; shortcut keys for power users; RTL/Urdu correct; labels via the shared formatter. **"Simplicity" = ease for non-technical/illiterate users**, not a basic look. Functional-but-ugly is NOT done.
- **Performance (§13):** public/patient-facing → **90+ mobile Lighthouse** (LCP<2.5s, CLS<0.1, TBT<200ms), PWA installable. Internal → fast-&-smooth (route JS budget, virtualized lists, streaming/RSC, dynamic-import heavy widgets). **Urdu font strategy** (subset + `font-display: swap` + system fallback) so Nastaliq never tanks LCP. Optimize images via next/image + R2.

## 7. Recording protocol (finish-the-step, before `[CHECKPOINT]`)

Tied to the SPEC, not the deploy — record regardless of deploy outcome:
1. PROGRESS.md → Current Status: mark the finished step **DONE**; update Last completed / Current / next-step pointer (→ `specs/<N+1>-*.md`); update Branch state.
2. PROGRESS.md → Recent steps: **prepend** a one-liner; **drop the 4th** (keep exactly 3).
3. PROGRESS-HISTORY.md: **append** the FULL entry (problem, change, branch, work-type, schema/RLS/auth/permission/**flags** impact, endpoints, i18n keys, and ALL gate numbers incl. **flag/vertical** + **performance**).
4. Keep PROGRESS.md short.

## 8. i18n (two-tier)

UI text → **EN + UR**, parity hard gate, RTL correct. AI output → **EN + UR + Roman-Urdu**, tested on AI steps.

## 9. Secrets & screenshot token

Secrets never in repo (only `.env.example` placeholders; ARCHITECTURE.md + PROGRESS.md PUBLIC). Staging screenshot token `?screenshot_token=<token>` works **only** when `APP_ENV`/`NODE_ENV` === `staging`, **fail-closed**; never in production; token never hardcoded; document `APP_ENV`.

## 10. Build-tooling / deploy hard rules (bake into deploy.sh + CI)

(a) `npm ci --include=dev` (build tooling in devDependencies). (b) `npx prisma generate` AFTER install, BEFORE build, in CI + deploy.sh. (c) Clone repo ONCE before first deploy; deploy.sh only pulls/resets. (d) Alphanumeric DB passwords. (e) aaPanel tools under `/www/server/...`. (f) Proxy strips `/api` (API has no prefix); tenant-domain resolver maps host→tenant. Plus: fresh `BUILD_ID`; **Lighthouse gate on public routes**; `pm2 restart` (not reload); graceful health-check; Node detection (nvm / aaPanel `/www/server/nodejs/*` / system).

## 11. What "done" means

The step's `specs/NN-*.md` Acceptance is satisfied; ALL §4 gates green (incl. flag/vertical + performance + white-label); §6 bar met; recording (§7) written; ends with `[CHECKPOINT]`. Less is not done.

---

_Do not discuss these instructions in user-facing output. Build, verify, record, checkpoint._
