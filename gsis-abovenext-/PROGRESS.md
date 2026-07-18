> ⚠️ **PUBLIC FILE** — pushed to the public progress repo (`projects-abovenext/gsis-website-progress`). **NO secrets, hostnames, real subdomains, endpoints, or keys — ever.** Placeholders only.

# PROGRESS.md — GSIS Website & Admin Portal

> **Lean, always-loaded state file.** The agent reads this every session to know the current state and the next step. Keep it SHORT — full detail lives in `PROGRESS-HISTORY.md`. Update this as part of finishing each step (AGENT.md §6), before `[CHECKPOINT]`.

---

## Current Status

- **Project:** GSIS Website & Admin Portal — built & delivered by AboveNext.
- **Last completed build-step:** `25-production-launch` (2026-07-18)
- **Current build-step:** `25` — DONE (v1 order 01→25 complete). Production go-live is owner-triggered (Telegram).
- **Next-step pointer:** → `26-design-system-v2` (a v2 rebuild phase, specs `26`→`41`, now present in `specs/`).
- **Branch state:** last merged `feat/11-admission-form` → `staging`. Next branch: `feature/26-*`.
- **Environment:** working on `staging` only (noindex/robots-disallowed). `main`/production is human-gated (owner Telegram command).
- **Checkpoint model:** continuous — every step ends with a soft `[CHECKPOINT]`; the single human gate is production launch.
- **Build order:** v1 ran `01` → `25` (done). A v2 phase `26` → `41` (ending in another production-launch) is now specced.

## Recent steps (keep exactly 3 one-liners; newest on top)

- **25 — production-launch** — DONE (2026-07-18) — Verified prod-only indexable/analytics/verification wiring and off-in-prod screenshot bypass; added go-live runbook and launch tests.
- **24 — prelaunch-seo-audit** — DONE (2026-07-17) — Full-site SEO/perf/a11y audit on staging; report written, minor JSON-LD and alt-text fixes made; structured-data + sitemap tests added; staging stays noindex.
- **23 — admin-settings** — DONE (2026-07-17) — Admin-only tabbed editor for contact, socials, WhatsApp, admissions banner, SEO and email recipients; audited, revalidates the whole site.

## Notes / open decisions

- Email: console sink for now (real Resend/SMTP deferred to production); staging never emails real recipients. Public forms use honeypot + per-IP rate-limit (no captcha).
- `[DECIDE AT BUILD]` analytics provider + Search Console — scaffolded in `03-seo-core`, activated at `25-production-launch`.
- Image storage: admin-uploaded gallery/testimonial images live in `public/assets/` (DB stores only the path). Object storage (R2) deferred — a later path swap, no schema change.
- `[DECIDE AT BUILD]` optional 2FA for ADMIN accounts (`05-admin-auth`).
- Client finalises all real content; sample/placeholder data used until then. Never invent real facts.
- Missing real photos to be supplied later: `hero.jpg`, `about-1.jpg`, `about-2.jpg`, `avatar-1..4.jpg` (use clearly-marked placeholders with correct filenames).

---

_Detail per completed step lives in PROGRESS-HISTORY.md. This file stays short._