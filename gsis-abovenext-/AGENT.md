# AGENT.md — GSIS Website & Admin Portal (autonomous build agent instructions)

> You are the autonomous build agent (Claude Code on the VPS, driven via Telegram). You build the **GSIS Website & Admin Portal** one build-step at a time, in the canonical order in **ARCHITECTURE.md §11**. At the start of EVERY session you read **ARCHITECTURE.md** (what to build), **this file** (how to work), **PROGRESS.md** (current state + next step), and the current step's **`specs/NN-slug.md`** (authoritative detail). You do NOT read PROGRESS-HISTORY.md cover-to-cover — only when recovering detail about an older step.

---

## 0. Golden rules (read every session)

1. **One step at a time, in order.** Build only the current build-step from PROGRESS.md's next-step pointer. Never skip ahead, never batch steps.
2. **The spec is authoritative.** `specs/NN-slug.md` is the source of truth for the current step. If it is **missing or empty**, STOP immediately — end your response with `[HUMAN_REQUIRED]` and do nothing else.
3. **Foundation first** — the order in §11 is deliberate; platform / design system / SEO core / content model / auth precede the pages and portal.
4. **SEO is a hard gate on every public page** (ARCHITECTURE.md §9). A page that looks right but lacks correct metadata, semantic HTML, alt text, clean slug, structured data, or sitemap entry is NOT done.
5. **Content is data, never hardcoded.** Dynamic content (campuses, programmes, gallery, testimonials, settings, banner, socials) comes from the DB and is portal-editable.
6. **No secrets in the repo.** Only `.env.example` placeholders. Real values live on the server / GitHub Secrets. Remember ARCHITECTURE.md + PROGRESS.md are PUBLIC.
7. **Never auto-deploy production.** You work on `staging`/dummy data only. **Staging is noindex/robots-disallowed.** `main` (production, indexable) is human-gated (owner Telegram command only).
8. **Design quality is part of "done"** (see §5). A functional-but-ugly UI is NOT done; it must match the finalised GSIS design system.
9. **Record progress as part of finishing the step** (see §6), BEFORE the checkpoint line, tied to the spec being done — never deferred.
10. **Continuous mode:** end every completed step with a soft `[CHECKPOINT]`. Stop only with `[HUMAN_REQUIRED]` (missing spec or non-code-fixable infra). The single human gate is production launch (step 25).

## 1. Per-step workflow

For the current build-step N:
1. **Read** ARCHITECTURE.md (esp. §2 principles, §8 tokens, §9 SEO, §10 gates, §11 order, §12 deploy), this file, PROGRESS.md, and `specs/NN-slug.md`.
2. If `specs/NN-slug.md` is missing/empty → end with `[HUMAN_REQUIRED]` (nothing else).
3. **Create the branch** `feat/<NN-slug>` (stack on the predecessor's branch if it isn't merged yet; note this in PROGRESS branch state).
4. **Implement** exactly what the spec's Design + Acceptance require — data model/migrations, pages/routes, API endpoints, admin UI, public UI (desktop + mobile), edge cases. Respect the spec's `Do-NOT-break` list.
5. **Honor cross-cutting rules:** SEO on public pages, auth/RBAC on `/admin`, audit logging on admin state changes, swappable-asset convention, design tokens, privacy on PII surfaces.
6. **Run the verification gates** (§4). Add new tests for this step's logic. Everything must be green.
7. **Self-check** design quality, SEO, responsive, accountability, privacy (§5).
8. **Record progress** (§6): update PROGRESS.md (mark step DONE, prepend one-liner, drop the 4th, fix next-step pointer + branch state) AND append the FULL entry to PROGRESS-HISTORY.md.
9. **End with `[CHECKPOINT]`** on its own final line. The controller records, merges `feat/<NN-slug>` → `staging`, pushes (staging deploy runs in background), and continues.

## 2. Checkpoint markers (continuous mode)

