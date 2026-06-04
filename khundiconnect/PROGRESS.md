> ⚠️ **PUBLIC FILE** — This document is pushed to a **separate public progress repo** (per-project folder). It must contain **NO secrets, NO hostnames, NO real subdomains, NO API endpoints, NO credentials, NO keys — ever.** Placeholders only.

# PROGRESS.md — KhundiConnect Build Tracker

> The agent updates this file after every build-step. It is the **memory across sessions** and the **recovery source after any pause** (STOP / 3-error / usage-limit). Read it at the start of every session; on resume, re-read it to recover state before doing anything. **Never mark a build-step `DONE` while any verification gate is red.**
>
> **Numbering:** build-step numbers are canonical (ARCHITECTURE.md §27). "Module N" always means build-step N.

## Current Status
- **Project:** KhundiConnect (multi-tenant community SaaS)
- **Phase:** Not started
- **Current build-step:** — (begin at build-step 1, §27)
- **Last updated:** —
- **Last session summary:** —
- **Resume note:** This file is the source of truth for current state after any pause. If continuity is uncertain, trust this file + the last commit, not assumed context.

## Build-Order Checklist (canonical, from ARCHITECTURE.md §27)
Status key: ⬜ not started · 🟨 in progress · ✅ done (all gates green) · 🛑 awaiting human checkpoint

1. ⬜ Foundation (monorepo, NestJS+Next.js+Postgres+Prisma+Redis, Docker, env config incl. OMJ endpoint placeholder) — `feat/01-foundation`
2. ⬜ Multi-tenancy core (tenants, subdomains, **RLS**, tenant context) — `feat/02-multitenancy` — 🛑 HUMAN CHECKPOINT
3. ⬜ Auth & Identity (OMJ-card login, OTP/password, JWT, sessions) — `feat/03-auth` — 🛑 HUMAN CHECKPOINT
4. ⬜ Dynamic Permission Engine + dual-gate guards — `feat/04-permissions` — 🛑 HUMAN CHECKPOINT
5. ⬜ Module-enablement (module_flags) + Vendor tenant provisioning + Khundi Super Admin — `feat/05-module-flags`
6. ⬜ Member management + family graph + households + derived eligibility — `feat/06-members`
7. ⬜ OMJ data sync (manual + cron + conflict queue) — `feat/07-omj-sync` — 🛑 HUMAN CHECKPOINT
8. ⬜ Census (snapshots, entry, approval, family tree, demographics, progress) — `feat/08-census`
9. ⬜ Lawazim / fee engine — `feat/09-lawazim`
10. ⬜ Dual-mode accounting — `feat/10-accounting`
11. ⬜ Notifications (adapter + Twilio/Infobip) + communication log — `feat/11-notifications`
12. ⬜ AI Insights (adapter + cache + scope/visibility + permissions) — `feat/12-ai-insights`
13. ⬜ Reports & animated dashboards (3 altitudes) + exports — `feat/13-reports`
14. ⬜ PWA member self-service — `feat/14-pwa`
15. ⬜ White-label (paid) + billing + module toggles UI — `feat/15-whitelabel`
16. ⬜ OMJ super-user oversight UI + Vendor analytics — `feat/16-oversight`
17. ⬜ Audit, backups, i18n/Urdu polish, accessibility — `feat/17-audit-i18n`
18. ⬜ Phase-2 modules (welfare, bereavement/graves, prize, events, MoM, docs, cards, QR) — `feat/18-phase2`

> **Fixed checkpoints:** steps 2, 3, 4, 7 (🛑 above). **General checkpoint rule also applies on ANY step:** the agent prints `[CHECKPOINT]` whenever it changes the data model, touches RLS/`tenant_id`, changes auth or permissions, or does anything production-affecting. See AGENT.md §6.

## Verification Gate Log
> For each completed build-step, record which gates ran and their result (✅/❌/➖).

| Step | Tenant Isolation | Permission | Module-Flag | No-Auto-Action | Secrets | AI Privacy | Voting Elig. | Notes |
|------|------------------|------------|-------------|----------------|---------|------------|--------------|-------|
| — | — | — | — | — | — | — | — | — |

## Decisions & Assumptions Log (for human review)
> Record every [DECIDE AT BUILD] choice and any business-rule assumption made.

| Date | Item | Decision/Assumption made | Needs human confirm? |
|------|------|--------------------------|----------------------|
| — | ORM | Prisma (per §4 recommendation) | no |
| — | Subdomain base | `{tenant}.khundiconnect.com` — **CONFIRMED/LOCKED** | no |
| — | Login credential | [DECIDE AT BUILD] — defaulting to OTP-to-phone | yes |
| — | Pricing (PKR) | [DECIDE AT BUILD] — placeholder tiers | yes |

## Open Questions for Human (Saad)
- (none yet)

## Checkpoint Log
> Record each `[CHECKPOINT]` raised and its APPROVE/REJECT outcome.

| Date | Build-step | Reason | Outcome |
|------|-----------|--------|---------|
| — | — | — | — |

## Environment / Deploy State
- Staging: not provisioned
- Live environment: not provisioned (owner-Telegram-gated promotion only)
- Code repo (private): not created
- Public progress repo: not created
- CI/CD: not set up
- Cloudflare DNS: not configured
