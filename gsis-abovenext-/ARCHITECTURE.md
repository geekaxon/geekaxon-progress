> ⚠️ **PUBLIC FILE** — pushed to a separate public progress repo (`projects-abovenext/gsis-website-progress`). **NO secrets, hostnames, real subdomains, API endpoints, credentials, or keys — ever.** Placeholders only.

# ARCHITECTURE.md — GSIS Website & Admin Portal

> **This is the lean, always-loaded "table of contents".** It holds the overview, principles, stack, data-model summary, design tokens, the **canonical numbered build order**, the SEO spec, and the deployment spec. The DETAIL for each build-step lives in its own file at `specs/NN-slug.md` (private repo only) and is **authoritative** for that step. If a step's spec is missing/empty, the agent STOPS with `[HUMAN_REQUIRED]` rather than guessing.
>
> **Numbering (§ canonical):** build-step numbers here are the single source of truth. "Module N" / "build-step N" = the entry numbered N in §11 BUILD ORDER. Numbers are **continuous 1→N across all phases**; the phase is only a label.

---

## 1. Overview

**GSIS Website & Admin Portal** is a premium, SEO-optimised marketing website for GSIS (an international school in Pakistan, established 1992, tagline *"Education Above and Beyond"*), plus a private admin portal that lets non-technical school staff manage the site's dynamic content and view submissions.

- **Product:** GSIS Website + Admin Portal. **Built & delivered by:** AboveNext. **Client:** GSIS.
- **Two surfaces:** (a) a **public marketing website** (Home, About, Admissions, Admission Form, Campuses + Campus Detail, Curriculum, Gallery, Contact), and (b) a **login-only Admin Portal** (dashboard, admission applications, contact inquiries, and management of campuses / curriculum / gallery / testimonials / admission dates / FAQs / settings / users).
- **Audience:** prospective parents & students (public site) and GSIS staff (portal).
- **Core job-to-be-done:** give the school a **modern, fast, search-optimised web presence** that visitors trust, and give **non-technical staff simple control** over dynamic content and enquiries — with automated email notifications on form submissions.
- **Scope guardrails:** marketing site + light CMS/admin. **NOT** a student portal, fee/payment system, LMS, or ERP. No online payments, no student logins, no grades.

## 2. Principles (non-negotiable, apply to every step)

1. **SEO is a first-class requirement.** Every public page ships correct metadata, semantic HTML, clean URLs, Open Graph/Twitter cards, structured data (JSON-LD), sitemap + robots, and fast Core Web Vitals. Hard gate (§9, §10).
2. **Design quality is a requirement, not a polish step.** Every UI ships the GSIS design system (§8) — polished, spacious, animated, responsive, never raw/unstyled HTML. **The `/mockups/*.html` files are the authoritative visual reference** for all public pages.
3. **Content is data, not code.** ALL dynamic content (campuses, programmes, gallery, testimonials, admission dates, FAQs, settings, banner, socials, contact info) is stored in the database and editable from the Admin Portal — never hardcoded.
4. **Non-technical-friendly admin.** Plain language, few clicks, large controls, clear empty/loading states. ("Simplicity first" = ease of use, NOT a basic look.)
5. **Swappable assets.** EVERY image, icon, logo, and favicon lives under `public/assets/` with stable, human-readable names; replacing a file updates the site with no code change (§8a).
6. **No secrets in the repo.** Only `.env.example` with placeholders. Real values live on the server / GitHub Secrets. ARCHITECTURE.md + PROGRESS.md are PUBLIC.
7. **Staging first; production is human-gated.** The agent works on `staging` with dummy/sample data only. **Staging is hidden from search engines.** `main` (production) deploys ONLY on the owner's explicit Telegram command.
8. **Data privacy.** Admission-form and contact submissions contain personal data. Stored securely, access limited to authenticated admin users, staging uses non-real data only.
9. **Specs are authoritative.** The agent treats the current `specs/NN-*.md` as the source of truth; missing spec → `[HUMAN_REQUIRED]`.
10. **Accountability on admin actions.** Every create/update/delete in the portal is attributed to the logged-in user with a timestamp (AuditLog).
11. **Single light theme.** No dark mode, no theme toggle, no `prefers-color-scheme` handling — site AND portal (§8).
12. **Skeleton loaders only.** No full-page preloaders, spinner overlays, or splash screens anywhere (§8).

