> ⚠️ **PUBLIC FILE** — pushed to the public progress repo (`projects-abovenext/gsis-website-progress`). **NO secrets, hostnames, real subdomains, endpoints, or keys — ever.** Placeholders only.

# PROGRESS.md — GSIS Website & Admin Portal

> **Lean, always-loaded state file.** The agent reads this every session to know the current state and the next step. Keep it SHORT — full detail lives in `PROGRESS-HISTORY.md`. Update this as part of finishing each step (AGENT.md §6), before `[CHECKPOINT]`.

---

## Current Status

- **Project:** GSIS Website & Admin Portal — built & delivered by AboveNext.
- **Last completed build-step:** `24-prelaunch-seo-audit` (2026-07-17)
- **Current build-step:** `25` (see build order)
- **Next-step pointer:** → next spec in `specs/`
- **Branch state:** last merged `feat/11-admission-form` → `staging`. Next branch: `feature/25-*`.
- **Environment:** working on `staging` only (noindex/robots-disallowed). `main`/production is human-gated (owner Telegram command).
- **Checkpoint model:** continuous — every step ends with a soft `[CHECKPOINT]`; the single human gate is production launch (step 25).
- **Build order:** runs `01` → `25`. **NOT complete until `25-production-launch` is DONE.**

## Recent steps (keep exactly 3 one-liners; newest on top)

- **24 — prelaunch-seo-audit** — DONE (2026-07-17) — Full-site SEO/perf/a11y audit on staging; report written, minor JSON-LD and alt-text fixes made; structured-data + sitemap tests added; staging stays noindex.
- **23 — admin-settings** — DONE (2026-07-17) — Admin-only tabbed editor for contact, socials, WhatsApp, admissions banner, SEO and email recipients; audited, revalidates the whole site.
- **22 — admin-testimonials** — DONE (2026-07-17) — Staff CRUD for quotes — author, role, photo, reorder, publish; audited, revalidates the homepage slider.

## Notes / open decisions

- Email: console sink for now (real Resend/SMTP deferred to production); staging never emails real recipients. Public forms use honeypot + per-IP rate-limit (no captcha).
- `[DECIDE AT BUILD]` analytics provider + Search Console — scaffolded in `03-seo-core`, activated at `25-production-launch`.
- Image storage: admin-uploaded gallery/testimonial images live in `public/assets/` (DB stores only the path). Object storage (R2) deferred — a later path swap, no schema change.
- `[DECIDE AT BUILD]` optional 2FA for ADMIN accounts (`05-admin-auth`).
- Client finalises all real content; sample/placeholder data used until then. Never invent real facts.
- Missing real photos to be supplied later: `hero.jpg`, `about-1.jpg`, `about-2.jpg`, `avatar-1..4.jpg` (use clearly-marked placeholders with correct filenames).

---

_Detail per completed step lives in PROGRESS-HISTORY.md. This file stays short._