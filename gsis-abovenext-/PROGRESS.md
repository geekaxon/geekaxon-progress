> ⚠️ **PUBLIC FILE** — pushed to the public progress repo (`projects-abovenext/gsis-website-progress`). **NO secrets, hostnames, real subdomains, endpoints, or keys — ever.** Placeholders only.

# PROGRESS.md — GSIS Website & Admin Portal

> **Lean, always-loaded state file.** The agent reads this every session to know the current state and the next step. Keep it SHORT — full detail lives in `PROGRESS-HISTORY.md`. Update this as part of finishing each step (AGENT.md §6), before `[CHECKPOINT]`.

---

## Current Status

- **Project:** GSIS Website & Admin Portal — built & delivered by AboveNext.
- **Last completed build-step:** `10-admissions` (2026-07-17)
- **Current build-step:** `11-admission-form`
- **Next-step pointer:** → `specs/11-admission-form.md`
- **Branch state:** last merged `feat/10-admissions` → `staging`. Next branch: `feat/11-admission-form`.
- **Environment:** working on `staging` only (noindex/robots-disallowed). `main`/production is human-gated (owner Telegram command).
- **Checkpoint model:** continuous — every step ends with a soft `[CHECKPOINT]`; the single human gate is production launch (step 25).
- **Build order:** runs `01` → `25`. **NOT complete until `25-production-launch` is DONE.**

## Recent steps (keep exactly 3 one-liners; newest on top)

- **10 — admissions** — DONE (2026-07-17) — Admissions info page: process timeline, requirements, marketing fees note, FAQ accordion and CTA, with the full SEO gate.
- **09 — curriculum** — DONE (2026-07-17) — DB-driven curriculum listing plus slug detail pages with clay icons, published-only, 404s and the SEO gate.
- **08 — campuses** — DONE (2026-07-17) — Campuses listing plus slug-based detail pages driven from the DB, published-only with 404s, sitemap entries and SEO gate.

## Notes / open decisions

- `[DECIDE AT BUILD]` email provider (Resend vs SMTP) — needed by step `11-admission-form`.
- `[DECIDE AT BUILD]` analytics provider + Search Console — scaffolded in `03-seo-core`, activated at `25-production-launch`.
- `[DECIDE AT BUILD]` optional object storage (Cloudflare R2) for admin-uploaded gallery/testimonial images — decided at `21-admin-gallery`.
- `[DECIDE AT BUILD]` optional 2FA for ADMIN accounts (`05-admin-auth`).
- `[DECIDE AT BUILD]` spam protection for public forms — honeypot + rate-limit vs captcha (`11`, `13`).
- Client finalises all real content; sample/placeholder data used until then. Never invent real facts.
- Missing real photos to be supplied later: `hero.jpg`, `about-1.jpg`, `about-2.jpg`, `avatar-1..4.jpg` (use clearly-marked placeholders with correct filenames).

---

_Detail per completed step lives in PROGRESS-HISTORY.md. This file stays short._