## 3. Tech stack

- **Framework:** Next.js 15 (App Router) + React + TypeScript. Server components for SEO-critical public pages; SSG/ISR where possible.
- **Styling:** Tailwind CSS + the GSIS design system (§8). Fonts: Fraunces (display serif) + Inter (body/UI). Icons: Font Awesome (as used in mockups) + the custom 3D clay icon set.
- **ORM/DB:** Prisma + PostgreSQL 16. `prisma generate` runs after install, before typecheck/build, in CI **and** deploy.sh.
- **Auth (portal only):** email + password + JWT (short-lived access + rotating refresh); session timeout; failed-login lockout; password reset. Public site needs no auth.
- **Email:** **SMTP** (nodemailer) configured from `.env` — real transactional sending for admission-form and contact-form notifications (to staff) and acknowledgements (to submitter). See spec `38`.
- **Files/images:** all static brand/page imagery under `public/assets/` (§8a). Admin-uploaded media (gallery, campus, testimonial photos) stored under `public/assets/uploads/` at launch; object storage optional later.
- **Media handling:** Next.js `<Image>` for optimisation (responsive sizes, lazy-loading, modern formats).
- **Package manager:** **pnpm** (via corepack). All install/build commands use pnpm.
- **Infra:** AboveNext Oracle Cloud ARM VPS, aaPanel, Nginx (reverse proxy → PM2 app port), PM2, Node v22, Cloudflare DNS. Staging + human-gated production.
- **Analytics:** privacy-friendly analytics + Google Search Console verification on production. `[DECIDE AT BUILD]` exact provider.

## 4. Domains (subdomain placeholders — never commit real hosts)

- Production: `<PROD_APP_HOST>`.
- Staging: `<STAGING_APP_HOST>` — **noindex, robots-disallowed**.
- Email sending domain, analytics IDs → placeholders only; real values in server `.env` / GitHub Secrets.

## 5. Repos & branch flow

- **Private code:** contains `/specs/NN-slug.md` and `/mockups/*.html`.
- **Public progress:** ONLY `ARCHITECTURE.md` + `PROGRESS.md`.
- **Branches:** `feat/<NN-slug>` → merge to **`staging`** (auto-deploys) → **`main`** (production; human-gated).

## 6. Roles (portal RBAC — catalog-driven)

- **`ADMIN`** (GSIS staff): full access — view submissions, manage all content + settings, manage editors.
- **`EDITOR`**: manage content (campuses, curriculum, gallery, testimonials, dates, FAQs) but **not** settings or user management.
- **`SUPER_ADMIN`** (AboveNext): platform-level break-glass access.

Permissions are catalog-driven string keys (e.g. `applications.view`, `campuses.edit`, `faqs.edit`, `settings.edit`), seeded + reconciled on boot.

## 7. Auth (portal)

- Email + password + JWT (short-lived access + rotating refresh, httpOnly cookies). Optional 2FA `[DECIDE AT BUILD]`.
- Session timeout, failed-login lockout, password reset via email. All auth events audited.
- **Public website requires no authentication**; only `/admin/*` is protected.

## 8. Design tokens (authoritative — from `/mockups`)

**The `/mockups/*.html` files are the authoritative visual reference.** Tokens as defined there:

```
--navy:         #0F2A47   /* PRIMARY — dominant colour across the entire site */
--navy-deep:    #0A1E33   /* dark surfaces, footer */
--teal:         #0E8A7A   /* secondary */
--teal-bright:  #14A594   /* secondary highlight */
--amber:        #F5A623   /* ACCENT ONLY — sparing use, never dominant */
--off-white:    #F7F9FB   /* section backgrounds */
--paper:        #FFFFFF
--slate:        #2B3A4A   /* body text */
--slate-soft:   #5B6B7B
--line:         #E4EAEF   /* borders */
--radius:       18px
--maxw:         1320px
--shadow-sm:    0 2px 10px rgba(15,42,71,.06)
--shadow-md:    0 14px 40px rgba(15,42,71,.10)
--shadow-lg:    0 30px 70px rgba(15,42,71,.16)
```

