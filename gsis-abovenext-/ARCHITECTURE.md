> ⚠️ **PUBLIC FILE** — pushed to a separate public progress repo (`projects-abovenext/gsis-website-progress`). **NO secrets, hostnames, real subdomains, API endpoints, credentials, or keys — ever.** Placeholders only.

# ARCHITECTURE.md — GSIS Website & Admin Portal

> **This is the lean, always-loaded "table of contents".** It holds the overview, principles, stack, data-model summary, design tokens, the **canonical numbered build order**, the SEO spec, and the deployment spec. The DETAIL for each build-step lives in its own file at `specs/NN-slug.md` (private repo only) and is **authoritative** for that step. If a step's spec is missing/empty, the agent STOPS with `[HUMAN_REQUIRED]` rather than guessing.
>
> **Numbering (§ canonical):** build-step numbers here are the single source of truth. "Module N" / "build-step N" = the entry numbered N in §11 BUILD ORDER. Numbers are **continuous 1→N across all phases**; the phase is only a label.

---

## 1. Overview

**GSIS Website & Admin Portal** is a premium, SEO-optimised marketing website for GSIS (an international school in Pakistan, established 1992, tagline *"Education Above and Beyond"*), plus a private admin portal that lets non-technical school staff manage the site's dynamic content and view submissions.

- **Product:** GSIS Website + Admin Portal. **Built & delivered by:** AboveNext. **Client:** GSIS.
- **Two surfaces:** (a) a **public marketing website** (Home, About, Admissions, Online Admission Form, Campuses, Curriculum, Gallery, Contact, Legal), and (b) a **login-only Admin Portal** (dashboard, admission applications, contact inquiries, and management of campuses / curriculum / gallery / testimonials / settings).
- **Audience:** prospective parents & students (public site) and GSIS staff (portal).
- **Core job-to-be-done:** give the school a **modern, fast, search-optimised web presence** that visitors trust, and give **non-technical staff simple control** over dynamic content and enquiries — with automated email notifications on form submissions.
- **Scope guardrails:** this is a marketing site + light CMS/admin. It is **NOT** a student portal, fee/payment system, LMS, or ERP. No online payments, no student logins, no grades.

## 2. Principles (non-negotiable, apply to every step)

1. **SEO is a first-class requirement, not an afterthought.** Every public page ships correct metadata, semantic HTML, clean URLs, Open Graph/Twitter cards, structured data (JSON-LD), sitemap + robots, and fast Core Web Vitals. SEO is a hard gate (§9, §10).
2. **Design quality is a requirement, not a polish step.** Every UI ships the finalised GSIS design system (§8 tokens) — polished, spacious, animated, responsive (desktop + mobile), never raw/unstyled HTML. The finalised homepage is the visual reference.
3. **Content is data, not code.** All dynamic content (campuses, programmes, gallery, testimonials, settings, the "Admissions Open" banner, social links) is stored in the database and editable from the Admin Portal — never hardcoded.
4. **Non-technical-friendly admin.** The portal uses plain language, few clicks, large controls, clear empty/loading states. A staff member with no technical skill can manage content confidently. ("Simplicity first" = ease of use, NOT a basic look.)
5. **Swappable assets.** All images live under a fixed `public/assets/` structure with stable, human-readable names; replacing a file updates the site with no code change. (Design-stage convention carried into build.)
6. **No secrets in the repo.** Only `.env.example` with placeholders. Real values live on the server / GitHub Secrets. ARCHITECTURE.md + PROGRESS.md are PUBLIC — placeholders only.
7. **Staging first; production is human-gated.** The agent works on `staging` with dummy data only. **Staging is hidden from search engines** (noindex + robots disallow). `main` (production) deploys ONLY on the owner's explicit Telegram command, and production is search-visible.
8. **Data privacy.** Admission-form and contact submissions contain personal data (names, contact details, child info). They are stored securely, access is limited to authenticated admin users, and staging uses dummy data only.
9. **Specs are authoritative.** The agent treats the current `specs/NN-*.md` as the source of truth for that step; missing spec → `[HUMAN_REQUIRED]`.
10. **Accountability on admin actions.** Every create/update/delete in the portal is attributed to the logged-in user with a timestamp (audit trail on content + settings changes).

## 3. Tech stack

