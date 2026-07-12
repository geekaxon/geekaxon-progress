# PROGRESS.md — Rihaish

**Current build-step:** 24 — announcements — DONE (2026-07-12)
**Last completed:** 24 — announcements — DONE (2026-07-12)
**Next:** 25 — (see ARCHITECTURE.md build order)

### Recent steps

- **24 — announcements** — DONE (2026-07-12) — `announcements.core` feature (deps residents.registry, core.notifications, core.storage), perms `society.announcements.read/.publish/.moderate`, nav `announcements` (feature-gated, resident-facing). Migration `20260712270000_announcements`: `Announcement` (scoped: societyId+deletedAt; snapshots `recipientFlatIds/UserIds` at publish for the receipt denominator) + `BereavementDetail`/`AnnouncementRead`/`AnnouncementComment`(soft-delete via `removedAt`, kept as "removed")/`AnnouncementReaction` (pass-through child rows) + enums `AnnouncementType`/`AnnouncementTargetType`/`Priority`/`AnnouncementStatus`. **Type drives behaviour, enforced server-side** (`lib/announcements/rules.ts`): BEREAVEMENT can never carry a reaction (no code path) & never auto-expires; publish rejects a death without `familyOptedIn` and any zero-recipient target; URGENT maps to the `emergency` notification category (floors push, SMS still muteable). Targeting resolves ALL/BLOCK/FLAT_LIST/ROLE/CATEGORY → flats+users; feed is LIVE-targeted (late movers see it, aren't in the receipt). Notifications: fires `announcement.posted` via `notifySafe` with new `restrictChannels` (per-announcement channel picks ∩ society config ∩ prefs); template extended with SMS+WhatsApp (en+ur). Worker: society-scoped `announcements.tick` cron publishes due scheduled + expires lapsed (never bereavements), in society time. API `/api/announcements` (GET feed / `?scope=manage` table, POST), `/:id` (GET+PATCH), `/:id/{publish,read,read-report(+CSV),comments,reactions}`, `/comments/:id` DELETE, `/recipients` preview. UI Simple (resident feed + detail drawer w/ comments/reactions + committee composer w/ live recipient count + channel warning) & Pro (status/type/priority/target table + read-receipt report + CSV export); i18n en+ur `announcements.*`, RTL. Unit: rules (bereavement/opt-in/channels), targeting (5 types), sanitize; e2e `announcements.spec` (auth gates). Registry test + templates catalogue updated.

- **23 — pwa-mobile-shell** — DONE (2026-07-12) — `pwa.core` + `pwa.push`. Per-host manifest/service-worker (host∷BUILD_ID cache), dependency-free web push (RFC 8291/8292), role-aware bottom tab bar + install flow, `PUSH` channel wired. Unit + e2e.

- **22 — platform-billing** — DONE (2026-07-12) — `platform.billing` feature, migration `20260712250000_platform_billing`, monthly run + grace cron, MANUAL payments (+Stripe seam), dashboard/PDF/HTTP + per-society PricingPanel. Unit + DB-gated + e2e.

> Full history: PROGRESS-HISTORY.md · Build order: ARCHITECTURE.md
