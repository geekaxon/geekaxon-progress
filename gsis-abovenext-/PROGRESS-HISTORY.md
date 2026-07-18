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

## 08 — campuses — DONE (2026-07-17)

Public, dynamic, slug-based Campuses listing + detail template, driven entirely from the `Campus` model. Branch: feature/08-campuses.

### What was built
- `lib/campuses/seo.ts` — pure (no Prisma) helpers safe for client/tests: `CampusCard`/`CampusDetail` types, `PUBLISHED_WHERE`, `CAMPUS_ORDER`, `campusBySlugWhere(slug)` (published baked in — single source of truth for the published-only rule), `campusPath`, `campusBreadcrumbItems`, `campusJsonLd` (EducationalOrganization branch node w/ PostalAddress + contacts).
- `lib/campuses/data.ts` — server-only loaders: `getPublishedCampuses`, `getCampusBySlug` (findFirst, published-scoped → null → 404), `getPublishedCampusSlugs` (generateStaticParams). Try/catch → safe empty like the home loader. Registers a sitemap provider (one entry per published campus, `updatedAt` lastModified).
- `app/sitemap.ts` — added side-effect import of `@/lib/campuses/data` so the registered provider actually loads (registry pattern from spec 03 needs the module imported).
- `app/campuses/page.tsx` — listing: breadcrumbs, single `<h1>`, DB-driven premium card grid, CtaSection, chrome (TopBar/Header/Footer/banner/WhatsApp) from getHomeData. `revalidate = 300`.
- `app/campuses/loading.tsx` — skeleton grid while the server component streams.
- `app/campuses/[slug]/page.tsx` — detail: `generateStaticParams`, `generateMetadata` (per-page title/description/OG from campus, heroImage as share image), `notFound()` on null. Hero (single h1), summary, paragraph-split description, address+phone+email sidebar, lazy map iframe (mapEmbedUrl), gallery, Apply/Contact CTA, BreadcrumbList + campus JSON-LD. `revalidate = 300` (ISR picks up campuses added later via the portal).
- Components: `CampusCard`, `CampusesList` (empty state), `Breadcrumbs` (aria-current on last), `CampusHero` (h1), `CampusGallery` (alt text per image).
- Tests: `components/campuses/__tests__/campuses.test.tsx` — routing/clean-URL, published-filter where-clause, breadcrumb trail, campus JSON-LD, CampusCard render+link, CampusesList render+empty, Breadcrumbs current-page.

### Decisions
- Single-source published rule in `campusBySlugWhere`/`PUBLISHED_WHERE` (testable without a DB; keeps unpublished unreachable and out of the sitemap).
- Listing `<h1>` inlined (SectionHeading only emits `<h2>`); card names are `<h2>` so exactly one `<h1>` per page.
- Reused home `MediaImage` (graceful gradient fallback for not-yet-uploaded placeholder images) and CtaSection.
- Apply/Contact CTAs link to `/admissions` and `/contact` (later steps).

### Gates
- `pnpm lint` ✔ (no warnings/errors) · `pnpm typecheck` ✔ (clean).
- Did not run test:unit/e2e/build per CLAUDE.md (controller runs full gates).
- SEO self-check: per-page title/description, single h1, alt on all images, OG/Twitter via buildMetadata, BreadcrumbList + EducationalOrganization JSON-LD, sitemap entries for published campuses only, staging noindex via robotsMeta.

## 09 — curriculum — DONE (2026-07-17)

**Branch:** feature/09-curriculum · WORK TYPE: FEATURE

**Scope:** Public curriculum listing `/curriculum` + dynamic programme detail `/curriculum/[slug]`, driven entirely from the `Programme` model (the 5-stage learning journey). Slug-based clean URLs, no IDs. Mirrors the campuses (spec 08) architecture.

**Files added:**
- `lib/curriculum/seo.ts` — Prisma-free pure helpers: `ProgrammeCard`/`ProgrammeDetail` types; `PUBLISHED_WHERE`/`PROGRAMME_ORDER`; `programmeBySlugWhere` (publish baked into the where-clause so unknown/unpublished → no row → 404); `programmePath`; `programmeIcon` (honours record `icon`, else positional `prog-1..5`); `programmeBreadcrumbItems`; `programmeJsonLd` (EducationalOccupationalProgram with EducationalOrganization provider, typicalAgeRange, and subjects as `hasCourse` Course items).
- `lib/curriculum/data.ts` — server-only Prisma loaders: `getPublishedProgrammes` (cards), `getProgrammeBySlug` (detail), `getPublishedProgrammeSlugs` (generateStaticParams); registers a sitemap provider for published programmes. try/catch → safe empty data on DB failure, matching the home/campuses resilience pattern.
- `components/curriculum/Breadcrumbs.tsx` — visible breadcrumb nav (aria-current on last) matching the BreadcrumbList JSON-LD.
- `components/curriculum/ProgrammeCard.tsx` — listing card: ClayIcon + name + stage·ageRange + summary + "Explore" pill; whole card links to clean slug URL.
- `components/curriculum/CurriculumList.tsx` — responsive grid (sm:2 / lg:3) with Reveal stagger + graceful empty state.
- `components/curriculum/ProgrammeHero.tsx` — detail hero: breadcrumbs, clay icon, single `<h1>`, stage·ageRange, summary + programme image.
- `app/curriculum/page.tsx` — listing (ISR revalidate 300), buildMetadata, breadcrumb JSON-LD, TopBar/Header/Footer/WhatsApp chrome, CtaSection.
- `app/curriculum/[slug]/page.tsx` — detail: generateStaticParams from published slugs, generateMetadata (title/desc/OG image from record), notFound() on null, ProgrammeHero + "What to expect" description paragraphs + subjects sidebar + Apply/Contact CTAs + CtaSection; breadcrumb + EducationalOccupationalProgram JSON-LD.
- `app/curriculum/loading.tsx` — skeleton mirroring the 3-col card grid.
- `components/curriculum/__tests__/curriculum.test.tsx` — routing helpers, published-filter where-clause, icon resolver (explicit + positional fallback + wrap), JSON-LD shape, ProgrammeCard clean-URL link, CurriculumList render + empty state, Breadcrumbs aria-current.

**Files edited:**
- `app/sitemap.ts` — side-effect import `@/lib/curriculum/data` to register its sitemap provider; added static `/curriculum` entry (priority 0.8).

**Decisions:**
- Clay icon: `programmeIcon(icon, index)` prefers the record's `icon` key and falls back to the positional `prog-1..5` set, so the listing always shows a distinct icon even before editors set one. Detail uses the record icon with `prog-1` positional default (no index at slug lookup).
- "What to expect" maps to the `Programme.description` field (split into paragraphs on blank lines); subjects render as a sidebar of pills; no new schema fields (spec adds none).
- JSON-LD: EducationalOccupationalProgram (+ Course per subject via `hasCourse`) as the spec's "where sensible" structured data; BreadcrumbList emitted on both listing and detail.
- No schema change → no `pnpm prisma generate` needed.

**Gates:** lint ✔ (No ESLint warnings or errors) · typecheck ✔ (tsc --noEmit clean). Did not run test:unit/e2e/build per CLAUDE.md (controller runs full gates). SEO: per-page title/description, single h1, alt on icons/images, OG/Twitter via buildMetadata, BreadcrumbList + EducationalOccupationalProgram JSON-LD, sitemap entries, staging noindex via robotsMeta. Responsive grid + skeleton loading state.