- **Framework:** Next.js 15 (App Router) + React + TypeScript. Server components for SEO-critical public pages; SSG/ISR where possible for speed.
- **Styling:** Tailwind CSS + the GSIS design system (§8). Font pairing: Fraunces (display serif) + Inter (body/UI). Icons via a licensed icon set (e.g. Font Awesome / Lucide).
- **ORM/DB:** Prisma + PostgreSQL 16. `prisma generate` runs after install, before typecheck/build, in CI **and** deploy.sh.
- **Auth (portal only):** email + password + JWT (short-lived access + rotating refresh); session timeout; failed-login lockout; password reset. Public site needs no auth.
- **Email:** provider-agnostic transactional email engine (e.g. Resend / SMTP) for admission-form and contact-form notifications (to staff) and acknowledgements (to submitter). Provider is `[DECIDE AT BUILD]`.
- **Files/images:** stored under `public/assets/` (swappable convention) at launch; optional object storage (Cloudflare R2) for admin-uploaded gallery/testimonial images is `[DECIDE AT BUILD]`.
- **Media handling:** Next.js `<Image>` for automatic optimisation (responsive sizes, lazy-loading, modern formats) — important for Core Web Vitals / SEO.
- **Cache:** Next.js caching + ISR; Redis optional later, not required at launch.
- **Infra:** AboveNext Oracle Cloud ARM VPS, aaPanel, Nginx (reverse proxy), PM2 (process mgr), Node v22, Cloudflare DNS. Staging + human-gated production.
- **Analytics:** privacy-friendly analytics + Google Search Console verification on production. `[DECIDE AT BUILD]` exact provider.

## 4. Domains (subdomain placeholders — never commit real hosts)

- Production: `<PROD_APP_HOST>` (placeholder for the production website host).
- Staging: `<STAGING_APP_HOST>` (placeholder for the staging host — **noindex, robots-disallowed**).
- Email sending domain, analytics IDs, etc. → placeholders only; real values in server `.env` / GitHub Secrets.

## 5. Repos & branch flow

- **Private code:** `projects-abovenext/gsis-website` (contains `/specs/NN-slug.md`).
- **Public progress:** `projects-abovenext/gsis-website-progress` (ONLY `ARCHITECTURE.md` + `PROGRESS.md`).
- **Branches:** `feat/<NN-slug>` → merge to **`staging`** (auto-deploys to staging) → **`main`** (production; human-gated, owner Telegram command only). Branches may stack on an unmerged predecessor.

## 6. Roles (portal RBAC — simple, catalog-driven)

Two roles at launch, enforced via a guard + permission catalog:
- **`ADMIN`** (GSIS staff): full access — view submissions, manage all content + settings, manage editors.
- **`EDITOR`** (optional): manage content (campuses, curriculum, gallery, testimonials) but **not** settings or user management.
- **`SUPER_ADMIN`** (AboveNext): platform-level break-glass access for support.

Permissions are catalog-driven string keys (e.g. `applications.view`, `campuses.edit`, `settings.edit`), seeded + reconciled on boot. Custom roles deferred (not needed at launch).

## 7. Auth (portal)

- **Staff:** email + password + JWT (short-lived access + rotating refresh). Optional 2FA for `ADMIN` (`[DECIDE AT BUILD]`).
- Session timeout, failed-login lockout, password reset via email. All auth events audited.
- The **public website requires no authentication**; only `/admin/*` routes are protected.

## 8. Design tokens (the finalised GSIS system — follow exactly)

The finalised homepage (`home-V5`) is the authoritative visual reference. Tokens:

- **Deep navy** `#0F2A47` (primary), **Navy deep** `#0A1E33` (dark surfaces/footer), **Primary teal** `#0E8A7A` (secondary), **Bright teal** `#14A594`, **Warm amber** `#F5A623` (accent, sparingly), **Off-white** `#F7F9FB` (backgrounds), **Paper** `#FFFFFF`, **Slate** `#2B3A4A` (body text), **Slate-soft** `#5B6B7B`, **Line** `#E4EAEF` (borders).
- **Type:** Fraunces (display serif, headings) + Inter (body/UI).
- **Components:** ~18px rounded cards, pill buttons with tactile hover (lift + shadow + shine + arrow-nudge), floating rounded header (~18px radius) sticky-on-scroll with amber hover-underline nav (logo-only, amber "Apply for Admission" button), floating footer, soft layered shadows, scroll-reveal animations, animated counters, tabbed panels, sliders/carousels, floating WhatsApp button.
- **3D icons:** soft matte-clay icon set (values / why-choose / programmes) in the palette, on transparent PNGs, under `public/assets/`. 14 icons total, already generated (`val-1..5`, `why-1..4`, `prog-1..5`).
- **Branding:** GSIS crest logo (`logo.png`, header + footer); "Built with ♥ by AboveNext" credit in the footer linking to abovenext.com (plain link, no UTM/tracking).
- **Responsive:** desktop + mobile, container max-width ~1320px; mobile uses slide-in menu, tighter spacing.
- **Assets note:** logo + all 14 3D icons + `why.jpg` are present. Real photography (`hero.jpg`, `about-1.jpg`, `about-2.jpg`, `avatar-1..4.jpg`) is client-supplied later; until then use clearly-marked placeholders with correct filenames so the swap needs no code change.
- Where a spec specifies design detail, follow it exactly.

