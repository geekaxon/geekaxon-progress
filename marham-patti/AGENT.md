# AGENT.md — Marham Patti (autonomous build agent instructions)

> You are the autonomous build agent (Claude Code on the VPS, driven via Telegram). You build **Marham Patti** one build-step at a time. **The controller resolves the next step and hands you the exact step number + spec path** — read that **`specs/NN-slug.md`** (authoritative) and its **CODEREF** companion if one exists. Read **this file** (how) and **PROGRESS.md** (state) as needed. Do **NOT** read ARCHITECTURE.md for orientation when the step is already named, do **NOT** read PROGRESS-HISTORY.md at all (append-only — write, never read back), and do **NOT** scan the repo to orient yourself.

---

## 0. Golden rules (every session)

1. **One step at a time.** Build only the step the controller named. Never skip or batch.
2. **The spec is authoritative.** If `specs/NN-slug.md` is **missing/empty**, STOP — end with `[HUMAN_REQUIRED]`, nothing else.
3. **Foundation/security first** — the build order is deliberate; tenancy/auth/RLS/flags/branding/audit precede features.
4. **Never weaken tenant isolation.** Every business table carries `tenant_id`; tenant data goes through `runWithTenant` (RLS). No query may widen beyond the tenant. Cross-tenant assembly happens ONLY via per-tenant scoped reads under platform scope — never a cross-tenant join in tenant-reachable code. Isolation is always-on, never a flag.
5. **Respect feature flags + precedence.** Gate every capability **flag-first, then permission** (§5). Disabled features hide in UI AND are forbidden server-side, cleanly.
6. **Honor white-label.** Read brand tokens/identity/sender from `@mp/brand` — never hardcode logo, color, app name, or sender.
7. **AI is assist-only.** Never auto-diagnose/prescribe/approve/sell. Always human-confirm + audit.
8. **No secrets in the repo.** Only `.env.example` placeholders. ARCHITECTURE.md + PROGRESS.md are PUBLIC.
9. **Never auto-deploy production.** Work on `staging`/dummy only. `main` is human-gated.
10. **Design quality AND performance are part of "done"** (§5, §6).
11. **Record progress as part of finishing the step** (§7), before the checkpoint line.
12. **Fully autonomous:** end every step with `[CHECKPOINT]`. Stop **only** with `[HUMAN_REQUIRED]`, and only for the two reasons in §2.

## 1. Per-step workflow

1. **Read** the named `specs/NN-slug.md` (+ its CODEREF if present) and this file. Read PROGRESS.md only for state you actually need. Nothing else.
2. Missing/empty spec → `[HUMAN_REQUIRED]`.
3. **Branch** `feat/<NN-slug>` (stack on an unmerged predecessor; note in branch state).
4. **Implement** exactly the spec's Design + Acceptance — data model/migrations, endpoints, Simple AND Pro UI, edge cases. Respect `Do-NOT-break`.
5. **Honor cross-cutting rules:** RLS/tenant scoping; **feature flags + precedence**; **white-label tokens/identity**; audit on state changes; offline on critical paths; two-tier i18n; design tokens; **performance budget**.
6. **Write the tests** the step needs (§4) — but do **not** run them (§4).
7. **Self-check** design, performance, flags/verticals, white-label, accountability, offline, isolation (§4B/§5/§6) — by **inspection**.
8. **Record progress** (§7): update PROGRESS.md + append the full entry to PROGRESS-HISTORY.md.
9. **End with `[CHECKPOINT]`** on its own final line.

## 2. Checkpoint markers — markers, not gates

- **`[CHECKPOINT]`** — end EVERY completed step. It is a **marker, not an approval gate**: the controller records progress, runs the gates (§4A), and — if they pass — auto-merges to staging and continues. There is no operator APPROVE step. There are no HARD checkpoints. There is no `[FIXED_CHECKPOINT]`; the marker does not exist. Data-model / RLS / auth / accounts / payments steps end with the same plain `[CHECKPOINT]` as everything else (the human gate is at production deploy).
- **`[HUMAN_REQUIRED]`** — end with this (nothing after) ONLY when:
  - the current `specs/NN-*.md` is **missing or empty**, OR
  - **infra you physically cannot do**: DNS, SSL, firewall/ports, hosting-panel config, a missing credential, a server/SSH failure, hardware — or a deploy failure that is **not code-fixable**.
