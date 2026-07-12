# PROGRESS.md — Rihaish

**Current build-step:** 23 — pwa-mobile-shell — DONE (2026-07-12)
**Last completed:** 23 — pwa-mobile-shell — DONE (2026-07-12)
**Next:** 24 — (see ARCHITECTURE.md build order)

### Recent steps

- **23 — pwa-mobile-shell** — DONE (2026-07-12) — `pwa.core` + `pwa.push`. Migration `20260712260000_pwa_mobile_shell`: `PushSubscription` (per user per device, unique endpoint, pass-through) + `PwaInstall` (adoption per society, @@unique(userId,platform)). White-label manifest now points icons at stable host-scoped generator routes `/api/branding/{icon-192,icon-512,apple-touch-icon}.png` + `/splash?w=&h=` (sharp; branded fallback initial when no `iconFileId`). Per-host **service worker** `/sw.js` (dotted → middleware-skipped): cache name = host∷BUILD_ID (host-scoped, versioned; "Update available — reload" via SKIP_WAITING), network-first navigations w/ offline shell fallback, static SWR, **never caches authed `/api`**, opt-in offline write queue (IDB + Background Sync), push + notificationclick deep-link. **Web push** dependency-free (`lib/pwa/web-push`: RFC 8291 aes128gcm + RFC 8292 ES256 VAPID via node:crypto; round-trip + JWT-verify tested) → `buildPushTransport` fans out per device, prunes 404/410, wired in worker boot; `PUSH` added to `availableChannels` + dispatch carries userId/society/data; PUSH templates for invoice deep-link. Role-aware **bottom tab bar** (`lib/pwa/tabs`: resident/guard/committee, ≤5, entitlement-gated) + **More** off-canvas drawer; safe-area insets, momentum/standalone CSS, `viewport-fit=cover`. **Install flow** (beforeinstallprompt + iOS Add-to-Home sheet, 7-day reminder, adoption tracked) + push enrolment (bounded 2× nag). API `/api/pwa/{push/{public-key,subscribe,unsubscribe},installed}`. i18n en+ur `pwa.*`. Unit: web-push, tabs, push (guards), images, install; e2e `pwa.spec` (manifest/SW headers+host-scoping/icon PNG/apex 404/auth). Desktop unchanged (bottom bar `lg:hidden`).

- **22 — platform-billing** — DONE (2026-07-12) — `platform.billing` feature (deps platform.console, flats.registry, core.worker), perms `platform.billing.read/.manage`, migration `20260712250000_platform_billing` (`PricingProfile`/`PlatformInvoice`/`PlatformPayment`/`PlatformBillingSettings`/`PlatformInvoiceSequence`), separate tables/module/UI from society billing, monthly run + grace-check cron, MANUAL payments (+Stripe seam), dashboard/PDF/HTTP + per-society PricingPanel. Unit + DB-gated + e2e.

- **21 — society-onboarding-wizard** — DONE (2026-07-12) — `platform.onboarding` feature, perms `platform.onboarding.read/.manage`, nav `setup`. Migration `SocietyOnboarding` (resumable draft over an already-ACTIVE tenant; launch = completion + opening-balance LOCK). Full-screen wizard, 10 steps, i18n en+ur. Unit + DB-gated + e2e.

> Full history: PROGRESS-HISTORY.md · Build order: ARCHITECTURE.md