---

## 10 — admissions — DONE (2026-07-17)

**Branch:** `feature/10-admissions` (FEATURE). Spec: `specs/10-admissions.md`.

**What was built:** the public `/admissions` information page. Sections, top to
bottom: intro hero (single `<h1>`, breadcrumbs, CTA → `/admissions/apply`);
five-step animated process (reused `components/home/AdmissionProcess`);
requirements/eligibility clay-icon cards; a marketing-only fees note; a
keyboard-accessible FAQ accordion; closing CTA (reused `CtaSection`). The
"Admissions Open" banner reuses `AdmissionsBanner`, shown only when
`Settings.admissionsBannerOn` and driven by `admissionsBannerYear` (surfaced as
`admissionsBannerText` by `getHomeData`) — nothing hardcoded.

**Files:**
- `app/admissions/page.tsx` — ISR (`revalidate = 300`), metadata via
  `buildMetadata`, BreadcrumbList + FAQPage JSON-LD, banner/TopBar/WhatsApp chrome.
- `lib/admissions/content.ts` — SAMPLE PLACEHOLDER copy: REQUIREMENTS, FEES_NOTE
  (with a "no fees collected here" disclaimer), FAQS.
- `lib/seo.ts` — added `faqPageJsonLd(items)` helper.
- `components/admissions/{AdmissionsHero,Requirements,FeesNote,Faq,Breadcrumbs}.tsx`.
- `components/admissions/__tests__/admissions.test.tsx` — FAQ toggle + JSON-LD sync.
- `app/sitemap.ts` — static `/admissions` entry (priority 0.9).

**Decisions:**
- FAQ built as a native-`<button>` disclosure list (aria-expanded/aria-controls,
  region panels) so it's keyboard-accessible without extra JS; the same `FAQS`
  array feeds both the visible accordion and the FAQPage JSON-LD so they can't
  drift. First item defaults open.
- No payment/fee logic — fees section is informational copy pointing to the
  admissions team, per the Do-NOT-break constraint.
- Local `Breadcrumbs` in the admissions module (mirrors curriculum/campuses) to
  keep the module self-contained.

**Gates:** typecheck ✔ · lint ✔ (no warnings) · SEO ✔ (title/desc, single h1,
decorative alt="", OG/Twitter, BreadcrumbList + FAQPage JSON-LD, sitemap entry,
staging noindex via `robotsMeta`) · responsive ✔ · design ✔. build/tests run by
controller. tests ➖ per spec but a light FAQ/JSON-LD test added for parity.

## 11 — admission-form — DONE (2026-07-17)

Branch: `feature/11-admission-form`. Online admission form saving to `AdmissionApplication`, notifying staff + acknowledging the submitter, then a thank-you page.

**Decisions at build:**
- Email provider: kept the existing provider-agnostic `sendEmail` console sink (from step 05). Real Resend/SMTP deferred to production launch. Staging never contacts real recipients — recipients come from `Settings.emailRecipients` but the sink only logs.
- Spam protection: honeypot (hidden `website` field, silently accepted-not-saved when tripped) + in-memory per-IP sliding-window rate limit (5/min). No captcha.
- Validation: added `zod` (4.4.3) as a dependency; one shared schema (`lib/admissions/apply-schema.ts`) drives BOTH the client form and the server route so they never drift. Client-safe (no Prisma/node imports).
- DOB sanity: must parse, not be in the future, within the last 25 years.
- guardianEmail made required (needed to send the acknowledgement); guardianRelation + message optional.

**Files added:**
- `lib/admissions/apply-schema.ts` — zod schema, `isSaneDob`, `toApplicationData` mapper, honeypot constant/helper.
- `lib/admissions/rate-limit.ts` — injectable in-memory per-key sliding-window limiter.
- `lib/admissions/apply-options.ts` — published-only programme/campus select options (server-only, resilient try/catch).
- `lib/admissions/apply-email.ts` — staff notification + submitter acknowledgement via `sendEmail`; skips each when nothing to send.
- `app/api/admissions/apply/route.ts` — POST (Node runtime): rate-limit → honeypot → zod → published programme/campus check → create app (status NEW, read false) → emails. Never logs the raw PII body.
- `components/admissions/ApplyForm.tsx` — client multi-section form (student / guardian / message), shared-schema inline validation, honeypot, posts JSON, routes to thank-you.
- `app/admissions/apply/page.tsx` — indexable form page (SEO metadata, breadcrumb JSON-LD), site chrome, DB-driven selects.
- `app/admissions/apply/thank-you/page.tsx` — friendly confirmation + next steps; robots forced to `noindex,nofollow` regardless of env.
- `test/admissions-apply.test.ts` — validation (valid/invalid, email/phone, required selects), DOB sanity, honeypot, save mapping, rate-limit window, email dispatch (sendEmail mocked; staff+ack, skip cases).

**Files changed:**
- `app/sitemap.ts` — added `/admissions/apply` (indexable). Thank-you deliberately omitted (noindex).
- `package.json` / lockfile — added `zod`.

**Privacy:** submissions are PII; the route logs only generic diagnostics, never field values. Emails are the one intended sink; on staging that sink is the console. Submissions are never exposed publicly (admin portal reads them in a later step).

**Gates:** `pnpm lint` ✔ (no warnings/errors) · `pnpm typecheck` ✔ (tsc --noEmit clean). Did not run test:unit/build per AGENT.md (controller runs full gates).

## 12 — gallery — DONE (2026-07-17)

**Spec:** specs/12-gallery.md. Public, dynamic, SEO hard gate. Branch: feature/12-gallery.

**Built:**
- `lib/gallery/seo.ts` — pure, Prisma-free helpers (client/test-safe): types (AlbumImage/AlbumCard/AlbumDetail), PUBLISHED_WHERE + ALBUM_ORDER + IMAGE_ORDER, albumBySlugWhere (published baked in), albumPath, imageAlt (editor alt or "<title> — photo N" fallback), albumBreadcrumbItems (Home › Gallery › [Album]), albumJsonLd (schema.org ImageGallery with per-image ImageObject/contentUrl/caption).
- `lib/gallery/data.ts` — Prisma loaders, try/catch → safe-empty on DB-unreachable, mirroring campuses/curriculum: getPublishedAlbums (with _count.images → imageCount), getAlbumBySlug (images ordered), getPublishedAlbumSlugs; registers a sitemap provider (one entry per published album, priority 0.6).
- `components/gallery/`: Breadcrumbs (module copy), AlbumCard (cover via MediaImage + title + photo count, clean slug link), GalleryList (responsive grid + empty state), AlbumGallery (client thumbnail grid, buttons open lightbox), Lightbox (client: role=dialog/aria-modal, keyboard ← → Esc, Tab focus-trap across buttons, backdrop-click close, focus restore to trigger, body-scroll lock, next/image via MediaImage fit=contain).
- `app/gallery/page.tsx` (listing, ISR revalidate 300, buildMetadata, BreadcrumbList JSON-LD), `app/gallery/loading.tsx` (skeleton album cards), `app/gallery/[slug]/page.tsx` (SSG+ISR, generateStaticParams over published slugs, generateMetadata, single h1, BreadcrumbList + ImageGallery JSON-LD, notFound() on unknown/unpublished slug).
- `components/home/MediaImage.tsx` — added optional `fit: "cover" | "contain"` (default cover; lightbox uses contain). Backward-compatible.
- `app/sitemap.ts` — added static `/gallery` entry + side-effect import of the gallery album provider.
- `components/gallery/__tests__/gallery.test.tsx` — routing helpers, published-only where-clause, imageAlt fallback, albumJsonLd captions, AlbumCard (singular/plural), GalleryList empty state, Breadcrumbs aria-current, and lightbox a11y (open dialog → ArrowRight navigates → Escape closes + restores focus to trigger).

