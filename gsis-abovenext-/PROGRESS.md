> ⚠️ **PUBLIC FILE** — pushed to the public progress repo (`projects-abovenext/gsis-website-progress`). **NO secrets, hostnames, real subdomains, endpoints, or keys — ever.** Placeholders only.

# PROGRESS.md — GSIS Website & Admin Portal

> **Lean, always-loaded state file.** The agent reads this every session to know the current state and the next step. Keep it SHORT — full detail lives in `PROGRESS-HISTORY.md`. Update this as part of finishing each step (AGENT.md §6), before `[CHECKPOINT]`.

---

## Current Status

- **Project:** GSIS Website & Admin Portal — built & delivered by AboveNext.
- **Last completed build-step:** _none yet (build not started)._
- **Current build-step:** `01-foundation-nextjs`
- **Next-step pointer:** → `specs/01-foundation-nextjs.md`
- **Branch state:** none yet. First branch will be `feat/01-foundation-nextjs` → `staging`.
- **Environment:** working on `staging` only (noindex/robots-disallowed). `main`/production is human-gated (owner Telegram command).
- **Checkpoint model:** continuous — every step ends with a soft `[CHECKPOINT]`; the single human gate is production launch (step 25).

## Recent steps (keep exactly 3 one-liners; newest on top)

- _(none yet — build has not started)_
- _(none yet)_
- _(none yet)_

## Notes / open decisions

- `[DECIDE AT BUILD]` email provider (Resend vs SMTP) — needed by step `11-admission-form`.
- `[DECIDE AT BUILD]` analytics provider + Search Console — needed by step `03-seo-core` (scaffold) and `25-production-launch` (go-live).
- `[DECIDE AT BUILD]` optional object storage (Cloudflare R2) for admin-uploaded gallery/testimonial images — else `public/assets/` at launch (step `21-admin-gallery`).
- `[DECIDE AT BUILD]` optional 2FA for ADMIN accounts (step `05-admin-auth`).
- `[DECIDE AT BUILD]` spam protection for public forms — honeypot + rate-limit vs captcha (steps `11`, `13`).
- Client will finalise all real content (copy, real photos, campus/programme details, stats, testimonials, contact info). Sample/placeholder data is used until then. Never invent real facts.
- Assets present: `logo.png` + all 14 3D icons (`val-1..5`, `why-1..4`, `prog-1..5`) + `why.jpg`. **Missing real photos** to be supplied later: `hero.jpg`, `about-1.jpg`, `about-2.jpg`, `avatar-1..4.jpg` (use clearly-marked placeholders with correct filenames).
- Two campuses at launch (GSIS Primary, GSIS International); five programmes (Starters, Movers, Flyers, Anchors, High School Programme) — rest managed via the portal.
- "Admissions Open" banner text (e.g. "Admissions Open 2026–27") is admin-toggleable with a settable year range (step `23-admin-settings`).

---

_Detail per completed step lives in PROGRESS-HISTORY.md. This file stays short._