- **Colour rule:** **deep navy `#0F2A47` is the dominant brand colour** for headings, primary buttons, header/footer, key accents and emphasis. Amber is an **accent only** (a single underline or highlight) and must never dominate a page.
- **Type:** Fraunces (display serif, headings) + Inter (body/UI).
- **Components:** ~18px rounded cards; pill buttons with tactile hover (lift + shadow + shine + arrow-nudge); floating rounded sticky header with translucent blur (logo-only left, nav with hover-underline, "Apply for Admission" button); floating footer; soft layered shadows; scroll-reveal animations; animated counters; tabbed panels; sliders/carousels; floating WhatsApp button.
- **Backgrounds:** section backgrounds blend **gradually** (soft white ↔ off-white gradients). No hard/sharp colour edges between sections.
- **Ghost/outline buttons:** border must be clearly visible at rest (not dissolving into the background); hover state as designed.
- **THEME:** **single light theme only.** No dark mode, no theme toggle in the header, no `prefers-color-scheme` blocks, no `.dark`/`data-theme` hooks — public site AND portal.
- **LOADING:** **skeleton loaders only.** No full-page preloaders, spinner overlays, or splash screens. Inline button submit states are permitted.
- **Branding:** GSIS crest logo (header + footer); footer credit "Built with ♥ by AboveNext" → abovenext.com (plain link, no tracking).
- **Footer nav:** does NOT include Campuses, Privacy Policy, or Terms & Conditions; does NOT include a WhatsApp social icon (the floating WhatsApp button is retained).
- **Responsive:** desktop + mobile first-class; container ~1320px; mobile slide-in menu.

## 8a. Asset system (every image is swappable)

All imagery is a file under `public/assets/` — no inline/base64, no external hotlinks. Canonical structure:

```
public/assets/
  branding/    logo.png, logo-navy.png, favicon.ico, favicon-32.png,
               apple-touch-icon.png, og-default.jpg, site.webmanifest
  icons/       val-1..5.png, why-1..4.png, prog-1..5.png   (3D clay icons)
  pages/       hero.jpg, about-1.jpg, about-2.jpg, why.jpg, principal.jpg,
               avatar-1..4.jpg, page-hero-*.jpg
  campuses/    (campus imagery; DB-referenced)
  curriculum/  (programme imagery; DB-referenced)
  gallery/     (album imagery; DB-referenced)
  testimonials/(author photos; DB-referenced)
  uploads/     (admin-uploaded media at runtime)
```

Rules: stable lowercase hyphenated filenames; replacing a file updates the site with **no code change**; every `<Image>` has descriptive `alt`; DB image fields store `/assets/...` paths; a documented `ASSETS.md` manifest lists every filename, where it appears, and recommended dimensions.

## 9. SEO spec (hard gate on every public page)

1. **Metadata:** unique `<title>` + meta description per page (Metadata API).
2. **Semantic HTML:** correct heading hierarchy (one `<h1>`), landmarks, descriptive `alt` on every image.
3. **Clean URLs:** human-readable slugs (`/campuses/gsis-primary`), no query-string IDs.
4. **Social cards:** Open Graph + Twitter tags with a share image per page.
5. **Structured data (JSON-LD):** `EducationalOrganization` (site-wide), `BreadcrumbList`, `FAQPage` on Admissions, per-page types where relevant.
6. **Sitemap + robots:** auto-generated. **Staging = noindex + Disallow all; production = indexable** (fail-closed).
7. **Performance / CWV:** optimised images, lazy-loading, minimal blocking JS, good LCP/CLS/INP.
8. **Canonical URLs** on every page.
9. **Accessibility baseline** (contrast, focus states, keyboard nav).

## 10. Verification gates (every step's definition of done)

