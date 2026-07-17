> ⚠️ **PUBLIC FILE** — pushed to the public progress repo (`projects-abovenext/gsis-website-progress`). **NO secrets, hostnames, real subdomains, endpoints, or keys — ever.** Placeholders only.

# PROGRESS.md — GSIS Website & Admin Portal

> **Lean, always-loaded state file.** The agent reads this every session to know the current state and the next step. Keep it SHORT — full detail lives in `PROGRESS-HISTORY.md`. Update this as part of finishing each step (AGENT.md §6), before `[CHECKPOINT]`.

---

## Current Status

- **Project:** GSIS Website & Admin Portal — built & delivered by AboveNext.
- **Last completed build-step:** `19-admin-campuses` (2026-07-17)
- **Current build-step:** `20` (see build order)
- **Next-step pointer:** → next spec in `specs/`
- **Branch state:** last merged `feat/11-admission-form` → `staging`. Next branch: `feature/20-*`.
- **Environment:** working on `staging` only (noindex/robots-disallowed). `main`/production is human-gated (owner Telegram command).
- **Checkpoint model:** continuous — every step ends with a soft `[CHECKPOINT]`; the single human gate is production launch (step 25).
- **Build order:** runs `01` → `25`. **NOT complete until `25-production-launch` is DONE.**

## Recent steps (keep exactly 3 one-liners; newest on top)

- **19 — admin-campuses** — DONE (2026-07-17) — Staff CRUD for campuses — auto-slug + uniqueness, image upload/preview, reorder, publish toggle; audited and revalidating public pages.
- **18 — admin-inquiries** — DONE (2026-07-17) — Contact inquiries list with URL-driven search/filter/paging, PII detail with audited status and read toggles, reply-by-email link and an access-controlled CSV export.
- **17 — admin-applications** — DONE (2026-07-17) — Applications list with URL-driven search/filter/paging, PII detail with audited status and read toggles, and an access-controlled CSV export.

## Notes / open decisions

- Email: console sink for now (real Resend/SMTP deferred to production); staging never emails real recipients. Public forms use honeypot + per-IP rate-limit (no captcha).
- `[DECIDE AT BUILD]` analytics provider + Search Console — scaffolded in `03-seo-core`, activated at `25-production-launch`.
- `[DECIDE AT BUILD]` optional object storage (Cloudflare R2) for admin-uploaded gallery/testimonial images — decided at `21-admin-gallery`.
- `[DECIDE AT BUILD]` optional 2FA for ADMIN accounts (`05-admin-auth`).
- Client finalises all real content; sample/placeholder data used until then. Never invent real facts.
- Missing real photos to be supplied later: `hero.jpg`, `about-1.jpg`, `about-2.jpg`, `avatar-1..4.jpg` (use clearly-marked placeholders with correct filenames).

---

_Detail per completed step lives in PROGRESS-HISTORY.md. This file stays short._