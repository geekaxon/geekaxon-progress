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

## 02 — design-system — DONE (2026-07-17)

**Work type:** FEATURE (branch feature/02-design-system)

**Goal:** Implement the finalised GSIS design system (ARCHITECTURE §8) as reusable Tailwind tokens + shared UI components, fonts, swappable public asset structure, and animation utilities.

**What changed:**
- `tailwind.config.ts`: full palette by literal name (navy #0F2A47, navy-deep #0A1E33, teal #0E8A7A, teal-bright #14A594, amber #F5A623, off-white #F7F9FB, paper #FFFFFF, slate #2B3A4A, slate-soft #5B6B7B, line #E4EAEF) + semantic CSS-var tokens (background/surface/foreground/muted/border) for light/dark. Container capped 1320px, `rounded-card` 18px, `rounded-pill`, soft layered shadow scale (soft-sm/soft/soft-lg/soft-xl/header), keyframes+animations (fade-in, fade-up, slide-in-right, shine). darkMode:'class'.
- `app/globals.css`: light + `.dark` (navy-deep surfaces) semantic token vars as RGB channels; base body/heading fonts; prefers-reduced-motion guard.
- `app/layout.tsx`: Fraunces (display) + Inter (body) via next/font, exposed as CSS vars on <html>.
- Components in `components/ui/` (+ barrel index): Button (pill, tactile lift+shadow+shine+arrow-nudge, next/link for internal href), Card (18px radius/soft shadow), Container (1320px), SectionHeading, Header (client; floating, sticky-on-scroll shadow lift, translucent blur, amber hover-underline nav, logo-only, amber Apply CTA, mobile slide-in menu with scroll-lock), Footer (floating dark navy, AboveNext credit → abovenext.com), Tabs (client), Carousel/Slider (client), AnimatedCounter (client; IO count-up, reduced-motion aware), Reveal (client; scroll-reveal), WhatsAppButton (floating wa.me), ClayIcon (renders /assets PNG), Skeleton, ThemeToggle (demo dark toggle).
- Animation utils: `lib/hooks/useInView.ts` (IntersectionObserver, reduced-motion + SSR fallback to visible). `lib/cn.ts` dependency-free classname joiner.
- Assets: generated `public/assets/` placeholders with stable filenames — logo.png, val-1..5/why-1..4/prog-1..5 clay PNGs, why.jpg, hero.jpg, about-1/2.jpg, avatar-1..4.jpg (photos clearly marked "PLACEHOLDER — SWAP FILE"). `public/assets/README.md` documents swap contract.
- Demo route `app/dev/ui/page.tsx`: Storybook-style showcase of every component/token; noindex metadata + notFound() in production (staging-only, unlinked).
- Tests: added Vitest + jsdom + Testing Library harness (`vitest.config.ts`, `vitest.setup.ts`, next/image + next/link test stubs). `components/ui/__tests__/components.test.tsx` — 16 render/behaviour tests. `test:unit` now runs `vitest run`.

**Decisions:**
- Dependency-free `cn` (no clsx/tailwind-merge) to keep the kit lean.
- Semantic tokens via CSS vars so dark mode is a single `.dark` class swap.
- Button routes internal hrefs through next/link, external/protocol links stay <a> (avoids no-html-link-for-pages).
- Real photos not available → all assets are labelled placeholders with stable names; swapping a file needs no code change.

**Gates:** typecheck ✔ · lint ✔ (0 warnings/errors) · test:unit 16/16 (1 suite; +16) ✔ · build ➖ (controller runs) · responsive ✔ (md breakpoints, mobile menu) · design ✔ (palette/type/radius per §8) · SEO ➖ · audit ➖ · privacy ➖ · env ✔.

**Notes:** next/font fetches Fraunces/Inter at build (network needed). /dev/ui must stay unlinked + noindex.

## 03 — seo-core — DONE (2026-07-17)

Built the site-wide SEO framework (spec 03). Framework only, no page content.

Decisions:
- Env additions (lib/env.ts, all fail-closed / placeholder-safe): SITE_URL (from
  NEXT_PUBLIC_SITE_URL, trailing slash stripped, defaults http://localhost:3000),
  ANALYTICS_SRC (NEXT_PUBLIC_ANALYTICS_SRC, provider-agnostic script URL),
  SEARCH_CONSOLE_VERIFICATION (server-side token). No IDs/tokens committed —
  .env.example placeholders only.
- lib/seo.ts: buildMetadata({title,description,path,image,type}) → complete
  Next Metadata (metadataBase, absolute templated title via pageTitle/TITLE_TEMPLATE
  "%s — GSIS", canonicalUrl, robots via robotsMeta, OpenGraph + Twitter
  summary_large_image, OG default /assets/og-default.jpg). robotsMeta() is
  fail-closed: index only when APP_ENV==="production". organizationJsonLd()
  (EducationalOrganization, ORG_NAME GreenSprings International School, foundingDate
  1992, logo, sameAs [] — socials wired from Settings later) and breadcrumbJsonLd().
- components/JsonLd.tsx: <JsonLd data> renders application/ld+json.
- components/Analytics.tsx: next/script loader, renders null unless isProduction
  && ANALYTICS_SRC set → off on staging/local.
- app/layout.tsx: metadata = buildMetadata() spread + title {default,template} +
  verification (google) gated on isProduction && token. Injects
  <JsonLd data={organizationJsonLd()}> and <Analytics/> site-wide.
- app/robots.ts: kept fail-closed; production now advertises sitemap
  (canonicalUrl /sitemap.xml) + host.
- app/sitemap.ts + lib/sitemap-registry.ts: static home entry + registry
  (registerSitemapProvider/collectSitemapEntries) so later steps add dynamic slugs
  without editing the route.
- next.config.ts: images deviceSizes/imageSizes + remotePatterns https ** (future
  R2 host; concrete hostnames added later).
- public/assets/og-default.jpg: placeholder (copied from hero.jpg), swappable.

Tests (test/seo.test.ts, test/robots.test.ts): buildMetadata title/description/
canonical/OG/Twitter + defaults; robotsMeta + robots() fail-closed matrix
(production/staging/missing/typo); EducationalOrganization + BreadcrumbList shape.
Env-time consts tested via vi.resetModules + vi.stubEnv dynamic re-import.

Gates: typecheck PASS, lint PASS (no warnings/errors). Did not run test:unit/build
per CLAUDE.md (controller runs full gates).

WORK TYPE: FEATURE (branch feature/03-seo-core)

## 04 — content-model — DONE (2026-07-17)

**Branch:** feature/04-content-model (WORK TYPE: FEATURE)

**What:** Authored the complete Prisma data model that every downstream content,
submission, and admin step depends on, plus the initial migration and a
sample-only seed.

**Models / schema (prisma/schema.prisma):**
- Content: Campus, Programme, GalleryAlbum + GalleryImage (cascade), Testimonial.
- Settings: effectively single-row via fixed PK default `"singleton"`; Json
  socialLinks + seoDefaults, String[] addressLines/emailRecipients, banner flags,
  updatedById FK -> User (SetNull).
- PII (not exposed by any route yet): AdmissionApplication (FKs to Programme +
  Campus, both SetNull), ContactInquiry.
- Users/audit: User (unique email, passwordHash, Role enum, lock/lastLogin
  fields), AuditLog (actorId FK -> User SetNull, entity/entityId index).
- Enums: Role (SUPER_ADMIN/ADMIN/EDITOR), ApplicationStatus
  (NEW/REVIEWED/CONTACTED/CLOSED), InquiryStatus (NEW/REPLIED/CLOSED).
- Indexes on (published, order) for content collections, (status, createdAt) for
  submissions, audit lookups.

**Migration:** Hand-authored prisma/migrations/20260717120000_init_content_model/
migration.sql (no DB reachable in the build env) + migration_lock.toml. SQL
matches the schema so `prisma migrate deploy` applies clean on staging and the
generated client matches the DB. First migration in the repo.

**Seed:** Split into prisma/seed-data.ts (PURE sample data, no client/secrets —
2 campuses, 5 programmes Starters/Movers/Flyers/Anchors/High School Programme,
2 gallery albums w/ images, 3 testimonials, single Settings row with banner
"Admissions Open 2026-27") and prisma/seed.ts (idempotent upserts; delete+recreate
for gallery images/testimonials that lack a natural key). Admin user created from
env only (SEED_ADMIN_EMAIL/PASSWORD/NAME); password hashed with Node scrypt in
`scrypt$<saltHex>$<hashHex>` format for the step-05 auth layer to verify. No real
PII in seed — all copy marked sample placeholder, contact emails @example.com.

**Wiring:** package.json gains `prisma.seed` = `tsx prisma/seed.ts`, a
`prisma:seed` script, and tsx devDependency. .env.example gains SEED_ADMIN_*
placeholders (commented, skips admin if unset). Prisma warns package.json#prisma
is deprecated in favour of prisma.config.ts (v7) — still valid in v6.

**Tests:** test/seed.test.ts asserts seed integrity without a DB — campus/
programme/testimonial counts, unique human-readable slugs, banner enabled, images
present+ordered, and the "no real PII (@example.com only)" rule.

**Gates:** `prisma generate` OK (client v6.19). `pnpm lint` clean. `pnpm typecheck`
clean. Did not run build/test:unit/e2e per build-loop rules (controller runs full
gates). Privacy: PII models defined, not exposed by any route. Env: only
placeholders committed.

## 05 — admin-auth — DONE (2026-07-17)

Secured the `/admin` portal per specs/05-admin-auth.md. WORK TYPE: FEATURE (branch feature/05-admin-auth).

**Auth & sessions**
- Password hashing: Node scrypt, format `scrypt$salt$hash` — identical to the step-04 seed so seeded users authenticate without re-hash (lib/auth/password.ts).
- JWT: hand-rolled HS256 on Web Crypto (`crypto.subtle`) so the SAME code runs in Edge middleware and Node route handlers, zero deps (lib/auth/jwt.ts). Short-lived access (15m) + rotating refresh (7d).
- Session cookies: httpOnly, sameSite=lax, secure off only in local dev; names/TTLs/options shared via lib/auth/cookies.ts. Server helpers establish/destroy/read via next/headers (lib/auth/session.ts).

**Route protection (middleware.ts, Edge)**
- Matcher `/admin/:path*`. Valid access → allow; expired access + valid refresh → rotate BOTH tokens (response + forwarded request cookies) → allow; neither → redirect to /admin/login?next=…. Authed users are bounced off the login page. Public exceptions: /admin/login, /admin/forgot-password, /admin/reset-password, /admin/api/auth/*. Public site untouched.

**RBAC**
- Code catalog + role→permission map (lib/auth/permissions.ts): keys applications.view, inquiries.view, campuses.edit, curriculum.edit, gallery.edit, testimonials.edit, settings.edit, users.manage. EDITOR = content only; ADMIN/SUPER_ADMIN = all (SUPER_ADMIN break-glass). Guards requireUser/requirePermission/hasPermission + ForbiddenError (lib/auth/guard.ts). Catalog reconciled into new Permission table on boot via instrumentation.ts (Node runtime only, best-effort).
- Demonstrated by /admin (dashboard), /admin/settings (settings.edit), /admin/users (users.manage) — EDITOR sees inline Forbidden panel.

**Security**
- Lockout: pure policy (lib/auth/lockout.ts) — 5 consecutive failures → 15-min lock via failedLoginCount/lockedUntil; login route applies it and resets on success.
- Password reset: forgot-password issues a single-use SHA-256-hashed, 1h token (new PasswordResetToken model), emails link via console sink (lib/auth/email.ts), generic response (no enumeration). reset-password consumes token in a transaction, clears lockout. change-password verifies current + re-issues session.
- Audit base: audit() helper (lib/auth/audit.ts) writes AuditLog with actor + ip (x-forwarded-for) for every auth event (login success/failed/locked, logout, reset requested/done, password changed).
- Screenshot preview: pure fail-closed check (lib/auth/screenshot.ts) — grants read-only /admin only when APP_ENV is EXACTLY "staging", token matches SCREENSHOT_TOKEN, and method GET/HEAD. Middleware sets a preview header (stripping any forged incoming value); getCurrentUser returns a synthetic read-only viewer so pages render for screenshots. OFF in production / missing env.

**Schema/migration**: added Permission + PasswordResetToken models and PasswordResetToken FK to User; migration 20260717130000_admin_auth. env.example already carried JWT_SECRET + SCREENSHOT_TOKEN placeholders; lib/env.ts now exports JWT_SECRET (weak dev fallback), RAW_APP_ENV, SCREENSHOT_TOKEN.

**Tests** (test/auth-*.test.ts): password round-trip/malformed; JWT sign/verify/expiry/tamper/wrong-secret (node env); RBAC matrix + catalog keys; lockout threshold/isLocked; screenshot env logic (staging/prod/missing/non-GET/mismatch).

**Gates**: `pnpm prisma generate` ✔ · `pnpm lint` ✔ (no warnings) · `pnpm typecheck` ✔. Did not run build/test:unit/e2e per CLAUDE.md (controller runs full gates).

**Decisions**: no external auth/JWT/bcrypt deps — Web Crypto + Node scrypt keep the bundle lean and Edge-compatible. Refresh rotation is stateless (no session store) — acceptable for this admin surface. Preview viewer granted ADMIN-level READ (mutations are POST and structurally blocked) so staging screenshots capture all screens; staging holds only sample data.

## 06 — home — DONE (2026-07-17)

First public page. Server-component homepage with ISR (`revalidate = 300`).

Data wiring: new `lib/home/data.ts#getHomeData()` batches published campuses,
programmes, gallery albums, testimonials and the Settings singleton in one
`Promise.all`, mapped to view models. Wrapped in try/catch that returns safe
defaults (empty collections + neutral settings) so an unseeded/unreachable DB
never fails build/prerender — real data flows in on the next revalidation.
`lib/home/social.ts#parseSocialLinks()` normalises the free-form
`Settings.socialLinks` JSON into an ordered, validated list of known platforms
(http(s)-only, `x`→twitter alias, unknown keys dropped).

Sections (components/home/, spec order): AdmissionsBanner (dismissible, shown
only when `admissionsBannerOn` + text), TopBar (contact + socials from
Settings), reused Header, Hero (single `<h1>` "Education Above and Beyond",
amber underline, floating cards, trust avatars, hero.jpg), ValuesSection
(val-1..5 clay icons), AboutPreview (about-1/2 overlap + 1992 chip),
ProgrammesSection (client Tabs, prog-1..5 icons, DB-driven), CampusesSection
(premium cards, DB-driven), WhyChooseSection (why-1..4 + why.jpg), AdmissionProcess
(5-step revealed timeline), StatsSection (glass cards + animated counters,
figures explicitly marked sample), GallerySection (client Carousel, DB albums),
TestimonialsSection (client Carousel, DB testimonials), CtaSection (teal→navy),
reused Footer + floating WhatsAppButton (number from Settings, digits-sanitised).

Graceful images: new client `components/home/MediaImage.tsx` renders next/image
`fill` and falls back to a branded gradient block on empty/missing/404 src, so
not-yet-uploaded placeholder paths degrade cleanly while the correct filenames
stay in the DB. Static /assets images (hero, about, why, avatars, clay icons)
render directly. `components/home/SocialIcon.tsx` provides inline brand glyphs.

SEO: page exports unique `buildMetadata({ path:"/", description })`; layout
already emits EducationalOrganization JSON-LD site-wide, and the page adds a new
`websiteJsonLd()` (added to lib/seo.ts) WebSite node. Single `<h1>` (Hero only);
all sections use h2/h3. Every image has alt (decorative → alt=""/aria-hidden;
dynamic → required alt / role=img aria-label fallback). OG/Twitter + robots come
from buildMetadata (staging stays noindex). Homepage "/" already in sitemap.

Tests (components/home/__tests__/home.test.tsx): parseSocialLinks ordering /
alias / invalid-input; websiteJsonLd shape; AdmissionsBanner render + dismiss;
TopBar contact + socials; and DB-wiring assertions for Programmes (tab per
programme + subjects), Campuses (names/city, null when empty), Gallery (titles,
null when empty), Testimonials (quotes/authors, null when empty). Presentational
components take props only; the prisma-importing data loader is never imported
in tests (type-only imports are erased).

Gates: `pnpm typecheck` ✔ · `pnpm lint` ✔ (no warnings/errors). Did not run
build/test:unit per standing rules — controller runs full gates. Design
self-check by inspection: sections match spec §3 / home-V5 order, reduced-motion
honoured via existing Reveal/useInView/counter primitives.

Files: app/page.tsx (rewritten); lib/seo.ts (+websiteJsonLd); lib/home/data.ts,
lib/home/social.ts (new); components/home/*.tsx (new: AboutPreview,
AdmissionProcess, AdmissionsBanner, CampusesSection, CtaSection, GallerySection,
Hero, MediaImage, ProgrammesSection, SocialIcon, StatsSection,
TestimonialsSection, TopBar, ValuesSection, WhyChooseSection);
components/home/__tests__/home.test.tsx (new).

## 07 — about — DONE (2026-07-17)

Built the public About page at `/about` (feature/07-about).

Decisions:
- Route `app/about/page.tsx`, ISR `revalidate = 300`, matching the homepage chrome (AdmissionsBanner + TopBar + Header + Footer + WhatsAppButton), sourcing Settings (banner/contact/socials/whatsapp) via the existing fail-safe `getHomeData()` loader (settings only used here).
- Section order: AboutHero (single `<h1>`, story, Est. 1992 chip + overlapping about-1/about-2 images) → MissionVision (paired cards) → ValuesSection (reused from home, val-1..5 clay icons) → PrincipalMessage (placeholder portrait + pull-quote) → JourneyTimeline (new vertical scroll-reveal timeline) → StatsSection (reused, sample counters) → CtaSection (reused, Admissions CTA).
- New components under `components/about/`: AboutHero, MissionVision, PrincipalMessage, JourneyTimeline. All reuse the step-02 design system (SectionHeading, Card, Reveal, Container, Button, ClayIcon) and the step-02 Reveal for scroll-reveal + reduced-motion.
- Content discipline: all copy is static marketing sample. Principal name/photo are explicitly labelled placeholders; timeline uses only the established 1992 + "Today" as fixed and marks the rest ("Illustrative milestones — actual dates to be confirmed"); stats strip keeps its existing "sample figures" note. No real facts/stats invented.
- Placeholder images use stable filenames: /assets/about-1.jpg, /assets/about-2.jpg, /assets/hero.jpg (wash), /assets/principal.jpg (new stable name). All images carry alt text (decorative wash uses alt="").

SEO gate:
- `buildMetadata({ title: "About Us", description, path: "/about" })` → unique templated title/description, canonical, robots policy (staging noindex via env), OG + Twitter summary_large_image.
- Single `<h1>` (AboutHero only; SectionHeading renders `<h2>`).
- BreadcrumbList JSON-LD (Home → About Us) via `breadcrumbJsonLd`.
- Added `/about` to `app/sitemap.ts` static entries (changeFrequency monthly, priority 0.8).

Gates: `pnpm lint` ✔ (no warnings/errors), `pnpm typecheck` ✔ (tsc --noEmit clean). Tests skipped per spec (presentation). build/test:e2e left for the controller. Design self-check by inspection: desktop + mobile layouts use existing responsive grid patterns; timeline/counters animate and honour reduced-motion through Reveal/AnimatedCounter.

Files: app/about/page.tsx (new), components/about/{AboutHero,MissionVision,PrincipalMessage,JourneyTimeline}.tsx (new), app/sitemap.ts (edited).