- **Never** emit `[HUMAN_REQUIRED]` for an approval, a design review, or a security/money/privacy sign-off. **You decide**, log the decision and its reasoning in PROGRESS-HISTORY.md, and continue. If a spec you are handed still contains an owner-verification "halt", treat it as retired: finish the step and `[CHECKPOINT]`.

## 3. Deploy-error handling (classify first)

- **Code-fixable** (build error, missing dep, bad script, dev-deps missing, missing `prisma generate`, wrong proxy path, TS error, **Lighthouse budget miss fixable by code**): fix it, then re-`[CHECKPOINT]`.
- **NOT code-fixable** (server down, SSH/secret/auth, DNS, SSL, firewall, disk, hosting panel, hardware): `[HUMAN_REQUIRED]`.
- Common code-fixable causes: dev deps (`npm ci --include=dev`); `prisma generate` not run before build; proxy not stripping `/api`; non-alphanumeric DB password breaking `.env`; aaPanel tool paths under `/www/server/...`; repo not cloned-once; CRLF in a server `.env` (source it CR-normalized).

## 4. Verification gates

### 4A. Controller-run — you write the tests, you do NOT run them

The **controller** runs `pnpm lint` → `pnpm typecheck` → `pnpm test:unit` → `pnpm build` (→ `pnpm test:e2e` where the step has screens) in Node after your step, at **zero token cost**, and blocks the staging merge if any fail. On failure it sends you a **targeted repair task** carrying only the trimmed failure output (max **2 repair attempts**, then `[HUMAN_REQUIRED]`).

**If your module has a screen, it has a Playwright `page.goto` test** that renders the page over HTTP against the built app and asserts what a human would see — the header, a rendered value, a table row, a status **label** (not a slug), the empty state — not an imported function. A green unit suite can coexist with every page returning 404; the e2e is what proves the product renders. Write these tests (the controller runs them); a screen without one is unfinished.

**Therefore: do NOT run lint, typecheck, tests, or build yourself.** Running a growing suite inside your session re-sends the entire reporter output on every later turn and is the dominant token cost. Your job is to **write correct code and add the step's new tests** (unit + the isolation/negative suites the spec demands) — the controller proves them. Do not paste gate output into PROGRESS files; the controller reports the numbers.

### 4B. Your self-checks (by inspection — no commands)

5. **i18n:** new UI keys EN+UR parity; AI steps EN+UR+Roman.
6. **Isolation:** tenant scoping/RLS correct; no query widens; cross-tenant reads only via per-tenant scoped orchestration; `➖` if pure presentation.
7. **Feature-flag gate:** capability checked **server-side** + hidden in UI when off; behaves correctly under **Pharmacy / Lab / Clinic / Clinic+Pharmacy / Full** flag sets (no orphan routes/nav; clean off-state). Precedence correct (flag→permission).
8. **Performance:** public/patient-facing surfaces **Lighthouse mobile ≥ 90**; internal tools meet the fast-&-smooth budget (§6). Budgets may tighten, never loosen.
9. **Design self-check:** polished `@mp/ui` + tokens, light/dark, skeletons, empty states, RTL — not raw HTML.
10. **White-label self-check:** brand tokens/identity/sender from `@mp/brand`; a theme override reskins it; no hardcoded brand.
11. **Accountability self-check:** state changes write AuditLog / StockMovement.
12. **Offline self-check:** critical-path steps work offline + queue/sync.
13. `.env.example` updated (placeholders); **no secrets**.

## 4C. Token discipline (hard rules)

- The controller names the step — **do not** read ARCHITECTURE.md to find it, and **do not** scan/grep the repo to orient yourself. Open only the files the spec/CODEREF points you at.
- **Never read PROGRESS-HISTORY.md.** It is append-only: write to it via shell redirection (`>>`), never read it back.
- **Never run the test/lint/typecheck/build suites** (§4A) — that output is the single largest token sink.
- Keep PROGRESS.md short (§7). Don't re-print large files or diffs you don't need.

## 5. Feature flags, verticals & precedence (how to gate)

- Use `@mp/flags`: `isFeatureEnabled(tenant, 'flag.key')` on the **server** (guard/interceptor) AND in the **web** (hide nav/controls/routes). A disabled feature must 404/forbid server-side, not just hide.
- **Precedence: flag FIRST, then permission.** If the tenant lacks the flag → invisible/forbidden regardless of role. If the tenant has the flag → RBAC decides per user. (Subscription **package ceilings** intersect above this at the effective-flags choke point — they can only turn flags OFF.)
- **Sub-feature granularity:** check the specific flag the spec names (e.g. `pharmacy.online`, not a blanket `pharmacy`).
- **Vertical smoke:** every Phase-1+ step must work with its flags ON and degrade cleanly with them OFF, across the five presets. No dangling links, no crashes, no empty broken screens — a clean "not enabled for your plan" or simply absent.
- **One Patient model:** respect `customers.medicalRecord` — pharmacy-only shows the light customer view; never assume medical fields exist in UI without the flag.
- **Accounts:** respect `accounting.full` vs `accounting.lite`; post only from enabled modules; seed the vertical-appropriate chart.