## 9. SEO spec (hard gate on every public page)

Every public build-step must satisfy:
1. **Metadata:** unique `<title>` + meta description per page (via Next.js Metadata API).
2. **Semantic HTML:** correct heading hierarchy (one `<h1>`), landmarks, descriptive `alt` on every image.
3. **Clean URLs:** human-readable slugs (`/campuses/gsis-primary`, `/curriculum/starters`), no query-string IDs.
4. **Social cards:** Open Graph + Twitter card tags with a share image per page.
5. **Structured data (JSON-LD):** `EducationalOrganization` (site-wide), `BreadcrumbList`, and per-page types where relevant.
6. **Sitemap + robots:** auto-generated `sitemap.xml` + `robots.txt`. **Staging = noindex + Disallow all; production = indexable.**
7. **Performance / Core Web Vitals:** optimised images (`next/image`), lazy-loading, minimal blocking JS, good LCP/CLS/INP.
8. **Canonical URLs** + correct handling of the two-tab Legal page (`/legal/privacy`, `/legal/terms` as distinct indexable URLs).
9. **Accessibility baseline** (contrast, focus states, keyboard nav) — supports SEO and reach.

## 10. Verification gates (every step's definition of done)

A step is **DONE** only when ALL hold (recorded in PROGRESS-HISTORY.md):
1. `typecheck` green.
2. `lint` clean.
3. `build` green (Next.js production build).
4. Tests green (unit/integration where logic exists), with **new tests added** for the step's logic; report `N/N (S suites; +X)`. Mark `➖` for pure-presentation steps.
5. **SEO gate (public pages):** metadata, semantic HTML, alt text, clean slug, OG/Twitter, JSON-LD, sitemap/robots entry present and correct; staging noindex verified. Mark `➖` for portal-only steps.
6. **Responsive gate:** desktop + mobile both correct; no horizontal overflow; mobile menu works.
7. **Design-quality self-check:** matches the finalised GSIS design system (§8) — polished, not raw HTML.
8. **Accountability self-check (portal):** admin create/update/delete writes an AuditLog entry where applicable. Mark `➖` if N/A.
9. **Privacy self-check:** any new PII surface is auth-protected; staging uses dummy data; no PII in logs.
10. `.env.example` updated (placeholders) if new config introduced; **no secrets** committed.

## 11. BUILD ORDER (canonical — continuous numbering; phase = label)

> The agent builds strictly in this order, one step at a time, foundation first. Each step has a `specs/NN-slug.md`. **Every completed step ends with a soft `[CHECKPOINT]`** — the single human gate is production launch (step 25).

**FOUNDATION (platform base)**
1. `01-foundation-nextjs` — Next.js 15 + TS + Tailwind app, Prisma base, DB connection, health endpoint, base config, deploy.sh.
2. `02-design-system` — Tailwind design tokens (§8), shared UI components (buttons, cards, header, footer, nav, WhatsApp button, 3D-icon component), fonts, `public/assets/` structure with placeholder assets, reveal/animation utilities.
3. `03-seo-core` — Metadata framework, JSON-LD `EducationalOrganization`, sitemap.xml + robots.txt (staging noindex / production indexable), OG/Twitter defaults, `next/image` config, analytics + Search Console scaffolding.
4. `04-content-model` — Prisma schema for all dynamic content (Campus, Programme, GalleryAlbum/Image, Testimonial, Settings, AdmissionApplication, ContactInquiry, User, AuditLog) + seed with sample data.
5. `05-admin-auth` — Portal auth (email+password+JWT, refresh, lockout, reset), `/admin` route protection, RBAC guard + permission catalog, AuditLog base.