1. `typecheck` green. 2. `lint` clean. 3. `build` green. 4. Tests green with **new tests added**; report `N/N (S suites; +X)` (`➖` for pure-presentation).
5. **SEO gate** (public pages) — metadata, semantics, alt, slug, OG/Twitter, JSON-LD, sitemap/robots, staging noindex (`➖` for portal-only).
6. **Responsive gate** — desktop + mobile; no horizontal overflow; mobile menu works.
7. **Design-quality self-check** — matches `/mockups` + §8; **no dark mode / no theme toggle; skeleton loaders only, no preloaders**.
8. **Asset self-check** — every image referenced from `public/assets/`; no base64/hotlinks; `alt` present.
9. **Portal-data self-check** — any dynamic content introduced is DB-backed and editable in the portal (`➖` if N/A).
10. **Accountability** — admin mutations write AuditLog (`➖` if N/A). 11. **Privacy** — PII auth-protected, not logged. 12. `.env.example` updated; **no secrets** committed.

## 11. BUILD ORDER (canonical — continuous numbering; phase = label)

> One step at a time, foundation first. Each step has a `specs/NN-slug.md`. **Every completed step ends with a soft `[CHECKPOINT]`** — the single human gate is production launch (final step).
>
> **Completion rule (controller MUST honour — read this before choosing the next step):**
> - The build is complete **ONLY when step `41-production-launch` is DONE**. Nothing before 41 ends the build.
> - After finishing step N, the next step is **always N+1 in this list**. Never infer the next step by scanning or sorting the `/specs` folder.
> - **Step 25 is SUPERSEDED and its spec file is deleted.** Production launch is now step **41**. If any state file, cached context, or leftover file points at `25-production-launch`, that is stale — ignore it and continue from the highest completed step in the 26–41 range.
> - Steps **01–24 are COMPLETE**. The live build order continues at **26**.
> - PROGRESS.md must always name an explicit `Current build-step` and a `Next` pointer naming the next spec (e.g. `27 — asset-system`). A vague, empty, or "complete" pointer before step 41 is a bug, not a stop signal.

**FOUNDATION (platform base) — COMPLETE**
1. `01-foundation-nextjs` · 2. `02-design-system` · 3. `03-seo-core` · 4. `04-content-model` · 5. `05-admin-auth`

**PHASE 1 — PUBLIC WEBSITE (v1) — COMPLETE**
6. `06-home` · 7. `07-about` · 8. `08-campuses` · 9. `09-curriculum` · 10. `10-admissions` · 11. `11-admission-form` · 12. `12-gallery` · 13. `13-contact` · 14. `14-legal` · 15. `15-supporting-pages`

**PHASE 2 — ADMIN PORTAL — COMPLETE**
16. `16-admin-dashboard` · 17. `17-admin-applications` · 18. `18-admin-inquiries` · 19. `19-admin-campuses` · 20. `20-admin-curriculum` · 21. `21-admin-gallery` · 22. `22-admin-testimonials` · 23. `23-admin-settings`

**PHASE 3 — PRE-LAUNCH — COMPLETE / SUPERSEDED**
24. `24-prelaunch-seo-audit` — complete (re-run at step 40).
25. `25-production-launch` — **superseded**; production launch now happens at step **41**.