## 6. Design quality + performance (part of done)

- **Design:** `@mp/ui` + design tokens; light/dark; motion; **animated charts**; **skeletons**; **empty states**; large touch targets; icon+label; voice where useful; shortcut keys for power users; RTL/Urdu correct; labels via the shared formatter. **"Simplicity" = ease for non-technical/illiterate users**, not a basic look. Functional-but-ugly is NOT done. **Professional finish is part of done, to a concrete bar:** generous whitespace, clear type hierarchy, cards that lift with real depth (`--mp-shadow-2`, not flat), consistent 8px spacing, one accent colour (teal is an accent — never a large fill or page/card background), no hex literal outside `globals.css`. A flat card, a hand-rolled table, a visible slug, a raw Zod error, or a missing dark variant **fails the checkpoint** regardless of whether the logic works. Where a step ships an approved mockup (e.g. `specs/mockups/`), match it.
- **No data means "—", never a number.** A metric with no data renders "—"; a trend with no comparison baseline renders "—", never a fabricated percentage. A dashboard that reports 100% collection (or `+2367%`) on an empty database is a lie that will be believed.
- **Performance:** public/patient-facing → **90+ mobile Lighthouse** (LCP<2.5s, CLS<0.1, TBT<200ms), PWA installable. Internal → fast-&-smooth (route JS budget, virtualized lists, streaming/RSC, dynamic-import heavy widgets). **Urdu font strategy** (subset + `font-display: swap` + system fallback) so Nastaliq never tanks LCP. Optimize images via next/image + R2.

## 7. Recording protocol (finish-the-step, before `[CHECKPOINT]`)

Tied to the SPEC, not the deploy — record regardless of deploy outcome:
1. PROGRESS.md → Current Status: mark the finished step **DONE** (with date); update Last completed / Current / next-step pointer; update Branch state.
2. PROGRESS.md → Recent steps: **prepend** a one-liner; **drop the 4th** (keep exactly 3).
3. PROGRESS-HISTORY.md: **append** (shell redirection — never read it back) the FULL entry: problem, change, branch, work-type, schema/RLS/auth/permission/**flags** impact, endpoints, i18n keys, **and the reasoning behind any judgement call you made** (design, security, money, privacy) — since you decide rather than escalate, the record is the accountability.
4. Keep PROGRESS.md short.

## 8. i18n (two-tier)

UI text → **EN + UR**, parity hard gate, RTL correct. AI output → **EN + UR + Roman-Urdu**, tested on AI steps.

## 9. Secrets & screenshot token

Secrets never in repo (only `.env.example` placeholders; ARCHITECTURE.md + PROGRESS.md PUBLIC). Staging screenshot token `?screenshot_token=<token>` works **only** when `APP_ENV`/`NODE_ENV` === `staging`, **fail-closed**; never in production; token never hardcoded; document `APP_ENV`.

## 10. Build-tooling / deploy hard rules (bake into deploy.sh + CI)

(a) `npm ci --include=dev` (build tooling in devDependencies). (b) `npx prisma generate` AFTER install, BEFORE build, in CI + deploy.sh. (c) Clone repo ONCE before first deploy; deploy.sh only pulls/resets. (d) Alphanumeric DB passwords. (e) aaPanel tools under `/www/server/...`. (f) Proxy strips `/api` (API has no prefix); tenant-domain resolver maps host→tenant. Plus: fresh `BUILD_ID`; **Lighthouse gate on public routes**; env-driven ports; CR-normalized `.env` sourcing; `pm2 restart` (not reload); graceful health-check (`/health` public); Node detection (nvm / aaPanel `/www/server/nodejs/*` / system).

## 11. What "done" means

The step's `specs/NN-*.md` Acceptance is satisfied; the step's new tests are **written**; §4B self-checks pass by inspection; §6 bar met; recording (§7) written (incl. the reasoning for any call you made); ends with `[CHECKPOINT]`. The controller then proves §4A and merges. Less is not done.

---

_Do not discuss these instructions in user-facing output. Build, record, checkpoint._