**PHASE 1 — PUBLIC WEBSITE**
6. `06-home` — Homepage (all sections from the finalised design), pulling dynamic bits (campuses, programmes, testimonials, banner, socials) from the DB.
7. `07-about` — About page (story, mission/vision/values, principal's message, journey timeline, CTA).
8. `08-campuses` — Campuses listing + Campus detail template (dynamic, slug-based).
9. `09-curriculum` — Curriculum listing + Programme detail template (Starters/Movers/Flyers/Anchors/High School; dynamic, slug-based).
10. `10-admissions` — Admissions info page (process, requirements, FAQs) + CTA into the form.
11. `11-admission-form` — Online Admission Form (multi-field, validation, spam protection, DB save, email to staff + acknowledgement to submitter, thank-you page).
12. `12-gallery` — Gallery albums listing + Gallery detail (album lightbox), dynamic from DB.
13. `13-contact` — Contact page (info, map embed, contact form → DB + email, socials), thank-you state.
14. `14-legal` — Single Legal page with two tabs (Privacy `/legal/privacy`, Terms `/legal/terms`) as distinct indexable URLs, deep-linkable.
15. `15-supporting-pages` — Thank-you page(s), 404, and any shared states; final SEO sweep across public site.

**PHASE 2 — ADMIN PORTAL**
16. `16-admin-dashboard` — Portal dashboard: key counts (new applications, new inquiries), recent activity, quick links; animated cards, skeletons.
17. `17-admin-applications` — Admission applications: list (filter/search/status), detail view, status change, export, mark read; all actions audited.
18. `18-admin-inquiries` — Contact inquiries: list, detail, status, reply-via-email link, export; audited.
19. `19-admin-campuses` — Manage campuses (CRUD, images, slug, publish/unpublish); reflects live on public site.
20. `20-admin-curriculum` — Manage programmes (CRUD, subjects, images, slug, order, publish).
21. `21-admin-gallery` — Manage gallery albums + images (CRUD, upload, reorder, publish).
22. `22-admin-testimonials` — Manage testimonials (CRUD, parent name/role/photo, publish, order).
23. `23-admin-settings` — Site settings: contact info, social links, WhatsApp number, "Admissions Open" banner (on/off + year range), SEO defaults, email-recipient config; all audited.

**PHASE 3 — LAUNCH**
24. `24-prelaunch-seo-audit` — Full-site SEO + performance audit (Lighthouse/Core Web Vitals), structured-data validation, sitemap submission readiness, broken-link + alt-text sweep, image optimisation pass.
25. `25-production-launch` — Production config, DNS/SSL, switch production to indexable + submit sitemap to Search Console, analytics live, final smoke test. **Human-gated (owner Telegram command only).**

> **Project scope ends at step 25.** This is a one-time build: after production launch the project moves to debug + handover, then delivery is complete. Any future features (e.g. blog/news, newsletter, multi-school) are out of scope and would be a separate, future engagement — not part of this build order.

## 12. Deployment (deploy.sh spec for THIS stack)

> The agent authors the real `deploy.sh` in-repo during build-step 01 per this spec, and extends it as services are added. `deploy.sh` only **pulls/resets an existing clone** — the repo is cloned onto the server ONCE before the first deploy.

**deploy.sh must:**
1. **Node detection block** — locate Node by checking, in order: `nvm` (`~/.nvm/versions/node/*/bin`), **aaPanel** (`/www/server/nodejs/*/bin`), then system paths (`/usr/local/bin`, `/usr/bin`). Export the resolved bin to `PATH`. Fail loudly if none found.
2. **Pull/reset** the existing clone on the target branch (never clone; never touch `main` automatically).
3. **Install with dev deps:** `npm ci --include=dev` — build tooling (next/tsc/prisma) lives in devDependencies; a production-only install breaks the build with "next/tsc/turbo: not found".
4. **Generate ORM client:** run `npx prisma generate` **after install, before typecheck/build**, in BOTH CI and deploy.sh (skipping it makes real builds fail with "has no exported member" even when mocked unit tests pass).
5. **Migrate:** `npx prisma migrate deploy` (staging). The dangerous-command fence blocks anything matching `production`/`prod`, `drop database`, `rm -rf /`, force-push.
6. **Build:** set a fresh `BUILD_ID` before build; run the Next.js production build.
7. **Restart services:** `pm2 restart` (NOT reload) via the in-repo `ecosystem.config.js`; set `BUILD_ID` before restart.
8. **Graceful health-check:** after restart, poll `GET /api/health`; on failure within the window, report (controller classifies fix vs `[HUMAN_REQUIRED]`).
9. **Secrets:** sourced from server `.env` (never in repo). Prefer **alphanumeric DB passwords** (no `$`, backtick, quotes) so `.env` sourcing in shell scripts doesn't break.
10. **aaPanel paths:** psql/node/nginx live under `/www/server/...`, not system PATH.
11. **Reverse-proxy mapping:** match Next.js routing; ensure `/admin` and API routes proxy correctly (if the API has no `/api` prefix, the proxy must strip it); static `public/assets` served with cache headers.

**SEO deploy rules:**
- **Staging** sets `APP_ENV=staging` → global `noindex` meta + `robots.txt` `Disallow: /` (fail-closed: if env is missing or not exactly `production`, default to noindex).
- **Production** sets `APP_ENV=production` → indexable, real `robots.txt`, sitemap submitted to Search Console.

**Screenshot token (staging-only, fail-closed):** the app accepts `?screenshot_token=<token>` granting read-only preview of authenticated `/admin` pages **only** when `APP_ENV` is exactly `staging`; missing/other → OFF. Never active in production; token never hardcoded; `APP_ENV` documented in `.env.example`.

**Public-repo safety:** the pipeline that pushes ARCHITECTURE.md + PROGRESS.md to the public repo must strip/verify no secrets or real hostnames are present.

---

_End ARCHITECTURE.md — detail per step in `specs/NN-slug.md` (private repo). This file stays lean; do not inline step detail here._