**PHASE 4 — DESIGN REBUILD & FINALISATION (current)**
26. `26-design-system-v2` — Navy-dominant tokens, gradient section blending, hover fixes, ghost-button borders, **remove dark mode + theme toggle**, **skeleton-loaders-only** rule.
27. `27-asset-system` — Every image/icon/logo/**favicon** under `public/assets/`, swappable; `ASSETS.md` manifest; favicon set from the navy logo.
28. `28-shared-components` — Header (no theme toggle), footer (no Campuses/Privacy/Terms/WhatsApp icon), buttons, cards, skeletons, floating WhatsApp button.
29. `29-routing-cleanup` — Campus detail **retained**; **curriculum detail removed**; **legal pages removed**; redirects, nav, sitemap updated.
30. `30-content-models-v2` — New **AdmissionDate** + **FAQ** models (+ permissions, migrations, clean seed copy).
31. `31-home-rebuild` — per `/mockups/homepage.html`.
32. `32-about-rebuild` — per `/mockups/about.html`.
33. `33-admissions-rebuild` — per `/mockups/admissions.html` (portal-driven Key Dates + FAQs).
34. `34-admission-form-rebuild` — per `/mockups/admission-form.html` (+ thank-you; no consent checkbox).
35. `35-campuses-rebuild` — per `/mockups/campuses.html` + `/mockups/campus-detail.html`.
36. `36-curriculum-rebuild` — per `/mockups/curriculum.html` (listing only).
37. `37-gallery-contact-rebuild` — per `/mockups/gallery.html` + `/mockups/contact.html`.
38. `38-smtp-email` — Real SMTP (nodemailer) from `.env`, replacing the console stub; staff notifications + submitter acknowledgements.
39. `39-portal-finalization` — Admin screens for Admission Dates + FAQs; light-only portal; skeletons; verify EVERY dynamic surface is portal-editable; audit coverage.

**PHASE 5 — LAUNCH**
40. `40-prelaunch-audit-v2` — Full-site SEO/performance/a11y/asset/portal audit on staging; fix code-fixable findings.
41. `41-production-launch` — Production config, DNS/SSL, indexable flip, sitemap submission, analytics live, smoke tests. **Human-gated (owner Telegram command only).**

> **Project scope ends at step 41.** One-time build: launch → debug → handover. Future features are a separate engagement.

## 12. Deployment (deploy.sh spec for THIS stack)

> `deploy.sh` lives in-repo and only **pulls/resets an existing clone** — the repo is cloned onto the server ONCE before the first deploy.

**deploy.sh must:**
1. **Node detection** — check `nvm` (`~/.nvm/versions/node/*/bin`), **aaPanel** (`/www/server/nodejs/*/bin`), then system paths (`/usr/local/bin`, `/usr/bin`). Export to `PATH`; fail loudly if none found.
2. **Pull/reset** the existing clone on the target branch (never clone; never touch `main`).
3. **Install with dev deps** — `pnpm install --frozen-lockfile` (build tooling lives in devDependencies; a production-only install breaks the build).
4. **Generate ORM client** — `pnpm prisma generate` **after install, before typecheck/build**, in BOTH CI and deploy.sh.
5. **Migrate** — `pnpm prisma migrate deploy`. The dangerous-command fence blocks `production`/`prod`, `drop database`, `rm -rf /`, force-push.
6. **Build** — set a fresh `BUILD_ID`; run the Next.js production build.
7. **Restart** — `pm2 restart` (NOT reload) via in-repo `ecosystem.config.js`.
8. **Graceful health-check** — poll `GET /api/health`; on failure the controller classifies fix vs `[HUMAN_REQUIRED]`.
9. **Secrets** — from server `.env` (never in repo). Prefer **alphanumeric DB passwords** (no `$`, backtick, quotes).
10. **aaPanel paths** — psql/node/nginx under `/www/server/...`, not system PATH.
11. **Reverse proxy** — Nginx `location /` proxies to the PM2 app port with `Host`, `X-Real-IP`, `X-Forwarded-For`, `X-Forwarded-Proto`, and WebSocket upgrade headers. Static-file/PHP blocks must NOT intercept app routes or `/assets`.
12. **Seed is NOT run by deploy.sh** — seeding is a deliberate manual step (`pnpm prisma db seed`) so content is never reset by a deploy.

**SEO deploy rules:** staging `APP_ENV=staging` → global `noindex` + `robots.txt Disallow: /` (fail-closed if env missing/other). Production `APP_ENV=production` → indexable, sitemap submitted.

**Screenshot token (staging-only, fail-closed):** `?screenshot_token=<token>` grants read-only preview of authenticated `/admin` pages **only** when `APP_ENV` is exactly `staging`. Never active in production; never hardcoded; `APP_ENV` documented in `.env.example`.

**Public-repo safety:** the pipeline pushing ARCHITECTURE.md + PROGRESS.md to the public repo must verify no secrets or real hostnames are present.

---

_End ARCHITECTURE.md — detail per step in `specs/NN-slug.md`; visual reference in `/mockups/*.html` (private repo)._
