> ⚠️ **PUBLIC FILE** — pushed to the public progress repo (`projects-abovenext/gsis-website-progress`). **NO secrets, hostnames, real subdomains, endpoints, or keys — ever.** Placeholders only.

# PROGRESS.md — GSIS Website & Admin Portal

> **Lean, always-loaded state file.** The agent reads this every session to know the current state and the next step. Keep it SHORT — full detail lives in `PROGRESS-HISTORY.md`. Update this as part of finishing each step (AGENT.md §6), before `[CHECKPOINT]`.

---

## Current Status

- **Project:** GSIS Website & Admin Portal — built & delivered by AboveNext.
- **Last completed build-step:** `15-supporting-pages` (2026-07-17)
- **Current build-step:** `16` (see build order)
- **Next-step pointer:** → next spec in `specs/`
- **Branch state:** last merged `feat/11-admission-form` → `staging`. Next branch: `feature/16-*`.
- **Environment:** working on `staging` only (noindex/robots-disallowed). `main`/production is human-gated (owner Telegram command).
- **Checkpoint model:** continuous — every step ends with a soft `[CHECKPOINT]`; the single human gate is production launch (step 25).
- **Build order:** runs `01` → `25`. **NOT complete until `25-production-launch` is DONE.**

## Recent steps (keep exactly 3 one-liners; newest on top)

- **15 — supporting-pages** — DONE (2026-07-17) — Shared branded thank-you template, on-design noindex 404 and error boundary, plus a full-site SEO and broken-link sweep.
- **14 — legal** — DONE (2026-07-17) — Privacy and Terms as two distinct indexable URLs with a path-driven tabbed shell, in-page anchors, back-to-top, accurate PII disclosure, footer links, sitemap and full SEO gate.
- **13 — contact** — DONE (2026-07-17) — Contact page with details, map and socials from Settings; shared-schema form with honeypot plus rate-limit saves an inquiry, emails staff and sender, inline thank-you, full SEO gate.

## Notes / open decisions

- Email: console sink for now (real Resend/SMTP deferred to production); staging never emails real recipients. Public forms use honeypot + per-IP rate-limit (no captcha).
- `[DECIDE AT BUILD]` analytics provider + Search Console — scaffolded in `03-seo-core`, activated at `25-production-launch`.
- `[DECIDE AT BUILD]` optional object storage (Cloudflare R2) for admin-uploaded gallery/testimonial images — decided at `21-admin-gallery`.
- `[DECIDE AT BUILD]` optional 2FA for ADMIN accounts (`05-admin-auth`).
- Client finalises all real content; sample/placeholder data used until then. Never invent real facts.
- Missing real photos to be supplied later: `hero.jpg`, `about-1.jpg`, `about-2.jpg`, `avatar-1..4.jpg` (use clearly-marked placeholders with correct filenames).

---

_Detail per completed step lives in PROGRESS-HISTORY.md. This file stays short._