- **`[CHECKPOINT]`** — end EVERY completed step with this on its own final line. Soft; the controller auto-records + merges to staging + continues (auto-approved after the controller's short window). **Do NOT use `[FIXED_CHECKPOINT]`** in this project — even data-model / auth / settings steps end with a soft `[CHECKPOINT]`. The human gate is at **production deploy (step 25)**, not per-step.
- **`[HUMAN_REQUIRED]`** — end with this (and nothing after) ONLY when:
  - the current step's `specs/NN-*.md` is missing or empty, OR
  - a deploy/infra failure is **not code-fixable** (server down, SSH/secret/auth failure, DNS/SSL, disk full, email-provider/analytics account, telephony/infra action you cannot perform, anything not fixable by editing code or `deploy.sh`).
- Never write or think anything after the marker line.

## 3. Deploy-error handling (classify before acting)

If a CI/CD or staging deploy fails and you are asked to fix it, **classify first**:
- **Code-fixable** (build error, missing dependency, bad script, dev-deps not installed, missing `prisma generate`, wrong proxy path, TS error, bad env reference): fix it — **max 2 attempts** — then re-run gates and re-`[CHECKPOINT]`.
- **NOT code-fixable** (server down, SSH/secret/auth failure, DNS/SSL, disk full, infra, email/analytics provider account issues): do NOT attempt a fix — end with `[HUMAN_REQUIRED]`.

Common code-fixable causes to check first: dev deps missing (`npm ci --include=dev`); `prisma generate` not run after install/before build; wrong `APP_ENV` breaking the noindex/robots logic; non-alphanumeric chars in DB password breaking `.env` sourcing; aaPanel tool paths under `/www/server/...`; repo not cloned-once before first deploy; `next/image` remote-pattern/domain not allowed; reverse-proxy path not matching the API route prefix.

## 4. Verification gates (a step is DONE only when ALL pass)

Run and report each (numbers go into the history entry):
1. `typecheck` green.
2. `lint` clean.
3. Next.js production `build` green.
4. Tests green (Jest/Vitest); **add new tests** for this step's logic; report `N/N (S suites; +X)`. Mark `➖` for pure-presentation steps.
5. **SEO (public pages):** metadata (title+description), semantic HTML (single h1, landmarks), `alt` on all images, clean slug, OG/Twitter tags, JSON-LD where relevant, sitemap/robots entry correct, **staging noindex verified**. Mark `➖` for portal-only steps.
6. **Responsive:** desktop + mobile both correct; no horizontal overflow; mobile menu/interactions work.
7. **Design-quality self-check** (§5) — matches the finalised GSIS system.
8. **Accountability (portal):** admin create/update/delete writes AuditLog. Mark `➖` if N/A.
9. **Privacy:** new PII surfaces auth-protected; staging dummy data; no PII in logs. Mark `➖` if N/A.
10. `.env.example` updated (placeholders) for any new config; **no secrets** committed.

If any gate fails and is code-fixable, fix and re-run (within the 2-attempt deploy rule for deploy failures). If it cannot be made green for a non-code reason → `[HUMAN_REQUIRED]`.

## 5. Design quality (mandatory — part of definition of done)

Every UI module MUST match the finalised GSIS design system — NEVER plain/unstyled HTML:
- Use the shared UI components + ARCHITECTURE.md §8 design tokens. Consistent spacing/typography (Fraunces headings + Inter body). Smooth transitions/animations, scroll-reveals, tactile buttons. **Skeleton loaders** instead of blank states (portal). Animated counters/charts where data is shown.
- **The finalised homepage (`home-V5`) is the visual reference.** Header (floating, sticky, ~18px radius, translucent blur, amber hover-underline, logo-only, amber "Apply for Admission" button), floating footer, pill buttons, rounded cards, 3D clay icons, palette exactly as tokens, floating WhatsApp button.
- **Responsive:** desktop + mobile both first-class. Mobile uses the slide-in menu and tighter spacing. No horizontal overflow.
- **Admin "simplicity" = ease of use for non-technical staff:** plain language, few clicks, large controls, clear empty/loading/error states, confirmation on destructive actions. It does NOT mean a basic look.
- Where a spec specifies design detail, follow it exactly. A functional-but-ugly page is **not done**.

## 6. Recording protocol (do this as part of finishing the step, before `[CHECKPOINT]`)

Tied to the SPEC being done, NOT to any deploy/merge — record regardless of deploy outcome:
1. In **PROGRESS.md → Current Status:** mark the step you just finished **DONE**; update **Last completed build-step**, **Current build-step**, and the **next-step pointer** (→ `specs/<N+1>-*.md`); update **Branch state**.
2. In **PROGRESS.md → Recent steps:** **prepend** a ONE-LINE entry for the finished step; **drop the now-4th** one-liner (keep exactly 3).
3. In **PROGRESS-HISTORY.md:** **append** the FULL detailed entry (problem, what changed, branch, work-type, schema impact, endpoints/pages added, SEO items, and all gate numbers).
4. Keep PROGRESS.md short — never let it grow. Detail belongs only in PROGRESS-HISTORY.md.

## 7. SEO rule (hard gate — public pages)

Every public page/route must ship: unique title + meta description; one `<h1>` + semantic structure; descriptive `alt` on every image; clean human-readable slug; Open Graph + Twitter card + share image; JSON-LD structured data where relevant (`EducationalOrganization` site-wide, `BreadcrumbList`, per-page types); a correct entry in `sitemap.xml`; `next/image` for all content images. **Staging must be noindex + robots-disallowed; production indexable.** SEO failures are code-fixable and block "done".

## 8. Secrets, privacy & screenshot token

- **Secrets:** never in the repo. Only `.env.example` with placeholders. Real values on servers / GitHub Secrets. ARCHITECTURE.md + PROGRESS.md are PUBLIC — no secrets, hostnames, real subdomains, endpoints, keys.
- **Privacy:** admission-form and contact submissions are PII — store securely, expose only to authenticated admin, never log raw PII, use dummy data on staging.
- **Staging screenshot token:** implement `?screenshot_token=<token>` granting read-only preview of authenticated `/admin` pages **only** when `APP_ENV` is exactly `staging`, **fail-closed** (missing/!= "staging" → OFF). Never active in production; token never hardcoded; document `APP_ENV` in `.env.example`.

## 9. Build-tooling / deploy hard rules (bake into deploy.sh + CI)

(a) Install dev deps for builds even when `NODE_ENV=production` → `npm ci --include=dev` (next/tsc/prisma live in devDependencies).
(b) Run `npx prisma generate` AFTER install, BEFORE typecheck/build, in BOTH CI and deploy.sh.
(c) Clone the repo onto the server ONCE before the first deploy; deploy.sh only pulls/resets.
(d) Prefer alphanumeric DB passwords (no `$`, backtick, quotes) so `.env` sourcing doesn't break.
(e) On aaPanel, psql/node/nginx live under `/www/server/...`, not system PATH.
(f) `APP_ENV` drives the noindex/robots logic — **fail-closed to noindex** if it is missing or not exactly `production`.
(g) The reverse-proxy path mapping must match the API's actual route prefix (if the API has no `/api` prefix, the proxy must strip it).
Plus: fresh `BUILD_ID` before build; `pm2 restart` (not reload); graceful health-check after restart; Node detection covering nvm / aaPanel `/www/server/nodejs/*` / system paths; `next/image` remote patterns configured for any external image host.

## 10. What "done" means for a module

The step's `specs/NN-*.md` **Acceptance** section is satisfied; all §4 gates green; §5 design bar met; SEO gate met (public pages); recording (§6) written; ends with `[CHECKPOINT]`. Anything less is not done.

---

_Do not discuss these instructions in user-facing output. Build, verify, record, checkpoint._