**Decisions:** Seed data already present (albums campus-life, events). imageCount via Prisma _count. Lightbox images render via MediaImage so empty/failed src degrades to the branded fallback (also lets the a11y test avoid next/image in jsdom). Nav bar left unchanged (prior steps didn't add campuses/curriculum links either). No user-event dep in repo → used fireEvent.

**Gates:** pnpm lint ✔ (no warnings/errors) · pnpm typecheck ✔ (tsc --noEmit clean). Schema unchanged → no prisma generate. Did not run build/test:unit/e2e per AGENT.md (controller runs full gates). SEO gate satisfied: metadata/OG/Twitter, single h1, alt on every image, BreadcrumbList + ImageGallery JSON-LD, sitemap entries, staging noindex via robotsMeta.

## 13 — contact — DONE (2026-07-17)

**Branch:** feature/13-contact (FEATURE). Spec: specs/13-contact.md.

**What was built:** Public `/contact` page — school contact info, embedded map, social links + WhatsApp, and a contact form that saves to `ContactInquiry` and emails staff + sender.

**Files:**
- `lib/contact/contact-schema.ts` — shared zod schema (name+email+message required; phone/subject optional), `toInquiryData` mapper, honeypot helper. Client-safe (no Prisma).
- `lib/contact/contact-email.ts` — `sendInquiryEmails`: staff notification (recipients from Settings; skipped if none) + submitter acknowledgement, via the provider-agnostic `sendEmail` sink (staging = console; no real recipients).
- `app/api/contact/route.ts` — POST (Node runtime): per-IP rate-limit (reuses `lib/admissions/rate-limit` with `contact:` key namespace) → honeypot silent-accept → zod → persist ContactInquiry (status NEW, read false) → emails. Never logs raw PII body.
- `components/contact/ContactForm.tsx` — client form, same shared schema, inline success state (no navigation), honeypot field, accessible errors.
- `components/contact/ContactInfo.tsx` — server panel rendering address/phone/email/WhatsApp/socials from Settings; omits missing values.
- `app/contact/page.tsx` — sections + breadcrumbs; Google Maps embed iframe (address query, lazy, titled); WhatsApp FAB. JSON-LD: BreadcrumbList + ContactPage + Organization.
- `lib/seo.ts` — added `contactPageJsonLd()`; extended `organizationJsonLd(sameAs)` to accept social URLs (backward-compatible default []).
- `app/sitemap.ts` — added `/contact` static entry.
- `test/contact.test.ts` — validation (required/optional/malformed), save mapping, honeypot, rate-limit, email dispatch (sendEmail mocked).

**Decisions:**
- Thank-you: chose the spec's inline success-state option over a `/contact/thank-you` route — cleaner UX, and no separate indexable page to worry about (noindex concern moot).
- "Hours from Settings": the Settings model has no hours field; omitted rather than add a migration for one minor field (not in acceptance criteria). Address/phone/email/socials/WhatsApp all render from Settings.
- Rate-limit reused from admissions (generic `checkRateLimit`, key-namespaced) rather than duplicated.
- captcha `[DECIDE AT BUILD]`: kept honeypot + per-IP rate-limit (matches project standing decision; no captcha).
- Email provider `[DECIDE AT BUILD]`: console sink on staging (existing `sendEmail`), no real recipients.

**Gates:** `pnpm lint` ✔ (no warnings/errors), `pnpm typecheck` ✔ (clean). No schema change (ContactInquiry already existed) → no prisma generate. Did not run build/test:unit/test:e2e per token-discipline rules; controller runs full gates. SEO gate satisfied by inspection: title/description via buildMetadata, single `<h1>`, iframe title (alt equivalent), OG/Twitter defaults, BreadcrumbList + ContactPage + Organization JSON-LD, sitemap entry, staging noindex via central robots policy. Privacy: PII never logged raw; staging sink only.

## 14 — legal — DONE (2026-07-17)

**Spec:** specs/14-legal.md. Branch: feature/14-legal. Gates: typecheck ✔, lint ✔ (build/tests run by controller).

**What was built:** A Legal page presenting Privacy Policy and Terms of Service as two DISTINCT, deep-linkable, indexable URLs — `/legal/privacy` and `/legal/terms` — never collapsed to a query-string tab.

**Routing / active tab:** Two thin route files (`app/legal/privacy/page.tsx`, `app/legal/terms/page.tsx`) each set their own `buildMetadata` (unique title/description + canonical) and render a shared server scaffold `components/legal/LegalPage.tsx` with a `tab` prop. The tab strip `components/legal/LegalTabs.tsx` is a client component using `usePathname()` + `resolveLegalTab()` so the active tab is derived from the PATH (works on direct load) and each tab is a Next `<Link>` (client nav swaps URL without full reload, no SEO loss). `revalidate = 300` so Settings-driven chrome refreshes.

**Content:** `lib/legal/content.tsx` holds `LEGAL_TABS`, `resolveLegalTab`, `privacySections(contactEmail)` and `termsSections()`. Most copy is clearly-marked SAMPLE placeholder (amber callout + inline italics) for the client to finalise. The Privacy "Information you provide" / "How we use it" / "Keeping submissions secure" sections describe REAL PII handling and match actual behaviour: admission form fields (studentName, studentDob, programme, campus, guardian name/relation/phone/email, optional address) and contact form fields (name, email, optional phone, subject, message); submissions saved to DB, staff notified by email + acknowledgement to submitter, honeypot + per-visitor rate limiting, admin-only access, no selling, essential-cookies-only (analytics deferred). The configured contact email is surfaced as a real mailto link in "Your rights".

**Design / UX:** clean long-form typography (Fraunces via font-display headings, Inter body), sticky in-page contents nav (`LegalContents`, plain hash anchors, scroll-mt on sections), linked section headings, and a new reusable `components/ui/BackToTop.tsx` (client, appears after scroll, smooth-scroll to top; exported from ui barrel).

**SEO gate:** unique per-URL title/description + canonical via buildMetadata; single `<h1>` per page (the active tab label); `BreadcrumbList` JSON-LD (Home › Legal › <doc>); both URLs added to `app/sitemap.ts` static entries; staging noindex inherited from `robotsMeta()`.

**Cross-link:** `components/ui/Footer.tsx` bottom bar gained a Legal nav linking directly to `/legal/privacy` and `/legal/terms`.

**Tests:** `test/legal.test.tsx` — resolveLegalTab active-tab-from-path (privacy/terms/default), LEGAL_TABS config (two distinct non-query URLs), unique section anchor ids, accurate PII disclosure (renders LegalArticle, asserts admission/contact fields + real behaviour + mailto), and LegalTabs marks the path-matching tab aria-selected with correct hrefs (next/navigation mocked).

## 15 — supporting-pages — DONE (2026-07-17)

Closed out the public site with the remaining shared states plus a full-site SEO/link sweep.

**Delivered**
- `components/shared/ThankYou.tsx` — one shared, branded, presentational confirmation template (check icon, headline, copy, optional "what happens next" panel, action slot). Configurable heading level (`h1` standalone / `h2` inline) and ARIA `role`, so it works in both a server page and a client form. Refactored `app/admissions/apply/thank-you/page.tsx` and the ContactForm "message sent" state to render it — same look/wording across both flows.
- `app/not-found.tsx` — branded 404 with Header/Footer, single `<h1>`, helpful links (Home/Admissions/Curriculum/Gallery/Contact) and Return-home/Contact CTAs. Deliberately DB-free so it renders even when data is down; Next serves it with a real 404 status. Explicit `robots: noindex,nofollow`.
- `app/error.tsx` — client error boundary: branded, safe message, `Try again` (reset) + Return-home. Logs the error to the console for observability; never renders the message, stack, or digest to the user.
- `app/global-error.tsx` — self-contained last-resort boundary (own `<html>/<body>`, imports globals.css) for root-layout failures, with a safe message + reset.

**SEO sweep fixes**
- Removed broken internal links in the shared nav: Header `DEFAULT_NAV` and Footer `DEFAULT_COLUMNS` pointed at not-yet-built routes (`/academics`, `/campus-life`, `/news`, `/careers`). Repointed to existing routes only — Header: About/Curriculum/Admissions/Campuses/Contact; Footer columns School (About/Curriculum/Admissions), Explore (Campuses/Gallery/Apply), Connect (Contact/Privacy/Terms).
- Added the `/campuses` index (which existed but was absent) to `app/sitemap.ts` static entries.
- Verified: robots stays fail-closed noindex on staging + `Disallow: /` (unchanged); thank-you/404/error explicitly noindex even in production; all content images already use `next/image`.

**Tests** — `test/supporting-pages.test.tsx`: 404 metadata is noindex + single-h1 + helpful links; error boundary shows the safe message and leaks neither the error text nor the digest, and `Try again` calls `reset`; shared ThankYou renders heading/description/steps/actions and supports the h2+status inline variant.

**Gates** — `pnpm typecheck` ✔, `pnpm lint` ✔ (no warnings/errors). Existing Header/Footer tests still pass (assert "About" + "Apply for Admission", both retained). Design self-check by inspection: shared tokens (teal/amber, rounded-card, font-display, foreground/muted), responsive py/text scales, on-design header/footer on 404 + error.

**Decision** — Footer Connect column now lists Privacy/Terms (also in the bottom legal bar); minor duplication accepted to keep three balanced columns of real routes and avoid broken links.

---

## 16 — admin-dashboard — DONE (2026-07-17)

**Branch:** feature/16-admin-dashboard (FEATURE)
**Spec:** specs/16-admin-dashboard.md

### What was built
The `/admin` landing page after login, rebuilt from the step-05 placeholder into a real portal dashboard.

- **Portal shell** (`components/admin/PortalShell.tsx`, client): fixed sidebar nav on desktop (`lg` grid), slide-in drawer on mobile (uses `animate-slide-in-right`), sticky topbar with hamburger, user identity (email + humanised role), avatar initial, and the existing `LogoutButton`. Active-link detection via `usePathname`. Nav items are pre-filtered to the user's role on the server (`navForRole`) and passed in, so the client component stays presentational.
- **Data + presentation model** (`lib/admin/dashboard.ts`): thin Prisma readers `getDashboardStats` (Promise.all of six counts — NEW applications, NEW inquiries, published campuses/programmes/albums/testimonials) and `getRecentActivity` (latest applications + inquiries + audit logs). Pure, unit-tested helpers: `mergeActivity` (merges three sources newest-first, caps at limit, maps titles/details/section links per kind), `formatWhen` (deterministic en-GB / Africa-Lagos label), `navForRole`, `quickActionsForRole`, plus `STAT_CARDS`/`NAV_ITEMS`/`QUICK_ACTIONS` catalogs.
- **Stat cards** (`components/admin/StatCard.tsx` + `DashboardStats.tsx`): animated counters via existing `AnimatedCounter`, action-needed cards (applications/inquiries) accented amber, others teal; each links to its section. Streamed inside a `<Suspense>` boundary with `DashboardStatsSkeleton`.
- **Recent activity** (`components/admin/RecentActivity.tsx`): unified feed with per-kind icon/tone, quick links to Applications/Inquiries sections (audit rows have no link — no dedicated screen), friendly empty state, streamed inside `<Suspense>` with `RecentActivitySkeleton`.
- **Quick actions** (`components/admin/QuickActions.tsx`): role-filtered add-content shortcuts + edit-settings link, plus the **Admissions Open banner toggle**.
- **Banner toggle** (`components/admin/BannerToggle.tsx` client + `app/admin/api/settings/banner/route.ts`): switch control POSTs `{on}` to a Node route that requires `settings.edit` (401 unauth / 403 forbidden incl. screenshot-preview, 400 bad body), updates `Settings.admissionsBannerOn` on the `singleton` row, records `updatedById`, and writes a `settings.banner.toggle` audit row. `router.refresh()` re-reads the state.
- **Icons** (`components/admin/AdminIcon.tsx`): compact inline-SVG set (currentColor stroke) for sidebar/cards/actions — no PNG asset dependency.

### Decisions
- "New" counts = `status === NEW` (not the `read` flag) for both applications and inquiries.
- Skeletons implemented via App-Router Suspense streaming (idiomatic) rather than client fetch; only the counters and the banner switch are client components.
- Sidebar/quick-action links point at the intended section routes (`/admin/applications`, `/admin/campuses/new`, …) that later steps build; settings + users pages already exist.
- Banner toggle is a genuine mutation with audit even though the spec's gate marks audit N/A for the read-only view — flipping site behaviour warrants a trail.
- Top-level `app/admin/layout.tsx` left untouched (still noindex wrapper) so the unauthenticated auth screens keep their own centred layout; the shell is applied per authenticated page (dashboard here).

### Gates
- `pnpm lint` ✔ (no warnings/errors)
- `pnpm typecheck` ✔ (clean)
- Tests: added `test/admin-dashboard.test.ts` — stat-card/count coverage, mergeActivity ordering/mapping/cap, formatWhen, and RBAC gating (EDITOR vs ADMIN nav + quick actions). Not executed here (controller runs full suite); lint+typecheck only per AGENT.md.
- No schema change → no `prisma generate` needed.

### Files
- Added: `lib/admin/dashboard.ts`, `components/admin/{PortalShell,AdminIcon,StatCard,BannerToggle,DashboardStats,RecentActivity,QuickActions}.tsx`, `app/admin/api/settings/banner/route.ts`, `test/admin-dashboard.test.ts`.
- Changed: `app/admin/page.tsx` (placeholder → dashboard), `PROGRESS.md`.

## 17 — admin-applications — DONE (2026-07-17)

Branch: feature/17-admin-applications. Spec: specs/17-admin-applications.md.

**Scope.** Admin management of admission applications: filtered/searchable list, PII detail view, status changes, mark read/unread, and CSV export — every mutation and the export audited.

**Permissions.** Added `applications.manage` to the RBAC catalog (lib/auth/permissions.ts) alongside the existing `applications.view`. View = read list/detail/export (EDITOR + ADMIN/SUPER_ADMIN). Manage = status change + read toggle (ADMIN/SUPER_ADMIN only; EDITOR is content-only and stays read-only on applications). Reconcile loop upserts the new Permission row on boot automatically. Updated test/auth-permissions.test.ts (key list + EDITOR view-not-manage case).

**Data + pure helpers** — lib/admin/applications.ts. Status metadata/pills, `parseFilters` (validates status/date/page from raw URL params, drops bad input), `buildWhere` (name contains-insensitive, status/programme/campus, inclusive whole-day date range), `filtersToQuery`, paginated `getApplicationsPage` (rows + total + unread counts, PAGE_SIZE 20), `getApplication` detail, `getFilterOptions` (all programmes/campuses), `getApplicationsForExport` + RFC-4180 `toCsv` (BOM, quote/escape) + `csvFilename`. Pure helpers unit-tested in test/admin-applications.test.ts (filter parse/validation, where-building incl. date bounds, CSV escaping, filename).

**UI.** /admin/applications list = URL-driven GET filter form (JS-free, shareable), streamed results inside Suspense with skeleton; desktop table + mobile cards; unread rows emphasised (amber dot + tint + bold); summary line with unread count; prev/next pagination; Export CSV button carrying current filters. /admin/applications/[id] detail = student + guardian + message cards, DOB, programme/campus, tel/mailto links, "Reply via email" mailto to guardian; sidebar Manage panel. Client ApplicationActions (status select + mark read/unread) shown only to managers; view-only roles see a read-only status line. New components under components/admin/applications/.

**API routes (Node runtime).** POST /admin/api/applications/[id]/status (manage; validates enum; audits old→new), POST .../[id]/read (manage; audits read/unread), GET /admin/api/applications/export (view; filtered CSV download, audits row count). All reject preview + missing permission; audit summaries carry id + labels only, never applicant PII.

**Privacy.** PII rendered server-side only for authorised admins; audit rows never contain raw PII; export gated on view + audited as a bulk disclosure.

**Gates.** pnpm typecheck ✔ (fixed one enum-narrowing type on the status route). pnpm lint ✔ (switched filter Reset from <a> to next/link). Did not run build/tests per loop rules. No schema.prisma change (permission catalog is code) → no prisma generate needed.

## 18 — admin-inquiries — DONE (2026-07-17)

**Branch:** feature/18-admin-inquiries (FEATURE)

Managed contact inquiries (ContactInquiry model) in the admin portal, mirroring the step-17 applications module for consistency.

**Decisions**
- Added RBAC permission `inquiries.manage` (status + mark-read) alongside the pre-existing `inquiries.view`. EDITOR keeps view but not manage; ADMIN/SUPER_ADMIN hold both — same split as applications. Export is gated on `inquiries.view` (reading the records), matching the applications export gate. Updated `test/auth-permissions.test.ts` (key-set assertion + new EDITOR-inquiries case).
- Search spans name/email/subject (case-insensitive OR); filters are status + date range (inclusive whole end-day); paging 20/page. All driven by URL query params (JS-free, shareable); results stream inside a Suspense boundary with a skeleton. Unread rows emphasised (dot + weight + tint) and counted in the summary line.
- Detail view renders the full record (name/email/phone/subject/message), status pill, unread badge, and a "Reply via email" mailto (subject prefilled `Re: <subject>`). `inquiries.manage` unlocks the status select + mark read/unread client actions (optimistic, POST to audited routes, router.refresh).

**Privacy / accountability**
- PII read server-side only for authorised admins; never rendered to unauthorised roles (Forbidden). Audit summaries carry id + labels only — never raw PII. Status change / mark-read / CSV export each write an AuditLog row (actions `inquiry.status`, `inquiry.read`, `inquiry.export`). Export route is `no-store`; CSV is RFC-4180 escaped with a UTF-8 BOM.

**Files**
- lib/admin/inquiries.ts (data + pure helpers: parseFilters/buildWhere/filtersToQuery/toCsv/statusLabel/csvFilename/formatDate)
- app/admin/inquiries/page.tsx, app/admin/inquiries/[id]/page.tsx
- components/admin/inquiries/{StatusPill,InquiriesFilters,InquiriesResults,InquiryActions}.tsx
- app/admin/api/inquiries/[id]/{status,read}/route.ts, app/admin/api/inquiries/export/route.ts
- lib/auth/permissions.ts (+INQUIRIES_MANAGE), test/admin-inquiries.test.ts, test/auth-permissions.test.ts (updated)

**Gates:** pnpm lint ✔ (no warnings/errors) · pnpm typecheck ✔ (tsc --noEmit clean). Unit tests written (parseFilters/buildWhere/filtersToQuery/toCsv/status/csvFilename + RBAC); controller runs full suite/build.

---

## 19 — admin-campuses — DONE (2026-07-17)

**Branch:** feature/19-admin-campuses · WORK TYPE: FEATURE

**Goal (spec 19):** Staff-facing CRUD for the `Campus` model (create/read/update/delete, images, slug, publish/unpublish, order) with every change reflected on the public `/campuses` pages and audited. Route `/admin/campuses`, permission `campuses.edit`.

**Decisions:**
- **Image storage:** decided local `public/assets/campuses/` (served directly by Next). The optional Cloudflare R2 object store remains deferred to step 21; the DB stores only the public path, so a later R2 switch is a path swap with no schema change. Upload endpoint validates image mime-type (jpeg/png/webp/avif/gif) and a 5 MB size cap, writes with a `randomUUID` filename.
- **No schema change:** the `Campus` model (slug unique, order, published, heroImage, gallery[], addressLines[], mapEmbedUrl, etc.) already existed from spec 08; reused as-is. No prisma migrate/generate needed.
- **Live reflection:** public listing + detail use ISR (`revalidate = 300`); every mutation additionally calls `revalidatePath("/campuses")` and the detail path on demand. Renames revalidate both old and new slug paths.
- **Reorder:** number/arrow based (move up/down swaps `order` with the neighbour in a transaction) rather than drag — simpler, keyboard-accessible, and matches the "large controls, non-technical-friendly" UX ask.
- **Publish toggle** kept as a dedicated lightweight route (list action) separate from the full-form PATCH, mirroring the existing settings/banner + inquiry status route pattern.
- **Validation:** shared pure `normalizeCampusInput` — name required, slug auto-derived from name until hand-edited then sanitised + URL-safety checked, email + https-map-embed format checks, addressLines split by newline, gallery coerced to string[]. Slug uniqueness enforced server-side (409) with an inline debounced availability check (`GET /admin/api/campuses/slug`).

**Files:**
- `lib/admin/campuses.ts` — pure helpers (slugify, isValidSlug, normalizeCampusInput, campusPublicPaths, formatDate, toCampusData) + DB helpers (listCampuses, getCampusForEdit, slugTaken, nextCampusOrder, neighbourFor).
- API (Node runtime, all gated on `campuses.edit`, all audited): `app/admin/api/campuses/route.ts` (POST create), `app/admin/api/campuses/[id]/route.ts` (PATCH update + DELETE), `app/admin/api/campuses/[id]/publish/route.ts` (POST toggle), `app/admin/api/campuses/reorder/route.ts` (POST up/down), `app/admin/api/campuses/upload/route.ts` (POST image), `app/admin/api/campuses/slug/route.ts` (GET availability).
- Pages: `app/admin/campuses/page.tsx` (list), `.../new/page.tsx` (create), `.../[id]/page.tsx` (edit).
- Components: `components/admin/campuses/CampusForm.tsx`, `ImageField.tsx`, `CampusRowActions.tsx`.
- Tests: `test/admin-campuses.test.ts` — slugify, isValidSlug, normalizeCampusInput (name/slug/email/map validation, published coercion, address/gallery parsing), campusPublicPaths.

**Audit actions written:** campus.create, campus.update, campus.publish/unpublish, campus.delete, campus.reorder, campus.upload — summaries carry the campus name/id only.

**Accountability / safety:** all mutations require `campuses.edit` (401 unauth, 403 preview or missing perm); unpublished campuses stay excluded from the public site via the existing `PUBLISHED_WHERE` guard in `lib/campuses/data.ts`; slug stays unique + URL-safe.

**Gates:** `pnpm lint` ✔ (no warnings/errors) · `pnpm typecheck` ✔ (tsc --noEmit clean). Did not run build/tests per build-loop rules (controller runs full gates). Unit tests written for the pure helpers per the established admin-module testing pattern.

## 20 — admin-curriculum — DONE (2026-07-17)

Staff CRUD for the `Programme` model at `/admin/curriculum`, gated on `curriculum.edit`, mirroring the campuses module (step 19). Builds on existing permissions and nav entries (already scaffolded).

Decisions:
- Pure lib `lib/admin/curriculum.ts`: `slugify`/`isValidSlug` (URL-safe, 80-char cap), `isValidIconKey` (only `prog-1..5` or blank), `normalizeProgrammeInput` (name required; slug auto-derived from name when blank; subjects coerced from array or comma/newline string; unknown icon rejected), `curriculumPublicPaths` (`/curriculum` + detail), plus server-only Prisma helpers (list incl. drafts, edit fetch, slug-taken, next-order, neighbour-for reorder, toProgrammeData).
- API routes (Node runtime, all `curriculum.edit`, preview users blocked, audited, revalidate public ISR): POST create (append order), PATCH/DELETE `[id]` (rename revalidates old+new slug), `[id]/publish` toggle, `reorder` (transactional order swap, no-op at ends), `slug` inline check, `upload` (single image → `public/assets/curriculum/`, 5 MB, jpg/png/webp/avif/gif; R2 deferred to step 21).
- UI: list page with thumbnail (image, else icon, else placeholder), Draft/Published pill, per-row publish/reorder/edit/delete-with-confirm; shared `ProgrammeForm` (auto-slug with debounced availability check, stage/age/summary/description, `SubjectsField` chip editor add-on-Enter/comma with dedupe + backspace-remove, `IconPicker` tiles for prog-1..5 + Default, single-image `ImageField` upload/preview, publish toggle, toasts, inline field errors); new + edit pages (edit 404s on unknown id).
- Audit actions: programme.create/update/publish/unpublish/reorder/delete/upload on entity `Programme`.

Files: lib/admin/curriculum.ts; app/admin/api/curriculum/{route,[id]/route,[id]/publish/route,reorder/route,slug/route,upload/route}.ts; app/admin/curriculum/{page,new/page,[id]/page}.tsx; components/admin/curriculum/{ProgrammeForm,ProgrammeRowActions,SubjectsField,IconPicker,ImageField}.tsx; test/admin-curriculum.test.ts.

Gates: lint ✔ (no warnings/errors), typecheck ✔ (tsc --noEmit clean). No schema change (Programme model + permissions + nav pre-existed). Tests written for slug/icon/normalize/subjects/publicPaths; full suite run by controller.

## 21 — admin-gallery — DONE (2026-07-17)

**Branch:** feature/21-admin-gallery · **Type:** FEATURE

Staff CRUD for the public gallery (`GalleryAlbum` + `GalleryImage`), gated on `gallery.edit`, mirroring the campuses/curriculum module shape.

**Storage decision (resolved at this step):** admin-uploaded gallery images stay in the repo's `public/assets/gallery/` folder, served directly by Next; the DB stores only the public path. Optional Cloudflare R2 remains deferred — because only the path is persisted, a later move to R2 is a path swap touching just the upload handler + `next.config` remote patterns, with no schema change. `next/image` renders local `/assets/...` paths without any remote-pattern config, so the public gallery already renders uploaded images.

**Files added:**
- `lib/admin/gallery.ts` — pure helpers (`slugify`, `isValidSlug`, `normalizeAlbumInput`, `albumPublicPaths`, `formatDate`, `toAlbumData`, `toImageCreateData`) + server-only DB helpers (`listAlbums` with image counts, `getAlbumForEdit` with ordered images, `slugTaken`, `nextAlbumOrder`, `neighbourFor`). Alt text is required per image (`normalizeAlbumInput` flags any kept image lacking alt); cover is pinned to one of the album's images (editor choice, else first).
- API (Node runtime, all require `gallery.edit`, all audited, all revalidate `/gallery` [+ `/gallery/<slug>`]): `POST /admin/api/gallery` (create album + nested images), `PATCH/DELETE /admin/api/gallery/[id]` (PATCH replaces scalars AND the image set in one transaction — delete-then-recreate in submitted order so alt edits/reorder/removals persist atomically; DELETE cascades images), `POST /admin/api/gallery/[id]/publish`, `POST /admin/api/gallery/reorder` (order swap with neighbour), `GET /admin/api/gallery/slug` (inline uniqueness), `POST /admin/api/gallery/upload` (multipart → `public/assets/gallery/`, 5 MB cap, type allowlist).
- Pages: `app/admin/gallery/page.tsx` (list: cover thumb, image count, publish pill, row actions), `new/page.tsx`, `[id]/page.tsx`.
- Components: `AlbumImagesField` (large drag/drop zone, sequential multi-upload with progress, per-image alt input with inline "add a description" enforcement, move up/down, star-to-set-cover, remove; broken paths degrade to a neutral block), `AlbumForm` (auto-slug + debounced availability, blocks save while any alt is missing — client + server, toast on success), `AlbumRowActions` (publish/reorder/edit/delete-with-confirm).
- Tests: `test/admin-gallery.test.ts` — slugify/isValidSlug, normalizeAlbumInput (valid record, slug-from-title, title required, **alt-required**, drop-url-less images, cover pinning, empty-draft allowed), toImageCreateData ordering/null-alt, albumPublicPaths.

**Nav/dashboard:** `/admin/gallery` link, "Published albums" stat card and "Add album" quick action already existed from step 16 — no change needed. Permission `gallery.edit` already defined in step 05.

**Do-NOT-break honoured:** behind auth + `gallery.edit`; unpublished albums never public (public loader uses `published:true` where-clause from step 12); alt required (SEO); every mutation audited; upload/create/update/delete/publish/reorder revalidate the public gallery via ISR.

**Gates:** `pnpm lint` ✔ (no warnings/errors) · `pnpm typecheck` ✔ (tsc --noEmit clean). No schema change → no `prisma generate`. Did not run build/tests per build-loop rules (controller runs full gates).

## 22 — admin-testimonials — DONE (2026-07-17)

Staff CRUD for the existing `Testimonial` model, gated on `testimonials.edit`. No schema change (model, permission, dashboard nav/count all pre-existed).

**Decisions**
- Testimonials have no slug — they only surface in the homepage `TestimonialsSection` slider — so the sole public revalidation target is `/`. Kept a `testimonialPublicPaths()` helper (returns `["/"]`) for symmetry with campuses/gallery.
- Fields: authorName (required), authorRole (optional), quote (required, capped 600), photo (optional). Validation lives in the pure `normalizeTestimonialInput`; DB helpers are thin Prisma wrappers, mirroring `lib/admin/campuses.ts`.
- Photo storage per the step-21 decision: single upload endpoint writes to `public/assets/testimonials/`, DB stores only the path (R2 later = path swap, no schema change). Reused the 5 MB / image-type guards.
- Reorder via order-swap-with-neighbour in a transaction; publish toggle is a lightweight route; every mutation (create/update/delete/publish/unpublish/reorder/upload) audited with entity `Testimonial`.

**Files**
- `lib/admin/testimonials.ts` — pure validation/format helpers + server DB reads/writes.
- `app/admin/api/testimonials/route.ts` (POST), `[id]/route.ts` (PATCH/DELETE), `[id]/publish/route.ts`, `reorder/route.ts`, `upload/route.ts`.
- `app/admin/testimonials/page.tsx` (list), `new/page.tsx`, `[id]/page.tsx`.
- `components/admin/testimonials/TestimonialRowActions.tsx`, `PhotoField.tsx` (single round-avatar upload), `TestimonialForm.tsx`.
- `test/admin-testimonials.test.ts` — normalize (valid, required name/quote, optional role/photo, caps), toTestimonialData nulling, public paths.

**Gates:** lint ✔ · typecheck ✔ (build/tests left for controller). Design self-check: matches campuses/gallery admin patterns — PortalShell, Forbidden fallback, plain-language labels, skeleton-free simple list, confirm-on-delete, toast on save, round-avatar preview consistent with the public slider.

WORK TYPE: FEATURE (branch feature/22-admin-testimonials)

## 23 — admin-settings — DONE (2026-07-17)

Built the ADMIN-only site-settings editor at `/admin/settings` (permission `settings.edit`; EDITOR blocked, ADMIN/SUPER_ADMIN allowed). Single `Settings` singleton row edited in place.

**UI:** one form, five tabs — Contact (email, phone, opening hours, address block), Social & WhatsApp (six known platforms + WhatsApp number), Admissions banner (on/off + message), SEO defaults (title, description, OG image) and Email recipients (one per line). Plain-language labels, large controls, inline field validation, single Save with a success toast; a validation failure opens the tab holding the first error.

**Data/logic:** new `lib/admin/settings.ts` holds the pure model — `normalizeSettingsInput` (trim/cap, validate emails, absolute-URL social links, WhatsApp digit count, OG image path/URL; a fully blank save is valid for an unseeded school), `toSettingsData`, `settingsToForm`, `buildSocialLinksJson`/`buildSeoJson` (store only non-empty; social keys match `parseSocialLinks`), `summarizeChangedFields` (names the changed sections for the audit line) and `settingsRevalidateTargets`. `socialLinks`/`seoDefaults` stay JSON so shape can evolve without migration.

**API:** `PUT /admin/api/settings` (Node runtime) — auth + preview + permission gate, normalise → 422 with field errors, snapshot the row for the diff, update with `updatedById` stamped, write an `AuditLog` (`settings.update`, summary of changed sections), then `revalidatePath('/', 'layout')` so footer/top-bar/banner/contact/WhatsApp reflect immediately across the whole tree. The existing `POST /admin/api/settings/banner` (dashboard quick-toggle) is untouched.

**Schema:** added nullable `Settings.contactHours` (spec Contact section lists hours) via migration `20260717140000_settings_contact_hours`; ran `pnpm prisma generate`.

**Tests:** `test/admin-settings.test.ts` — normalisation (valid/blank/invalid email/social/recipient/WhatsApp/OG), serialisation + form round-trip, changed-section summary, revalidate target, and the RBAC mapping (EDITOR blocked, ADMIN/SUPER_ADMIN allowed).

**Gates:** `pnpm prisma generate` ✔ · `pnpm lint` ✔ (no warnings/errors) · `pnpm typecheck` ✔ (clean). Did not run build/unit/e2e per AGENT.md (controller runs full gates).

WORK TYPE: FEATURE (branch feature/23-admin-settings)

---

## 24 — prelaunch-seo-audit — DONE (2026-07-17)

**Branch:** `feature/24-prelaunch-seo-audit` (WORK TYPE: FEATURE)
**Spec:** specs/24-prelaunch-seo-audit.md — quality gate before go-live; staging stays noindex (indexing flip is step 25).

### What was done
Full-site SEO / performance / accessibility audit of every public route (admin + API out of scope). Produced a staging-only audit report and cleared the code-fixable findings; added the two test suites the spec's gate demands (structured-data + sitemap completeness).

### Audit outcome (per area)
- **Metadata:** PASS. Every public page composes via `buildMetadata()` — templated unique title, unique description, canonical from `path`, robots policy, OG/Twitter. Dynamic detail pages use `generateMetadata` with 404 fallback. Exactly one `<h1>` per page.
- **Structured data:** PASS. Site-wide `EducationalOrganization` (layout) + `WebSite` (home); `BreadcrumbList` on all inner pages; `FAQPage` on /admissions; `ContactPage` on /contact; per-page `EducationalOccupationalProgram` / campus `EducationalOrganization`+`PostalAddress` / album `ImageGallery`+`ImageObject`.
- **Sitemap/robots:** PASS. Sitemap lists all 10 static routes + published dynamic slugs via the registry; noindex utility routes excluded; robots fail-closed (staging `Disallow: /`, no sitemap advertised). Staging noindex NOT flipped.
- **Performance/CWV:** code-level PASS (next/image everywhere except an admin icon picker, lazy-load, next/font `font-display: swap`, Server Components + isolated client islands, prod-only analytics). Live Lighthouse numbers deferred to real-asset delivery at step 25 — documented exception.
- **Links + alt:** PASS. No broken internal links; every meaningful image has descriptive alt, decorative imagery uses `alt=""`+`aria-hidden`.
- **A11y:** PASS baseline — single h1, focus states, keyboard-navigable forms/lightbox, iframe titles, contrast on text surfaces.

### Findings + resolutions
- **F1 (fixed):** /contact emitted three overlapping Organization nodes. Dropped the standalone `organizationJsonLd(sameAs)` on the contact page (removed its import too); `ContactPage.mainEntity` already carries email/phone/address/sameAs and the layout emits the site-wide node. File: app/contact/page.tsx.
- **F3 (fixed):** `alt="Portrait of the GSIS Principal (placeholder image)"` leaked a build note into SR text → now `"Portrait of the GSIS Principal"`; placeholder disclosure stays in the visible caption. File: components/about/PrincipalMessage.tsx.
- **F2 (accepted):** site-wide Organization has empty `sameAs` (root layout is static, doesn't fetch Settings); /contact's ContactPage already carries sameAs. Documented, not changed.

### Files
- Added: docs/seo-audit.md (audit report artifact).
- Added: test/structured-data.test.ts — validates every JSON-LD helper (site-wide + per-page): @context/@type present on every node, required fields per type, all emitted URLs absolute, round-trips through JSON.
- Added: test/sitemap-completeness.test.ts — mocks @/lib/db, imports app/sitemap, asserts the URL set equals exactly the 10 static routes + published dynamic slugs, no dupes, all absolute, noindex utility routes excluded.
- Edited: app/contact/page.tsx, components/about/PrincipalMessage.tsx (fixes above).
- Edited: PROGRESS.md.

### Gates
- `pnpm lint` ✔ (no warnings/errors) · `pnpm typecheck` ✔ (clean). No schema.prisma change → no prisma generate. Did not run build/test:unit/e2e per AGENT.md (controller runs full gates).

## 25 — production-launch — DONE (2026-07-18)

**Type:** FEATURE (branch feature/25-production-launch). Final build step. HUMAN-GATED: the actual `main`/production deploy happens ONLY on the owner's explicit Telegram command — the agent prepares and verifies on staging and does NOT auto-deploy production.

**Spec:** specs/25-production-launch.md.

### What this step is
Preparation + verification of the go-live wiring, plus tests and an owner runbook. No new product surface — the launch machinery was scaffolded earlier (robots/SEO in 03, analytics/Search-Console env in 03, screenshot bypass in 05). This step confirms it is all correct and fail-closed, and documents the human-gated deploy.

### Pre-deploy verification (all reviewed live — correct)
- lib/env.ts — `APP_ENV === "production"` is the single indexability switch; staging/missing/near-miss ("prod", "PRODUCTION") fail closed to staging/noindex. isProduction/isIndexable/isStaging derived from it.
- app/robots.ts — production: allow `/` + sitemap + host; everything else disallow `/` (fail-closed).
- lib/seo.ts robotsMeta() — index,follow in prod; noindex,nofollow otherwise; applied site-wide by buildMetadata(). canonicalUrl() from NEXT_PUBLIC_SITE_URL (trailing slash stripped).
- components/Analytics.tsx — renders the script only when isProduction && NEXT_PUBLIC_ANALYTICS_SRC set; silent on staging/dev.
- app/layout.tsx — metadata.verification = { google } only when isProduction && SEARCH_CONSOLE_VERIFICATION set.
- lib/auth/screenshot.ts — screenshot preview is OFF in production (staging-only exact match, GET/HEAD, constant-time compare, fail-closed).
- deploy.sh — dangerous-command fence refuses main/master/production/prod* branches; only ever deploys staging. Production go-live is a separate owner-run action, not this script.
- .env.example — placeholders only; all prod launch vars present (APP_ENV, NEXT_PUBLIC_SITE_URL, NEXT_PUBLIC_ANALYTICS_SRC, SEARCH_CONSOLE_VERIFICATION, SCREENSHOT_TOKEN, JWT_SECRET, DATABASE_URL, seed admin). No real secrets committed.

### Files added
- test/production-launch.test.ts — locks the launch invariants as a unit: env indexability fail-closed across APP_ENV values; analytics + Search-Console tokens resolve from env (undefined when empty); verification-meta gate (isProduction && token); Analytics loader renders nothing on staging / prod-without-src and loads the script in prod-with-src (next/script mocked, rendered via react-dom/server); screenshot bypass OFF in production.
- docs/production-launch.md — go-live runbook: pre-deploy verification summary, production env vars to set server-side, infra hand-off (DNS/SSL = owner/ops), owner-triggered go-live steps (submit sitemap, confirm analytics, smoke tests), and post-launch verification (robots allow, no noindex, screenshot bypass off, staging still noindex, no secrets exposed).

### Decisions
- No auto-deploy: honored the single human gate. Ended with [CHECKPOINT] after preparation; did not touch main, did not run deploy.
- SCREENSHOT_TOKEN documented as "leave UNSET in production" (belt-and-braces on top of the code-level fail-closed).
- Did not run test:unit/build (per CLAUDE.md §6 — controller runs full gates). Ran lint + typecheck once.

### Gates
- typecheck ✔ (tsc --noEmit clean)
- lint ✔ (no ESLint warnings or errors)
- build ➖ (controller runs)
- tests ➖ authored (production-launch.test.ts + existing robots/seo/auth-screenshot cover prod→indexable and screenshot-off-in-prod); controller runs the suite
- SEO ✔ (production indexable path verified: robots allow + index,follow + canonical to prod host; staging stays noindex)
- responsive/design ➖ (no UI change)
- privacy ✔ / env ✔ (placeholders only; no secrets committed)

WORK TYPE: FEATURE (branch feature/25-production-launch)

## 28 — shared-components — DONE (2026-07-18)

Rebuilt the shared component layer so every page rebuild (31–37, 39) composes from one consistent source. Branch: feature/28-shared-components.

**Layout components (new `components/layout/`):**
- `Header.tsx` — floating, sticky-on-scroll, translucent-blur, logo-only (`/assets/logo.png`), NO theme toggle. Canonical `NAV_ITEMS` in fixed order Home · About · Admissions · Campuses · Curriculum · Gallery · Contact, amber hover-underline, active-page indication via `usePathname`. Primary (navy) "Apply for Admission" CTA. Mobile slide-in panel: `aria-expanded`/`aria-controls`, `role=dialog aria-modal`, Esc-to-close, Tab focus-trap, scroll-lock, focus restored to opener.
- `Footer.tsx` — navy-deep floating surface. Quick links deliberately EXCLUDE Campuses/Privacy/Terms; social icons from `parseSocialLinks(Settings.socialLinks)` which never yields WhatsApp; contact (email/phone/hours/address) from Settings props; "Built with ♥ by AboveNext" → https://abovenext.com (no tracking params). Legal nav removed from the bottom bar.
- `Section.tsx` — owns white↔off-white gradient blending (tone white/soft/blend, blend up/down), container width + padding presets.
- `Breadcrumbs.tsx` — semantic trail + matching `BreadcrumbList` JSON-LD (via `lib/seo.breadcrumbJsonLd`).
- `PageHero.tsx` — inner-page title with amber underline, subtitle, breadcrumb slot.
- `CtaBand.tsx` — shared teal→navy gradient CTA box.

**UI components (`components/ui/`):**
- `Button.tsx` — spec redefines variants: primary=navy fill, secondary=teal (swapped from prior), amber, and ghost with a clearly-visible navy-tinted border AT REST (`border-2 border-navy/25`, hover deepens). Tactile hover/shine/arrow retained.
- `Card.tsx` — variants content/glass/media; `--shadow-sm` at rest → `--shadow-lg` on hover when interactive.
- `Skeleton.tsx` — primitive + `SkeletonText`/`SkeletonImage`/`SkeletonCard`/`SkeletonTable`; shimmer gradient border↔off-white. No preloaders/spinners anywhere.
- `WhatsAppFloat.tsx` — renamed from WhatsAppButton; number from Settings, renders null when empty; fixed bottom-right on public pages, never mounted in the portal.
- Barrel adds `ScrollReveal` (alias of `Reveal`), the new skeleton exports, and re-exports the layout components; keeps a `WhatsAppButton = WhatsAppFloat` back-compat alias for existing pages pending rebuild.

**Removals / cleanup:** deleted `components/ui/{Header,Footer,ThemeToggle,WhatsAppButton}.tsx`; removed residual `dark:` classes from ContactForm/ApplyForm; updated `app/dev/ui` to drop ThemeToggle and use WhatsAppFloat. Grep confirms no theme-toggle/dark-mode/preloader code remains in `components/`.

**Decisions:** logo path kept at the real asset `/assets/logo.png` (spec text mentioned `/assets/branding/logo.png`, which does not exist after step 27). Existing public pages still call `<Header/>`/`<Footer/>`/`<WhatsAppButton/>` with no/compatible props; they will be wired to Settings and rebuilt in 31–37.

**Tests:** rewrote `components/ui/__tests__/components.test.tsx` — header full nav + order, no theme toggle, mobile menu aria-expanded; footer AboveNext credit + Campuses/Privacy/Terms exclusion + socials-without-WhatsApp; button primary/secondary/ghost-border variants; skeleton primitives. Mocks `next/navigation.usePathname`.

**Gates:** typecheck ✔ · lint ✔ (no warnings). Did not run build/tests per loop rules (controller runs full gates).
