# GarageAxon — Build Progress

Tracks the module build order, current state, and per-step approval status.
Authoritative per-module detail lives in [`/specs`](./specs). Build one module
per step, on a feature branch, behind the verification gates
(lint · typecheck · build · test), then check off here.

**Checkpoint legend:** `[CHECKPOINT]` = soft (auto-approved after 30s) ·
`[FIXED_CHECKPOINT]` = hard stop, manual human approval required (steps 2, 3, 22).

## Current state

- **Last completed step:** 30 — Call center (DONE, pending approval, 2026-06-19)
- **Status:** step 30 DONE 2026-06-19 (soft `[CHECKPOINT]`; **`[HUMAN_REQUIRED]`** emitted for the live-telephony wiring — the FreePBX/Asterisk VPS + SIP trunk + DIDs + AMI credentials are an operator runbook the agent must NOT touch; the whole app-side is built + proven against mocked events). This is the **last module in the build order.**
- **Feature branch:** `feature/30-call-center`
- **Next step:** — (build order complete)

## Build order

| # | Module | Spec | Checkpoint | Status |
|---|--------|------|-----------|--------|
| 01 | Foundation (repo, scaffolding, CI, deploy.sh, design tokens) | [01-foundation](specs/01-foundation.md) | soft | ✅ DONE — pending approval |
| 02 | Tenancy / RLS | [02-tenancy](specs/02-tenancy.md) | **FIXED** | ✅ DONE — approved 2026-06-17 |
| 03 | Auth + suite interface | [03-auth](specs/03-auth.md) | **FIXED** | ✅ DONE — approved 2026-06-17 |
| 04 | Roles & permissions | [04-roles](specs/04-roles.md) | soft | ✅ DONE — approved 2026-06-17 |
| 05 | Mode framework (Simple/Pro) | [05-mode-framework](specs/05-mode-framework.md) | soft | ✅ DONE — approved 2026-06-17 |
| 06 | Plans & entitlements | [06-plans-entitlements](specs/06-plans-entitlements.md) | soft | ✅ DONE — approved 2026-06-17 |
| 07 | Credits | [07-credits](specs/07-credits.md) | soft | ✅ DONE — approved 2026-06-17 |
| 08 | Super admin | [08-super-admin](specs/08-super-admin.md) | soft | ✅ DONE — approved 2026-06-18 |
| 09 | White-label | [09-white-label](specs/09-white-label.md) | soft | ✅ DONE — approved 2026-06-18 |
| 10 | Settings, audit, i18n | [10-settings-audit-i18n](specs/10-settings-audit-i18n.md) | soft | ✅ DONE — approved 2026-06-18 |
| 11 | Notifications | [11-notifications](specs/11-notifications.md) | soft | ✅ DONE — approved 2026-06-18 |
| 12 | Customers | [12-customers](specs/12-customers.md) | soft | ✅ DONE — approved 2026-06-18 |
| 13 | Vehicles & DVLA | [13-vehicles-dvla](specs/13-vehicles-dvla.md) | soft | ✅ DONE — approved 2026-06-18 |
| 14 | New job pipeline | [14-new-job-pipeline](specs/14-new-job-pipeline.md) | soft | ✅ DONE — approved 2026-06-18 |
| 15 | Workshop / technician | [15-workshop-technician](specs/15-workshop-technician.md) | soft | ✅ DONE — approved 2026-06-18 |
| 16 | QR drop-off & VCR | [16-qr-dropoff-vcr](specs/16-qr-dropoff-vcr.md) | soft | ✅ DONE — approved 2026-06-18 |
| 17 | EVHC authorisation | [17-evhc-authorisation](specs/17-evhc-authorisation.md) | soft | ✅ DONE — approved 2026-06-18 |
| 18 | Estimates & follow-ups | [18-estimates-followups](specs/18-estimates-followups.md) | soft | ✅ DONE — approved 2026-06-18 |
| 19 | Parts & inventory | [19-parts-inventory](specs/19-parts-inventory.md) | soft | ✅ DONE — approved 2026-06-18 |
| 20 | Suppliers & purchasing | [20-suppliers-purchasing](specs/20-suppliers-purchasing.md) | soft | ✅ DONE — approved 2026-06-18 |
| 21 | Accounting engine | [21-accounting-engine](specs/21-accounting-engine.md) | soft | ✅ DONE — approved 2026-06-18 |
| 22 | Invoicing & payments | [22-invoicing-payments](specs/22-invoicing-payments.md) | **FIXED** | ✅ DONE — approved 2026-06-18 |
| 23 | Plans & coverage | [23-plans-coverage](specs/23-plans-coverage.md) | soft | ✅ DONE — approved 2026-06-18 |
| 24 | Courtesy / collection & delivery | [24-courtesy-collection-delivery](specs/24-courtesy-collection-delivery.md) | soft | ✅ DONE — approved 2026-06-18 |
| 25 | Staff & assets | [25-staff-assets](specs/25-staff-assets.md) | soft | ✅ DONE — approved 2026-06-18 |
| 26 | Reporting, KPI, reminders | [26-reporting-kpi-reminders](specs/26-reporting-kpi-reminders.md) | soft | ✅ DONE — approved 2026-06-18 |
| 27 | Customer PWA | [27-customer-pwa](specs/27-customer-pwa.md) | soft | ✅ DONE — approved 2026-06-18 |
| 28 | Smart booking | [28-smart-booking](specs/28-smart-booking.md) | soft | ✅ DONE — approved 2026-06-18 |
| 29 | WordPress plugin | [29-wordpress-plugin](specs/29-wordpress-plugin.md) | soft | ✅ DONE — approved 2026-06-19 |
| 30 | Call center | [30-call-center](specs/30-call-center.md) | soft | ✅ DONE — pending approval |

## Step log

### 30 — Call center — DONE (pending approval), 2026-06-19
The **application-side telephony integration** — the CRM half of a two-server design. The agent built
**every app-side feature** (DID→tenant routing, realtime screen-pop, click-to-call, recording link +
sync consumer, book-from-call, missed-call recovery, the auto-lead pipeline, call history/analytics)
and proved it against **mocked AMI/WebSocket events**, because the phone server itself (FreePBX/Asterisk
VPS + SIP trunk + DIDs + AMI credentials) is a **separate operator runbook on an isolated box the agent
must NOT touch** — so this step **emits `[HUMAN_REQUIRED]`** for the live-telephony wiring with an
operator checklist. App and Asterisk stay isolated; the app reaches the phone server only via the
AMI-bridge consumer + recording sync, using **server-side credentials that are never in the repo**.
No money here; time in whole seconds (UTC-deterministic).

- **Platform safety:** the live bridge is gated on **server-side env only** (`AMI_HOST`/`AMI_PORT`/
  `AMI_USERNAME`/`AMI_PASSWORD` + a shared `AMI_BRIDGE_SECRET`) — never in the repo, surfaced through
  the `AMI_BRIDGE` seam (a `MockAmiBridge` that reports "not connected" until every key is present, the
  same swap-one-binding discipline as the payment-gateway + object-storage seams). The operator
  bridge → app surface (`/call-center/bridge/*`) is **unauthenticated but secret-gated** (shared
  `x-bridge-secret`) and **503s until the secret is configured** (never silently accepts traffic); each
  bridge request names its tenant by the **dialled DID**, resolved to the owning org on the admin client
  before any tenant-scoped write. Recordings are sensitive (GDPR): **permission-gated + audited +
  retention-aware + deletable** (right-to-be-forgotten). The agent never configures Asterisk/SIP itself.
- **Data model** (`packages/db`): three TENANT-SCOPED tables (RLS) — `PhoneNumber` (the app-side
  DID→tenant map: `did` unique per org, `routing` JSON [webrtc/softphone/desk + ring extensions],
  `enabled`), `CallRecord` (the per-call log keyed to the Asterisk `uniqueCallId` [linkedid], unique per
  org so the recording sync can attach later; loose within-org `customerId`/`vehicleId`/`jobId`/
  `handledByUserId`; `recordingKey` [object-storage] + `recordingDeletedAt` for GDPR deletion) and
  `Lead` (the auto-lead pipeline for unknown callers). New enums `CallDirection`/`CallStatus`/
  `LeadSource`/`LeadStatus`. Migration `20260619390000_call_center` (standard tenant-isolation RLS on
  all three).
- **Source of truth + pure maths** (`packages/db/src/call-center.ts`): the status/direction/lead
  catalogues + validators, `normaliseDid`/`phoneMatchKey`/**`matchCustomerByPhone`** (the ONE
  screen-pop match rule — national-significant-tail match, UK trunk-zero/`+44`-aware), the lifecycle
  rules (`callStatusForBridgeEvent`/`callDurationSec`/**`shouldRecoverMissedCall`**/`shouldCreateLead`/
  `leadSourceForCall`), the analytics (`computeCallStats` [missed-call/answer rate, avg duration] +
  `callVolumeByDay`), the **recording-retention** maths (`recordingExpiry`/`isRecordingExpired`,
  default 365 days, overridable via the module-10 `gdpr` setting), the `CALL_CHANNEL` realtime event
  vocabulary, the `call.missed` template key + `CALL_AUDIT_ACTIONS`, and the live-wiring
  **`amiBridgeChecklist`**/`amiBridgeConfigured`/`AMI_BRIDGE_ENV_KEYS` (the HUMAN_REQUIRED hand-off).
  `call-center-provision.ts` (`seedPhoneNumber`/`seedCallRecord`/`seedLead`, edit-preserving). Added the
  `call.missed` notification template (module 11, sms/whatsapp/email) **mapped to the `reminders`
  consent stream** (module 12 — an opted-out customer is never auto-texted), the four `phone.*` RBAC
  permissions (`phone.view`/`phone.call`/`phone.manage`/`phone.recording` + the `Call center` group +
  dependency closure + Owner/Manager full, Receptionist `phone.call`) and the `callcenter` Simple/Pro
  mode entry.
- **API** (`apps/api/src/call-center/`): the `AMI_BRIDGE` seam + `MockAmiBridge`; **`CallEventsService`**
  (the AMI-bridge CONSUMER — idempotent on `uniqueCallId`, screen-pop match + realtime fan-out, the
  missed-call recovery [a module-18 follow-up + a consent-gated, metered module-11 auto-SMS for a known
  customer] and the auto-lead for an unknown caller); **`CallCenterService`** (staff: DID CRUD + routing,
  call history + analytics, the lead pipeline + convert-to-customer, click-to-call through the bridge,
  book-from-call pre-fill, and the GDPR recording playback/delete + the recording-sync `attachRecording`
  over `OBJECT_STORAGE`). `CallCenterController` (`/call-center`, `SessionGuard`+`PermissionGuard`,
  per-route `phone.*`; a `simulate/inbound` proof route) + the secret-gated `CallBridgeController`
  (`/call-center/bridge/*`). `RealtimeService.emitCall` (+ the gateway's generic `broadcastToOrgChannel`)
  fans the screen-pop to the org room on `CALL_CHANNEL`. `CallCenterModule` provides a **bounded
  admin client** (`connection_limit=2`, disconnected on `onModuleDestroy`), wired into `app.module`.
- **Web** (`apps/web/app/call-center`): `/call-center` (`useMode('callcenter')`) — Simple = the everyday
  reception console (live screen-pop on inbound ring, click-to-call softphone, the missed-calls-to-follow-up
  list); Pro adds tabs (Console / History / Leads / Numbers / Analytics) — DID routing config, full call
  history with permission-gated recording playback + delete + retention, the lead pipeline and the animated
  call analytics. Subscribes to the realtime `callcenter` channel for the screen-pop. i18n extended
  (`callCenter.*`, no hardcoded strings); dashboard "Call center" link gated by `phone.view`. Contract:
  `packages/ui/CALL_CENTER_CONTRACT.md`.
- **Seed**: Org A seeds a proof inbound number (DID `+441234500000`), an answered call + a missed call
  (against its first phone-bearing customer) and the auto-lead the missed unknown call creates. Org B left
  empty (empty-state + isolation proof). Idempotent (4 call-center records Org A first run, 0 on re-run).
- **Scope note (disclosed `[HUMAN_REQUIRED]`):** the **live telephony server is operator-provisioned** —
  the agent built everything app-side and proves it against mocked events, and the live wiring stays a
  `[HUMAN_REQUIRED]` hand-off (the staff console shows the operator checklist + the env keys until
  `AMI_HOST`/`AMI_PORT`/`AMI_USERNAME`/`AMI_PASSWORD`/`AMI_BRIDGE_SECRET` are set server-side). The
  WebRTC click-to-call logs the CallRecord and asks the bridge, which returns `humanRequired` (the call
  is logged, not placed) until the trunk is wired. Recordings are stored through the `OBJECT_STORAGE`
  seam (DB-backed in Phase 1, MinIO/S3 later). The realtime fan-out reuses the single-instance Socket.io
  gateway (the pm2 host is single-instance).
- **Harness:** raised the `scripts/verify-db.mjs` embedded-Postgres `max_connections` to 400 — the api
  e2e runs every spec in ONE process (`--runInBand`) and each boots the whole AppModule, so the idle
  admin pools across 30+ modules outgrew Postgres's default 100 (a test-harness-only knob; production uses
  one bounded pool per process). This was the only cross-module change and is **not** a behaviour change.

**Gates:** `lint` ✓ (0 warnings) · `typecheck` ✓ (db/types/api/web) · `build` ✓ (`/call-center` 5.51 kB)
· `test` ✓ (db pure DID/phone-match/lifecycle/recovery/lead/analytics/retention/checklist cases; api unit;
DB-backed suites skip cleanly with no DB). **Verified end-to-end against a real Postgres** (embedded, via
`scripts/verify-db.mjs`): all **28 migrations apply with zero drift** (`migrate diff` → "No difference
detected"); app role `rolsuper=false, rolbypassrls=false`; RLS **enabled on all three new tables**
(`phone_number`/`call_record`/`lead`). The new `call-center.integration.spec.ts` proves DIDs/calls/leads
seed edit-preservingly, the **`(org, did)`** + **`(org, uniqueCallId)`** uniqueness hold, and all three
tables are **RLS-isolated** (cross-tenant write rejected by `WITH CHECK`). The **HTTP call-center e2e**
(`call-center-http.e2e.spec.ts`, 16 cases) boots the full Nest app and proves: a no-token request is 401;
the console shows telephony **NOT connected** (`humanRequired` + the operator checklist + env keys) — the
`[HUMAN_REQUIRED]` surface — and the owner adds a DID (normalised); a simulated inbound **ring from a known
number screen-pops + matches the customer**, then completes; a **missed call from an unknown number
auto-creates a lead + a follow-up**; a **missed call from a known customer fires the consent-gated, metered
recovery SMS** while an **opted-out customer is suppressed**; **click-to-call logs an outbound CallRecord**
(`humanRequired` until the trunk is wired); a **lead converts to a real customer**; **book-from-call
pre-fills** the matched caller; a **recording syncs via the secret-gated bridge (by DID + uniqueCallId),
plays back (permission-gated + audited) and deletes (right-to-be-forgotten → then 404)**; the bridge is
**401 without the secret**; the surface is **permission-gated** (technician 403; receptionist views via the
`phone.call`⇒`phone.view` closure but is 403 on managing/playback) and **tenant-isolated** (Org B sees no
calls/DIDs/leads); and **setting the AMI env flips telephony to connected**. Full **db suite (55 suites /
500 tests)** + **api suite (36 suites / 243 tests)** pass against the live DB — no cross-module regressions.
Seed idempotent (4 call-center records Org A first run, 0 on re-run; Org B 0). Live screenshot pending
deploy. **Ends at `[CHECKPOINT]` — with `[HUMAN_REQUIRED]` emitted for the live-telephony wiring.**

### 29 — WordPress booking plugin — DONE (pending approval), 2026-06-19
A standalone **PHP WordPress plugin** (`wordpress-plugin/`, NOT part of the Next/Nest build) so the
many UK garages on WordPress add GarageAxon online booking in a few clicks. It is a **thin client over
the module-28 Smart Booking API**: the garage pastes only their **PUBLIC `garageKey`** (no secret ever
lives in WordPress), drops in the `[garageaxon_booking]` shortcode or the Gutenberg block, and bookings
write into the **same Job pipeline** (module 14) — never a parallel system. Distributed two ways: a
**direct-download `.zip`** served by the API + linked from the GarageAxon admin, and the **WordPress.org
directory** (the directory **submission is a human/operator step** — the agent built + packaged a compliant
plugin and wrote the runbook). This was the **5th booking surface** module 28 already reserved
(`source: 'wordpress'`).

- **Platform safety:** the plugin holds **no secret** — only the public `garageKey` (rate-limited +
  validated upstream). Writes go through the **public** booking-create endpoint designed for browser/plugin
  surfaces (the partner-API write-secret path is untouched and stays server-side on GarageAxon). The plugin
  never calls home beyond the documented module-28 API; every PHP file blocks direct access (`ABSPATH`
  guard), all output is escaped and all input sanitised.
- **The plugin** (`wordpress-plugin/`, PHP 7.4+, WP 5.8+): the main file (header/metadata, activation
  defaults, textdomain); `Gax_Booking_Api` (the WordPress-free, unit-tested thin client — URL building +
  garage-key sanitisation + create-payload shaping, attributed to the `wordpress` source); a **settings page**
  (garage key / heading / optional service filter / API base + a live **Test connection**); a **same-origin
  REST proxy** (`Gax_Booking_Rest` — the browser talks only to WordPress, which forwards to GarageAxon, so
  there is no CORS; a courtesy per-IP create throttle on top of the upstream limit; the test route is
  admin-only); the **`[garageaxon_booking]` shortcode** + a **dynamic Gutenberg block** (server-rendered via
  the same shortcode path, so editor and front end never drift); a dependency-free, accessible **front-end
  widget** (reg → free live DVLA → service → capacity-aware slot → details + marketing consent → optional
  deposit → reference) branded by the theme via CSS custom properties (`--gax-accent`, dark-mode aware);
  `uninstall.php`; the i18n `languages/garageaxon-booking.pot`; and a WordPress.org-format `readme.txt`
  (stable tag, tested-up-to, GPL, FAQ, screenshots, changelog).
- **Packaging + distribution:** `scripts/package-wordpress-plugin.mjs` stages the runtime files under the
  `garageaxon-booking/` slug folder and zips them (dev artefacts — `dist/`, the PHP tests, the operator
  checklist — excluded) → `wordpress-plugin/dist/garageaxon-booking.zip` (gitignored). The API serves it at
  **`GET /booking/public/wordpress-plugin.zip`** (no tenant context — no secrets; located via
  `WORDPRESS_PLUGIN_ZIP` or the conventional `wordpress-plugin/dist` path; clear 404 until built).
  `wordpress-plugin/SUBMISSION-CHECKLIST.md` is the operator's WordPress.org runbook (validate readme, build,
  prepare assets, submit, SVN, updates).
- **API + types (module-28 touch points, contract kept stable):** `browserSource` now accepts the
  `wordpress` source on the public create path (attribution only — the create path is identical);
  `BookingService.surfaces` adds `wordpressDownloadUrl` (+ `wordpressDirectoryUrl`, null until the operator
  publishes via `WORDPRESS_DIRECTORY_URL`); `PublicBookingController` adds the `.zip` download route.
  `BookingSurfacesView` extended (types).
- **Web:** the staff `/booking` **Get bookings** surfaces grid gains a **WordPress plugin** card — the public
  `garageKey` (copyable), the direct-download link, the optional directory link (once published) and the
  two-step setup note — so the plugin "appears as a booking surface in the GarageAxon admin with download +
  key + steps" (spec). i18n extended (`booking.wordpress*`).
- **No data model / migration changes** (the plugin is a thin client over the module-28 engine; the
  `wordpress` source already existed in `BOOKING_SOURCES`). No new RBAC/mode/notification entries.

**Scope note (disclosed):** the **WordPress.org directory submission** is a human/operator step (it needs a
WordPress.org account + their manual review queue) — the agent built + packaged a directory-ready plugin and
wrote `SUBMISSION-CHECKLIST.md`; the operator submits. This is a **post-build distribution task, NOT a
`[HUMAN_REQUIRED]` build-stop** (the spec is explicit). The screenshot of the rendered form in a live
WordPress instance is pending the operator standing up a test WP site / the staging deploy; the form's
behaviour is proven by the API e2e + the PHP unit tests below.

**Gates:** `lint` ✓ (0 warnings) · `typecheck` ✓ (db/types/api/web) · `build` ✓ (`/booking` 6.18 kB —
now with the WordPress surface card) · `test` ✓ (api unit; DB-backed suites skip cleanly with no DB) ·
**PHP** ✓ (`php -l` clean on all 6 plugin files; `php wordpress-plugin/tests/test-api.php` → **23 checks,
0 failed** over the URL-building / garage-key-sanitisation / create-payload-shaping helpers) · **packaging**
✓ (`node scripts/package-wordpress-plugin.mjs` → `wordpress-plugin/dist/garageaxon-booking.zip`, an
18-file archive under the `garageaxon-booking/` slug folder; dev artefacts excluded). **Verified
end-to-end against a real Postgres** (embedded, via `scripts/verify-db.mjs`): all **27 migrations apply
with zero drift** (`migrate diff` → "No difference detected" — module 29 adds **no migration**); app role
`rolsuper=false, rolbypassrls=false`; RLS still enabled on `online_booking`/`booking_config`. The
**HTTP booking e2e** (`booking-http.e2e.spec.ts`) gains **2 module-29 cases** proving: the staff dashboard
**advertises the WordPress plugin surface** (a `/booking/public/wordpress-plugin.zip` download URL +
a null directory URL until published); a **public booking with `source: 'wordpress'` creates (pending),
lands in Org A's inbox and is stored with the `wordpress` source** (the plugin proxies the public create
with the key alone — no secret); and the **`.zip` download route 404s when no package is present and
streams `application/zip` as an attachment when it is**. Full **db suite (53 suites / 479 tests)** + the
**api suite (35 suites / 228 tests** — +2 from the new booking e2e cases) pass against the live DB — no
cross-module regressions
(the module-28 booking contract is unchanged for existing surfaces). Seed idempotent (3 smart-booking
records Org A first run, 0 on re-run; Org B 0). Live screenshot pending deploy. **Ends at `[CHECKPOINT]`
(the WordPress.org directory submission is a separate operator task, NOT a build-stop).**

### 28 — Smart booking — APPROVED 2026-06-18 (operator instruction)
Approved by the human operator (soft checkpoint). See entry below for build detail.

### 28 — Smart booking — DONE (pending approval), 2026-06-18
ONE shared booking engine exposed through **five public surfaces** so any garage takes bookings
online: (1) a JS **embed widget**, (2) a **hosted page** (platform domain + SSL, zero setup),
(3) a custom-domain **microsite**, (4) a documented **booking API**, and (5) the WordPress
plugin (module 29). All write into the SAME Job pipeline (module 14) — a booking is never a
parallel system. Built on module 09 (branding), 11 (notifications), 12 (consent), 13 (free LIVE
DVLA), 14 (jobs), 22 (sandbox deposit gateway). Money in MINOR units (pence). The
custom-domain/SSL sub-task is **`[HUMAN_REQUIRED]`** (the agent built the mapping + runbook but
cannot provision DNS/certs).

- **Platform safety:** the public **`garageKey`** (`gk_…`) only NAMES the tenant for a public
  embed/api request (safe to expose, rate-limited). The partner-API **write secret** (`bsk_…`)
  is shown ONCE at issue and **only its sha256 hash is stored** (`hashBookingSecret`) — never the
  secret, never sent to the web (same discipline as the customer-app bearer + payment-gateway
  secret). Browser surfaces create with the key alone; the partner API write path additionally
  requires the secret. Optional deposits charge the **SandboxPaymentGateway** seam (no real funds
  on staging). The custom-domain connect returns the DNS runbook but the agent NEVER provisions
  DNS/certs — that is the operator step.
- **Data model** (`packages/db`): two TENANT-SCOPED tables (RLS) — `BookingConfig` (one per org:
  `services`/`availabilityRules`/`requiredFields` JSON, deposit, the globally-unique public
  `garageKey`, the hashed `apiSecretHash`, `autoConfirm`, the `customDomain` (unique) +
  `domainStatus`/`domainVerifyToken`) and `OnlineBooking` (`vrm`, name, contact, requested
  slot/service, `status` pending→confirmed→converted/rejected, `source` embed|hosted|microsite|
  api|wordpress, the captured DVLA snapshot + 3-stream consent + deposit, loose within-org
  `jobId`/`customerId`/`vehicleId`). New enums `BookingDomainStatus`/`OnlineBookingStatus`/
  `OnlineBookingSource`. Migration `20260618380000_smart_booking` (standard tenant-isolation RLS
  on both).
- **Source of truth + pure maths** (`packages/db/src/booking.ts`): `hashBookingSecret` (the ONE
  one-way secret hash) + key/secret prefixes, the **`computeAvailableSlots`** slot engine
  (working-days × open/close × slot length × capacity − blackouts − lead − horizon, UTC-determin-
  istic) + `isSlotOffered`/`slotKey(ForInstant)`, the `normaliseServices`/`normaliseAvailability-
  Rules`/`normaliseRequiredFields` defensive normalisers + sane UK defaults, `normaliseDomain`/
  **`domainDnsRecords`** (the CNAME+TXT runbook), `bookingChannelFor`/`normaliseContact`/
  `maskContact`, the source/status catalogues and `BOOKING_AUDIT_ACTIONS`. `booking-provision.ts`
  (edit-preserving `seedBookingConfig`/`seedOnlineBooking`). Added `booking.received`+
  `booking.confirmed` notification templates (module 11, operational) and the `booking.manage`
  RBAC permission (dep on `booking.view`; Owner via `*`, Manager) — `booking.view`/`.create`/
  `.edit` + the `bookings` Simple/Pro mode entry already existed.
- **API** (`apps/api/src/booking/`): **`PublicBookingService`** (EXPORTED) — the unauthenticated
  shared engine: resolve a tenant by garageKey/slug/custom-domain on its OWN `ADMIN_DB`, render
  the engine, compute capacity-aware availability, the free LIVE-DVLA confirm (no row created),
  and **`createBooking`** (validate required fields + that the slot is offered & has capacity →
  free DVLA snapshot → consent → optional sandbox deposit → OnlineBooking → branded
  acknowledgement → auto-confirm+convert via `JobsService.create` when configured). The partner
  write path verifies the hashed secret. **`BookingService`** (EXPORTED) — staff config CRUD
  (auto-provisions a config + key on first read), key rotation, write-secret issue (plaintext
  ONCE), custom-domain connect (→ DNS runbook + `humanRequired`) + verify (stays pending — no
  auto-provision), the inbox, and **accept** (opens the real Job through the ONE pipeline) +
  reject. A dependency-free in-memory **`BookingRateLimiter`** guards the public endpoints.
  Controllers: `BookingController` (`/booking`, SessionGuard+PermissionGuard — `booking.view`
  read, `booking.create` accept/reject, `booking.manage` setup), `PublicBookingController`
  (`/booking/public/*`, unauthenticated browser surfaces) and `BookingApiController`
  (`/booking/api/v1/*` — `services`/`availability`/`bookings` + `openapi.json`; writes need
  key + `X-Booking-Secret`). Sandbox `PAYMENT_GATEWAY` + Mock `VEHICLE_DATA_PROVIDER` + own
  `ADMIN_DB` provided in `BookingModule`, wired into `app.module`.
- **Web** (`apps/web`): staff `/booking` (`useMode('bookings')`) — Simple = the request inbox +
  the "get bookings" surfaces; Pro adds tabs (Requests / Get bookings / Services & hours / API &
  domain) with the service+hours+deposit editor, key rotation, write-secret issue and the
  custom-domain connect (renders the DNS runbook + the `[HUMAN_REQUIRED]` note). Public surfaces:
  the ONE branded `BookingWidget` (reg → live DVLA → slot → details → optional deposit) reused by
  the **hosted** `/book/[slug]`, the **embed** `/book/embed?g=<key>` (+ the `/embed.js` loader
  route that injects the iframe) and the **microsite** `/site/[key]` (+ `/site` host-resolved for
  a custom domain, with `middleware.ts` mapping a configured custom host's root). Branded at
  runtime via `brandStyleVars`/`logoSrc` off `/branding/public/:orgId`; public client
  `lib/booking-api.ts` (no credentials, embed-safe). i18n extended (`booking.*`, no hardcoded
  strings); money via `formatCurrency`; dashboard "Online booking" link gated by `booking.view`.
  Contract: `packages/ui/BOOKING_CONTRACT.md`.
- **Seed**: Org A seeds a working BookingConfig (services off its price list + default hours + a
  public garageKey + a HASHED API write secret), a pending request in the inbox and a second
  request already converted to a seeded job (so the inbox + conversion analytics have content).
  Org B left empty (empty-state + isolation proof). Idempotent (3 booking records first run).
- **Scope note (disclosed):** the **custom-domain microsite** needs the operator to point the
  garage's DNS at us + issue the TLS cert — the agent built the domain-mapping (`/site` + the
  `BOOKING_MICROSITE_HOSTS` middleware), the connect flow + the DNS runbook, and emits
  **`[HUMAN_REQUIRED]`** for the DNS/cert step (it never auto-provisions). The deposit is a sandbox
  gateway charge recorded on the booking (no invoice yet — that is taken when the job invoices,
  module 22). The booking API rate limiter is a single-instance in-memory fixed-window (no Redis
  offline; the pm2 host is single-instance) — a Redis-backed limiter is the production swap.

**Gates:** `lint` ✓ (0 warnings) · `typecheck` ✓ (db/types/api/web) · `build` ✓ (`/booking` 6.09
kB; `/book/[slug]`, `/book/embed`, `/site`, `/site/[key]` server-rendered; `/embed.js` route;
Middleware 26.5 kB) · `test` ✓ (db 14 new pure secret-hash/slot-engine/normaliser/domain/contact
cases; api unit; DB-backed suites skip cleanly with no DB). **Verified end-to-end against a real
Postgres** (embedded, via `scripts/verify-db.mjs`): all **27 migrations apply with zero drift**
(`migrate diff` → "No difference detected"); app role `rolsuper=false, rolbypassrls=false`; RLS
**enabled on both new tables** (`booking_config`/`online_booking`). The new
`booking.integration.spec.ts` (3 cases) proves config/bookings seed edit-preservingly, the
**`garage_key` is globally unique** (a second org cannot reuse it) and both tables are
**RLS-isolated** (cross-tenant write rejected by `WITH CHECK`). The **HTTP booking e2e**
(`booking-http.e2e.spec.ts`, 11 cases) boots the full Nest app and proves: the owner opens the
dashboard (config **auto-provisions with a public garageKey**) + configures services/hours; the
**PUBLIC engine renders by garageKey**, availability returns slots, a reg → **live DVLA** (Ford
Fiesta); a **public booking creates (pending)** + lands in the staff inbox; a missing required
field is **400**; staff **accept → a real Job opens** (status `converted`, jobNo); the **partner
API creates with key + secret** and is **401 without the secret**, and `openapi.json` serves; an
**auto-confirm garage converts on submit**; a **custom-domain connect returns the DNS runbook +
`humanRequired`** (no auto-provision); the surface is **permission-gated** (technician 403;
receptionist accepts but is 403 on configuring) and **tenant-isolated** (Org B sees only its own
bookings). Full db suite (**53 suites / 479 tests**) + api suite (**35 suites / 226 tests**) pass
against the live DB — no cross-module regressions. Seed idempotent (3 booking records Org A first
run, 0 on re-run; Org B 0). Live screenshot pending deploy. **Ends at `[CHECKPOINT]` (with
`[HUMAN_REQUIRED]` emitted for the custom-domain DNS/SSL sub-step).**

### 27 — Customer App (PWA) — APPROVED 2026-06-18 (operator instruction)
Approved by the human operator (soft checkpoint). See entry below for build detail.

### 27 — Customer App (PWA) — DONE (pending approval), 2026-06-18
The customer-facing **Progressive Web App** — a branded, installable, dead-simple app the garage's
customers launch from the garage's website/booking link/QR (no app store, no staff login). It is a
**FACE over the existing domain services**, never a parallel system: light magic-link/OTP auth mints a
**customer-scoped bearer session**, then bookings/history, approve-decline recommended work, view/pay
invoices, wallet + top-up, packages/coverage usage and self-service consent all REUSE the module
14/17/22/23/12 services under one customer scope. **No Simple/Pro split** (dead-simple by nature — the
mode framework governs staff surfaces only); no staff RBAC/mode entries. Money in MINOR units (pence).

- **Platform safety (auth):** **no OTP code, magic-link token or session bearer is ever stored in the
  clear** — only their sha256 hash (`hashCustomerToken`); a lookup recomputes the hash from the presented
  secret. Anti-enumeration: an OTP request for an unknown identifier returns the SAME shape as a match. The
  session bearer lives in the customer's browser (localStorage, scoped per garage), replayed via
  `Authorization: Bearer` — never the staff session cookie. The OTP request returns a `devCode` ONLY when
  `NODE_ENV !== 'production'` (the same sandbox-vs-real discipline as the payment gateway); never in prod.
  Online payments + wallet top-ups charge the **SandboxPaymentGateway** seam (no real funds on staging).
- **Data model** (`packages/db`): two TENANT-SCOPED tables (RLS) — `CustomerAuthChallenge` (the OTP/
  magic-link challenge: hashed `codeHash`/`tokenHash`, `expiresAt`, single-use `consumedAt`, `attempts`
  brute-force cap) and `CustomerAppSession` (the bearer session: **globally-unique `tokenHash`** for the
  cross-tenant guard lookup, `expiresAt`, `revokedAt`). Migration `20260618370000_customer_app` (standard
  tenant-isolation RLS on both). No new money source of truth.
- **Source of truth + pure maths** (`packages/db/src/customer-app.ts`): `hashCustomerToken` (the ONE one-way
  hash), TTLs/caps (`CUSTOMER_OTP_TTL_MINUTES`/`SESSION_TTL_DAYS`/`MAX_ATTEMPTS`), `otpChallengeExpiry`/
  `sessionExpiry`, **`challengeUsability`** (expired/consumed/locked) + **`isSessionActive`**, `normaliseOtpCode`/
  `maskIdentifier` (anti-leak display), **`friendlyJobStatus`** (the customer-facing plain-English mapping over
  the module-14 pipeline via the module-05 Simple buckets), **`walletBalanceMinor`** (sum of active prepaid
  holdings) and `CUSTOMER_APP_AUDIT_ACTIONS`. `customer-app-provision.ts` (edit-preserving seed helpers).
  Added the `customer.app.otp` notification template (module 11, operational/transactional — incl. a magic link).
- **API** (`apps/api/src/customer-app/`): `CustomerSessionGuard` (bearer → ADMIN_DB token-hash resolve →
  active check → stamps tenant context + attaches `{orgId, customerId, sessionId}`; fail-closed); a
  `@CurrentCustomer` decorator. `CustomerAuthService` (EXPORTED) — garage context, OTP request (sends via the
  hub, mints hashed challenge), verify (+ attempt cap), magic-link consume, logout, profile. `CustomerAppService`
  (EXPORTED) — home/bookings/vehicles (friendly status + MOT standing), approvals (REUSES `EvhcService`
  authorise + a customer-ownership guard, tracked), invoices + **pay online** (REUSES `InvoicingService.takePayment`
  → same gateway charge + ledger posting), wallet + **top-up** (sandbox charge → `CoverageService.sell` credits
  the wallet + posts the journal), coverage usage ledger, and self-service consent (REUSES
  `CustomersService.setConsent` — so a marketing opt-out suppresses marketing sends everywhere) + recent messages.
  `PublicCustomerAppController` (`/customer-app/auth/*`, unauthenticated) + `CustomerAppController`
  (`/customer-app/*`, behind `CustomerSessionGuard`); the sandbox `PAYMENT_GATEWAY` + own `ADMIN_DB` are provided
  in `CustomerAppModule`, wired into `app.module`.
- **Web** (`apps/web/app/app`): an installable PWA (`/app` scope, `app.webmanifest` + `app-sw.js` +
  `CustomerPwaRegister`), branded per garage at runtime (`brandStyleVars`/`logoSrc` off `/branding/public/:orgId`).
  `/app` is a mobile-first SPA: OTP login (garage from `?g=`/`?org=` then remembered) → a tabbed dashboard
  (Home · Bookings · Approvals · Invoices · Account) with Vehicles/Wallet/Plans reachable from Home tiles —
  bookings/history in friendly status, EVHC approve/decline per item, pay an invoice online, top up + view the
  wallet, the coverage "covered/used/left/expires" ledger, and the three-stream consent toggles + messages.
  `/app/m/[token]` consumes the magic link. A bearer client (`lib/customer-api.ts`, localStorage token).
  i18n extended (`customerApp.*`, no hardcoded strings); money via `formatCurrency`; skeletons + error/retry +
  empty states + large touch targets throughout. Contract: `packages/ui/CUSTOMER_APP_CONTRACT.md`.
- **Seed**: Org A seeds a historical (consumed) sign-in challenge + an active app session against its first
  emailable customer (HASHED secrets — proof content + RLS isolation visible). The bookings/EVHC/invoice/wallet/
  membership the app shows are the records modules 14/17/22/23 already seed. Org B left empty (isolation proof).
  Idempotent. The live demo signs in via the OTP dev code.
- **Scope note (disclosed):** OTP delivery rides the module-11 hub (best-effort); the `devCode` (non-prod only)
  is the reliable staging/test path. Wallet "top-up" buys a prepaid pack (pay X → get Y) via the sandbox gateway
  → `CoverageService.sell` (each top-up is a holding; the wallet = sum of active holdings). Online pay/top-up
  require the garage to have connected a gateway (sandbox on staging — never real funds). A real-charge binding is
  intentionally not implemented.

**Gates:** `lint` ✓ (0 warnings) · `typecheck` ✓ (db/types/api/web) · `build` ✓ (`/app` 8.88 kB,
`/app/m/[token]` server-rendered) · `test` ✓ (db 22 new pure token-hash/expiry/usability/session/OTP/friendly-
status/wallet cases; api unit; DB-backed suites skip cleanly with no DB). **Verified end-to-end against a real
Postgres** (embedded, via `scripts/verify-db.mjs`): all **26 migrations apply with zero drift** (`migrate diff` →
"No difference detected"); app role `rolsuper=false, rolbypassrls=false`; RLS **enabled on both new tables**
(`customer_auth_challenge`/`customer_app_session`). The new `customer-app.integration.spec.ts` (3 cases) proves
challenges/sessions seed edit-preservingly, the **session token hash is globally unique** (a duplicate rejected,
a bearer resolves to one session) and both tables are **RLS-isolated** (cross-tenant write rejected by
`WITH CHECK`). The **HTTP customer-app e2e** (`customer-app-http.e2e.spec.ts`, 14 cases) boots the full Nest app
and proves: a **no-token request is 401**; a customer **requests an OTP (dev code) + verifies → a bearer
session**; a **wrong code is 400**; an **unknown identifier doesn't disclose non-existence** (no dev code); the
bearer's **home** shows only their own active job + pending approval + £120 outstanding + £50 wallet + 2 plans;
**bookings** split active/history in friendly terms; **vehicles** show MOT due-soon; the customer **approves work
in-app** → the job advances to `approved`, the response is tracked (system-actor audit); they **pay an
outstanding invoice online (sandbox)** → it clears (`sandbox_…` ref) and a customer-payment journal posts; they
**top up the wallet** (£200 → +£220 credit via the coverage sell path, balance £50 → £270); **coverage holdings**
render; a **marketing opt-out via the app suppresses** a staff marketing send (`sent:false, opted_out`); **customer
isolation** holds (another customer's invoice is 404); and **logout revokes** the session (then 401). Full db suite
(**51 suites / 462 tests**) + api suite (**34 suites / 215 tests**) pass against the live DB — no cross-module
regressions. Seed idempotent. Live screenshot pending deploy. **Ends at `[CHECKPOINT]`.**

### 26 — Reporting, KPI, reminders — APPROVED 2026-06-18 (operator instruction)
Approved by the human operator (soft checkpoint). See entry below for build detail.

### 26 — Reporting, KPI, reminders — DONE (pending approval), 2026-06-18
The analytics + automated-communication layer over the whole app: **per-job profitability
(Job P&L)**, an owner **KPI dashboard**, the familiar **report suite** (with date ranges +
CSV export) and the rules-based **reminder/recall engine** (MOT/service reminders — timing-belt
dropped). Job P&L + KPIs are DERIVED from existing module data (jobs/invoices/parts/clocking/
accounting) — this module owns NO new money source of truth; the only persisted state is the
reminder rules + instance log + saved report definitions. Built on module 11 (notify hub),
12 (consent stream), 13 (DVSA MOT cache), 14 (jobs/clocking), 19–25 (parts/accounting/staff/etc).
Money in MINOR units (pence); time in whole seconds.

- **Data model** (`packages/db`): three TENANT-SCOPED tables (RLS) — `ReminderRule`
  (`type` mot|service|recall|custom, `channel`, `templateKey`, `leadDays`, `intervalMonths`,
  `enabled`, `segment`), `ReminderInstance` (the append-mostly LOG; **unique (org, dedupeKey)**
  ⇒ the daily scheduler is idempotent — never double-sends; `notificationId`/`reason`/`status`)
  and `ReportDefinition` (saved/custom reports — `key`+`params`). New enums `ReminderRuleType`/
  `ReminderInstanceStatus`. Migration `20260618360000_reporting_kpi_reminders` (standard
  tenant-isolation RLS on all three). Job P&L has **no table** (derived per spec).
- **Source of truth + pure maths** (`packages/db/src/reporting.ts`): **`computeJobPnl`** (the ONE
  P&L rule: revenue − parts/labour/sublet cost → gross profit + margin) + `rollupPnl` (by day/
  branch/technician), `REPORT_DEFINITIONS` (the 14-report catalogue across the six familiar
  families: sales/customer/labour/stock/accounting/personnel) + `isReportKey`/`reportDefinition`,
  **`toCsv`** (the ONE RFC-4180 serialiser the export uses), the reminder-engine maths
  (`reminderScheduledSendAt`/`isReminderDue`/**`reminderDedupeKey`** [the idempotency key]/
  `serviceDueDate`/`daysBetween`/`defaultTemplateKeyForType`), `REPORT_AUDIT_ACTIONS`/
  `REMINDER_AUDIT_ACTIONS`. `reporting-provision.ts` (`seedReminderRule`/`seedReminderInstance`/
  `seedReportDefinition`, edit-preserving). Added `service.reminder`+`recall.campaign`
  notification templates (module 11) + their consent-stream mapping (module 12, → reminders),
  the `reminders.serviceIntervalMonths` setting (module 10), and `report.export`+`reminder.manage`
  RBAC (deps on `report.view`; Owner via `*`, Manager + Accountant export, Manager reminder.manage).
  The `reporting` Simple/Pro mode entry + `report.view` already existed.
- **API** (`apps/api/src/reporting/`): **`ReportingService`** (EXPORTED) — `jobPnl` (batched, no
  N+1: invoice net revenue, used-on-job movement × product cost, clocked-seconds-attributed ×
  staff pay rate, sublet lines), `kpi` (tiles + jobs-by-status + takings chart; composes the
  module-21 `AccountingService` month profit/VAT + module-25 `StaffService` efficiency + credit
  balance, all guarded), the `report`/`exportCsv` suite (14 builders) and saved definitions.
  **`RemindersService`** (EXPORTED) — rules CRUD, `dueList` (who's-due preview with consent flag),
  **`runDueReminders`** (the engine — finds due MOT/service entities, sends by REUSING
  `CustomersService.notifyCustomer` so consent[12]+metering[07]+fallback[11] are enforced on the
  ONE path, logs an idempotent instance), `recall` (manual batch campaign to a segment) and the
  instance log. **`ReminderSchedulerService`** — a dependency-free hourly in-process tick (BullMQ
  was never delivered by spec 01 and is unavailable offline) that evaluates every org via its OWN
  admin client (cross-tenant enumeration only); disabled in tests / `REMINDERS_SCHEDULER=off`;
  idempotent so waking often never double-sends. `ReportingController` (`/reporting`) +
  `RemindersController` (`/reminders`), `SessionGuard`+`PermissionGuard`: reads `report.view`,
  export/save-definitions `report.export`, rule edits/run/recall `reminder.manage`. CSV export via
  an `@Res` stream (`text/csv` + Content-Disposition). `ReportingModule` wired into `app.module`.
- **Web** (`apps/web`): `/reporting` — `useMode('reporting')`. Simple = the glanceable KPI
  dashboard (tiles + two animated charts) + the reminders panel (on/off banner, who's-due list,
  one-tap "send due reminders now"); Pro adds tabs (Dashboard / Reports / Job profitability /
  Reminders) — the report suite (report picker + date range + CSV export + save), the per-job P&L
  with day/branch/technician roll-up + headline totals, the reminder rule editor + recall campaign
  form + instance log. i18n extended (`reporting.*`, no hardcoded strings; added `common.cancel`);
  money via `formatCurrency`; animated bar charts via `ChartContainer`; skeletons; dashboard
  "Reports & KPIs" link gated by `report.view`. Contract: `packages/ui/REPORTING_CONTRACT.md`.
- **Seed**: Org A seeds an MOT + a service reminder rule, a "this-month profitability" saved
  report definition and one logged (sent) MOT reminder instance against an existing vehicle+owner
  (so the log + dedupe index have content). Job P&L / KPI / reports render off the jobs/invoices/
  parts/clocking already seeded. Org B left empty (empty-state + isolation proof). Idempotent.
- **Scope note (disclosed):** the spec referenced BullMQ (spec 01) for the daily scheduler, but
  no queue infrastructure was ever delivered and none is available offline — so the scheduler is a
  dependency-free in-process hourly tick (engine idempotent per day; single-instance pm2 host) plus
  a manual `POST /reminders/run`. "PDF export" is the browser print of the CSV-backed table (the
  established customer-doc pattern); CSV export is server-side.

**Gates:** `lint` ✓ (0 warnings) · `typecheck` ✓ (db/types/api/web) · `build` ✓ (`/reporting`
6.4 kB) · `test` ✓ (db 13 new pure Job-P&L/rollup/CSV/catalogue/reminder-maths cases; api unit;
DB-backed suites skip cleanly with no DB). **Verified end-to-end against a real Postgres**
(embedded, via `scripts/verify-db.mjs`): all **24 migrations apply with zero drift** (`migrate
diff` → "No difference detected"); app role `rolsuper=false, rolbypassrls=false`; RLS **enabled on
all three new tables** (`reminder_rule`/`reminder_instance`/`report_definition`). The new
`reporting.integration.spec.ts` (4 cases) proves rules/instances/report-definitions seed
edit-preservingly, the **reminder-instance dedupe key is UNIQUE within an org** (a re-run is a
no-op; a direct duplicate rejected), and all three tables are **RLS-isolated** (cross-tenant write
rejected by `WITH CHECK`). The **HTTP reporting e2e** (`reporting-http.e2e.spec.ts`, 8 cases) boots
the full Nest app and proves: the **KPI dashboard** renders live tiles + jobs-by-status; **Job P&L
computes** revenue £200 − parts £50 − labour £40 → GP £110, margin 55%; a **report renders +
exports CSV** (with header); a **saved report definition round-trips**; an **MOT rule finds a due
vehicle, the engine SENDS it via the hub** (consent + metered, real notification logged) + **a
re-run is idempotent** (0 sent, skipped); an **opted-out customer is SUPPRESSED**; a **recall
campaign sends to a `make` segment**; and the surface is **permission-gated** (technician 403 on
KPI; receptionist 403 on `reminder.manage`) and **tenant-isolated** (Org B sees no rules). Full db
suite (**49 suites / 437 tests**, incl. reporting integration) + api suite (**33 suites / 201
tests**, incl. reporting e2e) pass against the live DB — no cross-module regressions. Seed
idempotent (2 reminder rules / 1 instance Org A first run, 0 on re-run; Org B 0). Live screenshot
pending deploy. **Ends at `[CHECKPOINT]`.**

### 25 — Staff & assets — APPROVED 2026-06-18 (operator instruction)
Approved by the human operator (soft checkpoint). See entry below for build detail.

### 25 — Staff & assets — DONE (pending approval), 2026-06-18
Two related admin modules under one mode key (`staff`) and one RBAC group (`Staff &
assets`). **A. Staff** — the people: garage **staff profiles** that EXTEND a User
(module 03) with a short clock-in `loginNumber`, the granular display/report toggles
(`showOnCalendar`/`showInCurrentStates`/`includeInEfficiencyReport`), an hourly pay
rate and notes; simple **attendance** clock in/out; **holiday/leave** booking →
approval → calendar; **technician notes**; a **productivity/efficiency** report
DERIVED from job clocking (module 14) vs attendance (respecting
`includeInEfficiencyReport`); and **staff pay** recording that posts a balanced
journal to the ledger (module 21: DR Wages / CR Bank net + PAYE-NI control). **B.
Assets & Tools** — the equipment: the **asset register** (ramps, diagnostic kit,
jacks, MOT-kit, tools), the legally-important **calibration/service due-date tracking
+ reminders** (module 11), **tool checkout** to a technician, a **maintenance log**
whose cost posts a balanced Repairs & maintenance expense (DR Repairs / CR Bank), and
straight-line **depreciation** (DR Depreciation / CR Accumulated depreciation, never
past cost). Money in MINOR units (pence); time in whole seconds. Privileged changes
audited (module 10).

- **Data model** (`packages/db`): six TENANT-SCOPED tables (RLS) — `StaffProfile`
  (per-(org,user); per-(org,loginNumber) unique), `Attendance` (`clockIn`/`clockOut`/
  `date`), `Holiday` (`type`/`from`/`to`/`status`/`approvedByUserId`), `TechnicianNote`,
  `Asset` (`category`, `serialNo`, `calibrationDueDate`/`serviceDueDate`,
  `assignedToUserId`, `purchaseValueMinor`, `depreciationMonths`,
  `accumulatedDepreciationMinor`, `status`) and `AssetMaintenanceLog`
  (`type`/`costMinor`/`byUserId`). New enums `HolidayType`/`HolidayStatus`/
  `AssetCategory`/`AssetStatus`/`AssetMaintenanceType`. Migration
  `20260618350000_staff_assets` (standard tenant-isolation RLS on all six). Added the
  chart accounts `repairs`/`depreciation` (expense) + `accumulated_depreciation`
  (contra-asset) to the default chart (`NOMINAL`, additive — no migration drift).
- **Source of truth + pure maths** (`packages/db/src/staff.ts`, `assets.ts`):
  `attendanceSeconds`/`isClockedIn`/`secondsToHours`, `HOLIDAY_TYPES`/`STATUSES` +
  `holidayDays`/`isOnHolidayOn`, `attributeJobSeconds`/**`efficiencyPercent`** (the ONE
  efficiency rule: job hours ÷ attended hours), `normaliseLoginNumber`/`payAmountMinor`/
  **`staffPaySplit`**, `STAFF_AUDIT_ACTIONS`; `ASSET_CATEGORIES`/`STATUSES`/
  `MAINTENANCE_TYPES`, **`assetDueReminders`** (overdue/due-soon, retired→none),
  `isAssetCheckedOut`, **`bookValueMinor`/`monthlyDepreciationMinor`/
  `depreciationChargeMinor`** (never past cost), `ASSET_AUDIT_ACTIONS`,
  `ASSET_REMINDER_TEMPLATE_KEY`. `staff-provision.ts`/`assets-provision.ts`
  (edit-preserving; a maintenance cost ALSO posts the matching Repairs journal). Added
  `staff.view`/`staff.manage`/`staff.pay` + `asset.view`/`asset.manage` RBAC (deps +
  the `Staff & assets` group + Owner/Manager/Receptionist/Technician/Accountant grants)
  and the `asset.reminder` notification template. The `staff` Simple/Pro mode entry
  already existed.
- **Accounting port** (`apps/api/src/accounting`): `buildStaffPayPosting` already
  existed (spec-21 forward-ref, now wired LIVE); added `buildAssetMaintenancePosting`
  (DR Repairs / CR Bank) + `buildDepreciationPosting` (DR Depreciation / CR Accumulated
  depreciation), with `asset_maintenance`/`depreciation` posting kinds mapped to the
  `manual` journal source.
- **API** (`apps/api/src/staff/`): `StaffService` (EXPORTED) — overview (team + live
  clocked-in/attended-today/on-holiday state + counts + pending holidays), member
  detail, profile save, clock in/out (self-service), holiday request + decide, notes,
  productivity report, **pay → ledger** (the spec-21 seam). `AssetsService` (EXPORTED) —
  overview (register + reminders + book value), detail, register CRUD, checkout,
  maintenance (cost → ledger), depreciate (→ ledger), date reminders (module 11).
  `StaffController` (`/staff`) + `AssetsController` (`/assets`),
  `SessionGuard`+`PermissionGuard`: reads `staff.view`/`asset.view`, clocking + booking
  own leave `staff.view` (self), manage `staff.manage`/`asset.manage`, pay `staff.pay`.
  `StaffModule` wired into `app.module`.
- **Web** (`apps/web`): `/staff` (`useMode('staff')`) — Simple = who's in today, self
  clock in/out, book time off; Pro adds tabs (Team / Holidays / Productivity), full
  profiles + toggles, holiday approvals, technician notes, the efficiency table and pay
  recording. `/assets` (`useMode('staff')`) — Simple = register + due-date reminders +
  add equipment + tool checkout; Pro adds the maintenance log (cost → ledger),
  depreciation and the full register editor. i18n extended (`staff.*`/`assets.*`, no
  hardcoded strings); money via `formatCurrency`; dashboard "Team"/"Assets & tools"
  links gated by `staff.view`/`asset.view`. Contract: `packages/ui/STAFF_ASSETS_CONTRACT.md`.
- **Seed**: Org A seeds four staff profiles (login numbers + a non-default
  `includeInEfficiencyReport=false` for reception + pay rates for the techs), attendance
  (one open, one closed shift), a holiday request awaiting approval, a technician note,
  an equipment register (MOT kit with a near-due calibration + depreciation, a ramp, a
  torque wrench checked out to a tech) and a calibration maintenance entry whose £120
  cost posts a Repairs & maintenance expense to the ledger. Org B left empty
  (empty-state + isolation proof). Idempotent.

**Gates:** `lint` ✓ (0 warnings) · `typecheck` ✓ (db/types/api/web) · `build` ✓
(`/staff` 5.81 kB, `/assets` 5.24 kB) · `test` ✓ (db pure attendance/holiday/
productivity/pay-split/reminder/depreciation cases; api unit; DB-backed suites skip
cleanly with no DB). **Verified end-to-end against a real Postgres** (embedded, via
`scripts/verify-db.mjs`): all **24 migrations apply with zero drift** (`migrate diff` →
"No difference detected"); app role `rolsuper=false, rolbypassrls=false`; RLS **enabled
on all six new tables** (`staff_profile`/`attendance`/`holiday`/`technician_note`/
`asset`/`asset_maintenance_log`). The new `staff.integration.spec.ts` (5 cases) proves
profiles/attendance/holidays/notes/assets seed edit-preservingly, an **open attendance
measures seconds to now**, a **holiday's inclusive day count holds**, an **asset
maintenance cost posts a balanced Repairs & maintenance journal** (DR Repairs / CR Bank)
+ book-value maths, and all six tables are **RLS-isolated** (cross-tenant write rejected
by `WITH CHECK`). The **HTTP staff/assets e2e** (`staff-http.e2e.spec.ts`, 9 cases) boots
the full Nest app and proves: a **profile saves** (login number/pay rate/toggle); a
technician **clocks in/out** (double clock-in 400); a **holiday requests → approves**
(technician 403 on deciding); **staff pay posts a balanced staff-pay journal** (DR Wages
£2000 / CR Bank £1500 net + PAYE-NI £500); the **productivity report reads job clocking**
(2 h, 1 job done); an **asset registers with a due-soon calibration reminder** + monthly
depreciation £80; a **tool checks out** to a tech; **maintenance posts a balanced Repairs
journal** + **depreciation posts** DR Depreciation / CR Accumulated depreciation; a **date
reminder fires** over the module-11 hub; and the surface is **permission-gated**
(technician views but is 403 on managing) and **tenant-isolated** (Org B sees no assets,
404 on Org A's). Full db suite (**47 suites / 420 tests**, incl. staff integration) + api
suite (**32 suites / 193 tests**, incl. staff e2e) pass against the live DB — no
cross-module regressions. Seed idempotent (4 staff profiles / 3 assets Org A first run,
0 on re-run; Org B 0). Live screenshot pending deploy. **Ends at `[CHECKPOINT]`.**

### 24 — Courtesy / collection & delivery — APPROVED 2026-06-18 (operator instruction)
Approved by the human operator (soft checkpoint). See entry below for build detail.

### 24 — Courtesy / collection & delivery — DONE (pending approval), 2026-06-18
Two related logistics modules under one mode key (`delivery`) and one RBAC group. **A. Courtesy cars** —
the garage's own loan sub-fleet tracked like mini-assets: availability (derived: retired → out → off-road
window → available), MOT/road-tax/service **date reminders** (module 11), **fuel records** (each posts a
balanced **Consumables expense** to the ledger via the spec-21 seam), and out/in **assignment history**.
**B. Collection & delivery** — collecting a customer's car and/or delivering it back: driver, schedule,
**parking info recalled per vehicle** (the Excel-by-reg pain point), the **pre-call** confirmation, an
optional courtesy car **left with the customer** (loans that car out), camera-first collection + drop-off
**photos** (MinIO via OBJECT_STORAGE), and a scheduled → enroute → completed status flow that **reflects on
the module-14/15 job board flags**. A minimal **Driver** role + PWA list. Money in MINOR units (pence);
litres stored as exact millilitres (no floats).

- **Data model** (`packages/db`): four TENANT-SCOPED tables (RLS) — `CourtesyCar` (make/model/registration,
  `motDate`/`roadTaxDate`/`serviceDate`, `enabled`, `unavailableFrom`/`To`/`Reason`, `currentJobId`/
  `currentCustomerId`), `CourtesyCarFuelRecord` (`litresMl`/`amountMinor`), `CourtesyCarAssignment`
  (out/in at + mileage history) and `CollectionDelivery` (`type` collection|delivery|both, `driverUserId`,
  `scheduledAt`, `address`, `parkingInfo`, `preCallStatus`, `courtesyCarLeftId` FK, `collectionPhotoKeys`/
  `dropoffPhotoKeys` JSON, `status`). New enums `CollectionDeliveryType`/`PreCallStatus`/
  `CollectionDeliveryStatus`. Migration `20260618340000_courtesy_collection_delivery` (standard
  tenant-isolation RLS on all four + the courtesy-car-left FK).
- **Source of truth + pure maths** (`packages/db/src/courtesy.ts`): `courtesyAvailability` (the ONE
  availability rule), `isWithinUnavailableWindow`, `courtesyDueReminders`/`daysUntil`/`courtesyDateLabel`
  (the ONE overdue/due-soon reminder rule + lead time), `litresToMl`/`mlToLitres`, `assignmentMileage`,
  `canAdvanceCollection`/`nextCollectionStatus` (the status flow — forward-one-step + enroute→scheduled
  re-open, completed terminal), `collectionJobFlags` (type → board flags + courtesyCar), `normalisePhotoKeys`
  (clamp/dedupe), `COURTESY_AUDIT_ACTIONS`/`COLLECTION_AUDIT_ACTIONS`, `COURTESY_REMINDER_TEMPLATE_KEY`.
  `courtesy-provision.ts` (`seedCourtesyCar`/`seedCourtesyFuelRecord`[+ Consumables journal]/
  `seedCourtesyAssignment`/`seedCollectionDelivery` — edit-preserving). Added `delivery.view`/`delivery.manage`/
  `delivery.perform` RBAC (deps + the new **Driver** role; Owner/Manager/Receptionist grants), the `delivery`
  Simple/Pro mode entry and the `courtesy.reminder` notification template.
- **API** (`apps/api/src/courtesy/`): the accounting port gained `buildFuelExpensePosting` (DR Consumables /
  CR Bank; `fuel_expense` kind → `manual` source, idempotent on the fuel-record id). `CourtesyService`
  (EXPORTED) — overview/detail, fleet CRUD, availability, assign/return (with out/in mileage history), fuel
  (posts the expense through the ONE `ACCOUNTING_POSTING` seam), date reminders (module 11).
  `CollectionDeliveryService` (EXPORTED) — overview (driver-scoped `/mine`), schedule context (jobs/drivers/
  free loan cars + **last-known parking for the vehicle**), schedule/update, status flow, camera-first photos
  (OBJECT_STORAGE); scheduling **reflects job board flags** via the validated `JobsService` and **loans a
  left courtesy car out** via `CourtesyService`. `CourtesyController` (`/courtesy`) + `CollectionDeliveryController`
  (`/collection-delivery`), `SessionGuard`+`PermissionGuard`: reads `delivery.view`, fleet/schedule writes
  `delivery.manage`, carrying a run out (status/photo) `delivery.perform`. `CourtesyModule` wired into `app.module`.
- **Web** (`apps/web`): `/courtesy` (`useMode('delivery')`) — Simple = fleet + add/loan/fuel; Pro adds dates +
  reminders, off-road window and per-car history. `/collections` (`useMode('delivery')`) — Simple = today's
  runs, assign driver, mark done, snap photos; Pro adds scheduling, parking management, pre-call, courtesy-swap,
  full photo sets. `/driver` — the minimal, phone-first Driver list (en route/completed + camera photos). i18n
  extended (`courtesy.*`/`collections.*`, no hardcoded strings); money via `formatCurrency`; dashboard
  "Loan cars"/"Collection & delivery" gated by `delivery.view`, "My runs" by `delivery.perform`. Contract:
  `packages/ui/COURTESY_CONTRACT.md`.
- **Seed**: Org A seeds a 3-car fleet (one MOT due soon, one MOT overdue, one off-road for bodywork), a fuel
  fill on the first car (the Consumables expense posts), a closed loan in its history, and a
  collection-and-delivery run on an open job — driver assigned, parking info stored, pre-call done, a courtesy
  car left with the customer (an open loan). Org B left empty (empty-state + isolation proof). Idempotent.

**Gates:** `lint` ✓ (0 warnings) · `typecheck` ✓ (db/types/api/web) · `build` ✓ (`/courtesy` 5.21 kB,
`/collections` 5.07 kB, `/driver` 3.06 kB) · `test` ✓ (db pure courtesy availability/reminder/fuel/status/
flags/photo cases; api unit; DB-backed suites skip cleanly with no DB). **Verified end-to-end against a real
Postgres** (embedded, via `scripts/verify-db.mjs`): all **22 migrations apply with zero drift** (`migrate
diff` → "No difference detected"); app role `rolsuper=false, rolbypassrls=false`; RLS **enabled on all four
new tables** (`courtesy_car`/`courtesy_car_fuel_record`/`courtesy_car_assignment`/`collection_delivery`). The
new `courtesy.integration.spec.ts` (5 cases) proves cars/fuel/assignments/runs seed edit-preservingly, a
**fuel record posts a balanced Consumables expense journal** (DR Consumables / CR Bank), an **open assignment
marks the car out** (derived availability), and all four tables are **RLS-isolated** (cross-tenant write
rejected by `WITH CHECK`). The **HTTP courtesy e2e** (`courtesy-http.e2e.spec.ts`, 7 cases) boots the full
Nest app and proves: a fleet registers with derived availability + a **due-soon reminder**; logging fuel
**posts a balanced Consumables journal**; a car **loans out (→ out) and returns (→ available, 250 miles
logged)**; a **date reminder fires** over the notifications hub; a collection+delivery run **schedules,
reflects the job board flags** (collection/delivery/courtesyCar) and **loans the left car out**; the **Driver**
carries it out (en route → photo → completed); and the surface is **permission-gated** (technician 403; driver
can view + perform but is 403 on scheduling) and **tenant-isolated** (Org B sees no cars, 404 on Org A's).
Full db suite (**44 suites / 395 tests**, incl. courtesy integration) + api suite (**31 suites / 184 tests**,
incl. courtesy e2e) pass against the live DB — no cross-module regressions. Seed idempotent (3 cars / 1 run
Org A first run, 0 on re-run; Org B 0). Live screenshot pending deploy. **Ends at `[CHECKPOINT]`.**

### 23 — Plans & coverage — APPROVED 2026-06-18 (operator instruction)
Approved by the human operator (soft checkpoint). See entry below for build detail.

### 23 — Plans & coverage — DONE (pending approval), 2026-06-18
A primitive-based engine for the garage's OWN customer offerings (distinct from the module-06 SaaS
plans): garages snap **coverage primitives** (discount / inclusion / waiver / warranty / prepaid) into
**Packages** (one-off) or **Subscriptions/Memberships** (recurring), sell them to customers, and the
**coverage-resolution engine** automatically applies ALL of a customer's active coverage at invoice
time (module 22) in a **fixed, locked order** — and bills only the remainder. The six headline offerings
(service bundle, service plan, parts/repair warranty, prepaid wallet, membership, hours bank) are ALL
primitive compositions. Built on module-12 customers/tax codes, the module-14 job + module-19 parts
history (warranty proof), the module-21 accounting engine (deferred revenue) and the module-05 mode
framework. Money in MINOR units (pence) end-to-end.

- **Data model** (`packages/db`): four TENANT-SCOPED tables (RLS) — `ServiceItem` (the garage's
  labour/service price list), `CoverageProduct` (the designed offering — `kind` package|subscription,
  `recognition` immediate|deferred, `primitives` JSON, `termMonths`/`billingCycle`), `CustomerCoverage`
  (an active holding — `status`, `balanceMinor` wallet/hours, `meta` warranty fitment facts,
  `deferredScheduleId`; a customer may hold MANY) and `CoverageUsage` (the ledger of what coverage
  applied to which invoice). New enums `CoverageKind`/`CoverageRecognition`/`CoverageStatus`. Migration
  `20260618330000_plans_coverage` (standard tenant-isolation RLS on all four).
- **Source of truth + pure maths** (`packages/db/src/coverage.ts`): `normalisePrimitives`,
  `OFFERING_TEMPLATES` (the six headline offerings), **`resolveCoverage`** (the LOCKED-order engine:
  ① waivers/warranties/inclusions → covered lines £0, ② %/£ discounts on the remaining non-zero lines,
  ③ prepaid/wallet drawn against the remaining total last, ④ floor at £0 — multiple holdings stack;
  returns the reduced lines + a breakdown + coveredMinor), **`isWarrantyEligible`/`warrantyExpiry`** (the
  per-part-instance, fitment-dated, garage-proven rule), `summariseHolding` (the plan-usage ledger),
  `perkLabel`, `COVERAGE_AUDIT_ACTIONS`. `coverage-provision.ts` (`seedServiceItem`/`seedCoverageProduct`/
  `seedCustomerCoverage` — edit-preserving; a deferred sale ALSO posts the matching plan-sale journal +
  a deferred-revenue schedule). Added `coverage.view`/`coverage.edit`/`coverage.sell` RBAC (deps +
  Owner/Manager/Receptionist/Accountant grants), the `Plans & coverage` permission group + the
  `coverage` Simple/Pro mode-registry entry.
- **API** (`apps/api/src/coverage/`): the spec-22 `COVERAGE_RESOLUTION` seam is **REBOUND** from the
  Phase-1 no-op `NoCoverageResolution` to the real `CoverageResolutionService` (provided + exported by
  `CoverageModule`, which the invoicing module imports) — `resolve` plans the reduction (pure, no
  mutation), the additive `recordUsage(orgId, invoiceId, applications)` hook (the ONLY change to the
  invoicing caller) writes CoverageUsage + draws the wallet down after the invoice exists. `CoverageService`
  (EXPORTED) + `CoverageController` (`/coverage`): reads (`coverage.view`) = overview / sell-context /
  customer plan-usage ledger; offerings + service items (`coverage.edit`); sell + cancel (`coverage.sell`).
  A deferred sale recognises revenue through the module-21 `AccountingService.createDeferral` (deferred
  income + a recognition schedule); an immediate package posts DR Bank / CR Sales–Other (+VAT) through the
  `ACCOUNTING_POSTING` seam. A warranty is sold ONLY against a part proven fitted by this garage (a
  `used_on_job` movement on the job — module 19) and the fitment facts are captured in `meta`.
- **Web** (`apps/web/app/coverage`): `/coverage` — `useMode('coverage')`. Simple = pick an offering
  template → create it, sell it to a customer, see active coverage; Pro adds the full primitive composer
  (Compose tab), the customer-coverage tab and the per-holding usage ledger. i18n extended (`coverage.*`,
  no hardcoded strings); money via `formatCurrency`; dashboard "Plans & coverage" link gated by
  `coverage.view`. Contract: `packages/ui/COVERAGE_CONTRACT.md`.
- **Seed**: Org A builds a membership (10% off labour + free MOT, deferred), a £200→£220 prepaid wallet
  and a 12-month brake-pad warranty, and sells the customer the membership (deferred income posts), the
  wallet (£220 credit) and an in-term warranty against a fitted part — so a later invoice stacks them.
  Org B left empty (empty-state + isolation proof). Idempotent.

**Gates:** `lint` ✓ (0 warnings) · `typecheck` ✓ (db/types/api/web) · `build` ✓ (`/coverage` 5.72 kB) ·
`test` ✓ (db 15 new pure primitive/template/warranty/locked-order/usage-ledger cases; api unit;
DB-backed suites skip cleanly with no DB). **Verified end-to-end against a real Postgres** (embedded, via
`scripts/verify-db.mjs`): all **22 migrations apply with zero drift** (`migrate diff` → "No difference
detected"); app role `rolsuper=false, rolbypassrls=false`; RLS **enabled on all four new tables**
(`service_item`/`coverage_product`/`customer_coverage`/`coverage_usage`). The new
`coverage.integration.spec.ts` (4 cases) proves service items/products/holdings seed edit-preservingly, a
**deferred sale posts a balanced plan-sale journal (CR deferred income £100) + a recognition schedule**,
the engine **stacks a membership discount + a wallet drawdown in the locked order, never negative**, and
all four tables are **RLS-isolated** (cross-tenant write rejected by `WITH CHECK`). The **HTTP coverage
e2e** (`coverage-http.e2e.spec.ts`, 6 cases) boots the full Nest app and proves: offerings compose; a
**deferred membership sale posts a deferred-revenue schedule + a balanced plan-sale journal** (CR deferred
income); a **wallet** sells with £220 credit; a **warranty is honoured ONLY against a part this garage
fitted** (rejected 400 otherwise, accepted 201); at **INVOICE time the resolver STACKS** warranty (part
£0) + 10% labour + wallet drawdown in the locked order → the £200 invoice bills £0, `coveredMinor` £200,
**three CoverageUsage rows** recorded and the **wallet drew down £108**; the **plan-usage ledger** shows
covered/left; and the surface is **permission-gated** (Technician 403; Receptionist views + sells but 403
on composing) and **tenant-isolated** (Org B sees no offerings). Full db suite (**42 suites / 376 tests**,
incl. coverage integration) + api suite (incl. coverage e2e) pass against the live DB — no cross-module
regressions; the module-22 invoicing e2e stays green with the seam rebound. Seed idempotent. Live
screenshot pending deploy. **Ends at `[CHECKPOINT]`.**

### 22 — Invoicing & payments — APPROVED 2026-06-18 (operator instruction)
Approved by the human operator (manual `[FIXED_CHECKPOINT]`). See entry below for build detail.

### 22 — Invoicing & payments — DONE (pending approval), 2026-06-18
ONE invoice→ledger pipeline under the whole app: a customer invoice is generated from a job, an
authorised estimate, a parts-only sale (module 19) or ad-hoc; payment is taken via **five online
gateways** (the garage's OWN connected account) **or four manual methods**; and EVERY money event —
invoice, payment, refund, void — posts a balanced double-entry journal through the SAME
`accounting.post(...)` seam module 20/21 use. Whether paid by Stripe online or cash at the desk the
invoice updates the same way and posts the same entries — no separate money paths. Built on module-12
customers (on-stop, per-customer tax code, customer-reference rule), module-18 estimates (the ONE
VAT-totals path), module-19 parts (suggested job lines), module-11 notifications (the branded invoice
link) and the module-05 mode framework. Money in MINOR units (pence) end-to-end. This is the third and
final Phase-1 **hard stop** — `[FIXED_CHECKPOINT]`.

- **Platform safety (payments):** **no gateway secret keys EVER cross the API, the repo or the DB.**
  `PaymentGatewayConfig.config` stores only NON-SECRET reference data (account id, publishable key,
  mode) — `sanitiseGatewayConfig` whitelists those; `secretRef` is a pointer to a server-side
  encrypted secret, never the secret. The build + staging only ever use the `SandboxPaymentGateway`
  binding of the `PAYMENT_GATEWAY` seam — it NEVER touches real credentials and NEVER moves real funds
  (it mints a synthetic `sandbox_…` reference). A real-charge binding is intentionally not implemented,
  so the agent can never transact. `.env.example` untouched (no new secrets).
- **Data model** (`packages/db`): four TENANT-SCOPED tables (RLS) — `Invoice` (per-org `invoiceNo`
  sequence; `lineItems` JSON [labour/part/sublet/other]; UK-VAT totals; `coveredMinor`;
  `amountPaidMinor`; `status` `outstanding|partial|paid|void`; `customerReference`; `splitBilling`
  JSON; loose within-org `branchId`/`customerId`/`vehicleId`/`jobId`/`estimateId`/`partsOnlySaleId`),
  `Payment` (`method` 9-way, `gatewayRef`, `feeMinor`, `status`), `PaymentGatewayConfig`
  (per-(org,gateway); non-secret `config` + `secretRef` pointer), `Refund`. New enums `InvoiceSource`/
  `InvoiceStatus`/`PaymentMethod`/`PaymentStatus`/`PaymentGateway`/`GatewayConnectionStatus`. Migration
  `20260618320000_invoicing_payments` (standard tenant-isolation RLS on all four).
- **Source of truth + pure maths** (`packages/db/src/invoicing.ts`): catalogues (`INVOICE_SOURCES`/
  `INVOICE_STATUSES`/`PAYMENT_METHODS` [5 online + 4 manual]/`PAYMENT_GATEWAYS`/
  `GATEWAY_CONNECTION_STATUSES`), `normaliseInvoiceLines` (per-customer tax override),
  **`computeInvoiceTotals`** (REUSES module-18 `computeEstimateTotals` — the ONE VAT path),
  **`salesAccountForLineType`/`invoiceSalesSplits`** + **`moneyAccountForMethod`** (the ONE nominal
  mapping), **`invoiceStatusFor`** (the ONE outstanding/partial/paid/void rule), **`normaliseSplitBilling`**
  (insurance-excess split always sums to the total), **`buildArAging`** (debtors), `INVOICE_TEMPLATE_KEY`,
  `INVOICING_AUDIT_ACTIONS`. `invoicing-provision.ts` (`seedInvoice`/`seedPayment`/`seedRefund`/
  `seedGatewayConfig` — edit-preserving; each money event ALSO posts the matching balanced journal via
  `seedJournalEntry`, so the seeded books match the documents). Added the `invoice.sent` notification
  template (module 11), `payment.refund` + `gateway.manage` RBAC permissions (deps + Owner/Manager/
  Accountant grants); `invoice.view`/`invoice.edit`/`invoice.send`/`payment.take` + the `invoicing`
  Simple/Pro mode entry already existed.
- **API** (`apps/api/src/invoicing/`): two SEAMS — `COVERAGE_RESOLUTION` (spec-23 forward-reference;
  the service runs the resolver BEFORE finalising totals; Phase-1 `NoCoverageResolution` covers nothing,
  rebound by spec 23 with no caller change) and `PAYMENT_GATEWAY` → `SandboxPaymentGateway`. Extended
  the accounting port with `buildCustomerRefundPosting` + a `feeMinor` arg on `buildCustomerPaymentPosting`
  (gateway fee → card fees). `InvoicingService` (EXPORTED) + `InvoicingController` (`/invoicing`): reads
  (`invoice.view`) = overview (outstanding + takings + AR aging + gateways) / context (suggested lines +
  customer + on-stop) / invoice / gateways; writes = generate+amend+void (`invoice.edit`), send
  (`invoice.send`), payments (`payment.take`), refunds (`payment.refund`), gateways
  (`gateway.manage`). On-stop customers are **hard-blocked** from new credit work at invoice time
  (Owner/Manager `overrideOnStop`); a require-reference customer is enforced. A `PublicInvoicingController`
  (`/invoicing/public/:id`, UNAUTHENTICATED) serves the branded read-only invoice via `ADMIN_DB`.
- **Web** (`apps/web`): `/invoicing` — `useMode('invoicing')`. Simple = take-a-payment + the
  outstanding list + headline stats; Pro adds tabs (Outstanding, Invoices, Debtors AR-aging, Payment
  gateways), the full invoice generator (line/tax detail, insurance-excess split), invoice detail (take
  payment, refund, send, copy-link, void). `?jobId`/`?estimateId`/`?partsOnlySaleId` deep-links open the
  generator pre-filled. `/i/[id]` — the PUBLIC, no-login, **branded** invoice (review + print/PDF). i18n
  extended (`invoicing.*`, no hardcoded strings); money via `formatCurrency`; dashboard "Invoicing" link
  gated by `invoice.view`. Contract: `packages/ui/INVOICING_CONTRACT.md`.
- **Seed**: Org A generates three customer invoices through the pipeline — a fully-paid one (cash at the
  desk), a part-paid one settled by a **Stripe SANDBOX/test charge** (no real funds, no real credentials)
  with a goodwill refund, and an **insurance-excess split** invoice left outstanding (90+ days, for the
  aging report) — and connects a Stripe gateway in **test mode** (non-secret reference only). Each money
  event posts to the ledger so the books stay reconciled. Org B left empty (empty-state + isolation
  proof). Idempotent.
- **Scope note (disclosed):** "send + branded invoice + print-to-PDF" is delivered as a branded no-login
  HTML page (the established pattern for customer docs — estimates/EVHC/day-sheet), NOT a server-side PDF
  library. Customer **self-serve online payment** (entering card on the public page) is a deliberate
  Phase-2 add — the live gateway redirect needs the garage's connected account + server-side secret; the
  staff surface takes online payments end-to-end today (sandbox), and the public page is review/print.

**Gates:** `lint` ✓ (0 warnings) · `typecheck` ✓ (db/types/api/web) · `build` ✓ (`/invoicing` 7.62 kB,
`/i/[id]` server-rendered) · `test` ✓ (db 13 new pure method/line/VAT/nominal-mapping/status/split/aging
cases; api unit; DB-backed suites skip cleanly with no DB). **Verified end-to-end against a real Postgres**
(embedded, via `scripts/verify-db.mjs`): all **21 migrations apply with zero drift** (`migrate diff` →
"No difference detected"); app role `rolsuper=false, rolbypassrls=false`; RLS **enabled on all four new
tables** (`invoice`/`payment`/`payment_gateway_config`/`refund`). The new `invoicing.integration.spec.ts`
(5 cases) proves an invoice seeds with exact UK-VAT totals + a **balanced** customer-invoice journal, a
payment + a refund re-derive the amount-paid + status through the ONE pipeline (both post to the ledger),
an insurance-excess split **sums to the total**, the trade-debtors balance reflects the unpaid amount, and
invoices/payments/refunds/gateway configs are **RLS-isolated** (cross-tenant write rejected by `WITH CHECK`).
The **HTTP invoicing e2e** (`invoicing-http.e2e.spec.ts`, 8 cases) boots the full Nest app and proves: an
invoice **generates with UK VAT** (£201.60) + posts a balanced journal; a **MANUAL cash** payment and an
**ONLINE Stripe SANDBOX** payment BOTH settle through the same pipeline + post to the ledger (the online
one is **blocked until the gateway is connected**, then clears through `card_clearing`, gatewayRef
`sandbox_stripe_…`); a partial payment → `partial`, a **refund** reverses on the ledger + re-derives status
(over-refund 400); an **on-stop** customer is **403** (override 201); a **require-reference** customer is
**400** without a reference (201 with); an **insurance-excess split** sums to the total; a **void** reverses
an unpaid invoice (voiding a paid one 400); the **PUBLIC no-login invoice** serves branded (unknown id 404);
and the surface is **permission-gated** (Technician 403 on viewing; Receptionist takes payments but is 403
on generate/refund/gateways) and **tenant-isolated** (Org B sees no invoices, 404 on Org A's). Full db
suite (**40 suites / 357 tests**, incl. invoicing integration) + api suite (**29 suites / 171 tests**, incl.
invoicing e2e) pass against the live DB — no cross-module regressions. Seed idempotent (3 invoices / 2
payments / 1 refund / 1 gateway Org A first run, 0 on re-run; Org B 0). Live screenshot pending deploy.
**Ends at `[FIXED_CHECKPOINT]` — HARD stop, manual approval required.**

### 21 — Accounting engine — APPROVED 2026-06-18 (operator instruction)
Approved by the human operator (soft checkpoint). See entry below for build detail.

### 21 — Accounting engine — DONE (pending approval), 2026-06-18
One ALWAYS-double-entry ledger under the whole app, with two faces over the SAME data: **Simple
"Money In/Out"** and **Full accounting** (chart of accounts, journals, trial balance, P&L, balance
sheet, VAT return, deferred revenue) — presentation only (module 05), lossless, switchable anytime.
Every money action auto-posts a balanced journal through the ONE posting seam. Built on module-20's
accounting POSTING SEAM (the forward-reference, now **rebound to the real engine with ZERO caller
change**), the module-12 tax codes, the module-10 audit trail and the module-05 mode framework. Money
in MINOR units (pence) end-to-end.

- **Data model** (`packages/db`): five TENANT-SCOPED tables (RLS) — `Account` (chart of accounts;
  system codes match the stable nominal keys the seam references — `bank`/`accounts_payable`/
  `parts_cogs`/`vat_input`/…), `JournalEntry` (balanced; `source`/`sourceRef` trace it to the
  document; **unique (org, source, sourceRef) ⇒ idempotent posting**; `reversedById` links a reversal
  — never a delete), `JournalLine` (one side; optional `taxCode`), `VatCode` (standard UK T-codes) and
  `DeferredRevenueSchedule` (straight-line monthly recognition). New enums `AccountType`/
  `JournalSource`/`DeferredScheduleStatus`. Migration `20260618310000_accounting_engine` (standard
  tenant-isolation RLS on all five).
- **Source of truth + pure maths** (`packages/db/src/accounting.ts`): `NOMINAL` (stable codes incl.
  the four module-20 keys verbatim), `DEFAULT_CHART_OF_ACCOUNTS` (the default UK garage chart),
  `DEFAULT_VAT_CODES`, `normalSide`/`accountBalanceMinor` (the ONE sign convention),
  **`isPostingBalanced`** (Σdebits = Σcredits), **`buildTrialBalance`/`buildProfitAndLoss`/
  `buildBalanceSheet`/`buildVatReturn`/`buildMoneyInOut`** (the reports — each over plain aggregated
  rows, unit-testable without a DB; the balance sheet balances because un-closed earnings surface
  inside equity), the straight-line deferred maths (`recognitionAmountMinor`/`dueRecognitions`/
  `advanceSchedule`/`addMonths`), `ACCOUNTING_AUDIT_ACTIONS`. `accounting-provision.ts`
  (`ensureChartOfAccounts`/`ensureVatCodes` — idempotent + the lazy-provision path; `seedJournalEntry`
  refuses an unbalanced entry; `seedDeferredSchedule`). Added `invoicing.vatRegistered` setting
  (module-10 catalogue). `accounting.view`/`accounting.manage` RBAC + the `accounting` Pro-only mode
  entry already existed.
- **The engine + seam** (`apps/api/src/accounting/`): **rebound** `ACCOUNTING_POSTING` from the
  Phase-1 audit-only `StagedAccountingPosting` (deleted) to the REAL `LedgerAccountingPosting` — it
  resolves each posting line's nominal `account` code to the org's Account row (lazily provisioning the
  chart on first use), writes a balanced `JournalEntry` + `JournalLine`s under `withTenant` (RLS),
  maps `kind → JournalSource`, and is **idempotent on (source, sourceRef)**. Module-20 callers UNCHANGED
  (they already hand over fully-formed postings). Extended the port with builders for spec 22/07/23/25
  (`buildCustomerInvoicePosting`/`buildCustomerPaymentPosting`/`buildCreditSalePosting`/
  `buildStaffPayPosting`/`buildPlanSalePosting`/`buildDeferralReleasePosting`). Wired the **module-07
  credit top-up LIVE** (`MeteringService.purchase` posts a VATable `credit_sale`). `AccountingService`
  (EXPORTED) + `AccountingController` (`/accounting`): reads (`accounting.view`) = overview/reference/
  accounts/journal/trial-balance/profit-and-loss/balance-sheet/vat-return/deferred; writes
  (`accounting.manage`) = provision/account CRUD/manual journal/reverse/deferral create+recognise/
  vat-registration.
- **Web** (`apps/web`): `/accounting` — `useMode('accounting')`. Simple = the "Money In/Out" stat
  dashboard; Pro adds tabs (chart of accounts, journals + reverse, trial balance, P&L, balance sheet,
  VAT return [hidden when not VAT-registered], deferred revenue + run-recognition), with a "Set up the
  books" CTA that provisions on first use. i18n extended (`accounting.*`, no hardcoded strings); money
  via `formatCurrency`; dashboard "Accounting" link gated by `accounting.view`. Contract:
  `packages/ui/ACCOUNTING_CONTRACT.md`.
- **Seed**: Org A provisions the default chart + VAT codes and seeds six BALANCED proof journals (a
  customer invoice + its payment, a purchase + a part-payment, a VATable credit top-up, a deferred plan
  sale) + a 12-month deferred-revenue schedule — so the trial balance balances, P&L/balance sheet/VAT
  return render and Money In/Out shows real figures out of the box. Org B left empty (empty-state +
  isolation proof). Idempotent.

**Gates:** `lint` ✓ (0 warnings) · `typecheck` ✓ (db/types/api/web) · `build` ✓ (`/accounting`
5.08 kB) · `test` ✓ (db 18 new pure chart/sign/balanced/trial-balance/P&L/balance-sheet/VAT/money-in-out/
deferred cases; api unit; DB-backed suites skip cleanly with no DB). **Verified end-to-end against a
real Postgres** (embedded, via `scripts/verify-db.mjs`): all **20 migrations apply with zero drift**
(`migrate diff` → "No difference detected"); app role `rolsuper=false, rolbypassrls=false`; RLS
**enabled on all five new tables** (`account`/`journal_entry`/`journal_line`/`vat_code`/
`deferred_revenue_schedule`). The new `accounting.integration.spec.ts` (5 cases) proves the chart +
VAT codes provision edit-preservingly, a **balanced journal seeds (its lines resolve to accounts) and
an unbalanced one is refused**, a re-seed of the same (org, source, sourceRef) is a no-op (idempotent),
the **trial balance balances** + the **balance sheet reconciles** over the seeded ledger, and accounts/
journals/lines/schedules are **RLS-isolated** (Org A invisible to Org B; cross-tenant write rejected by
`WITH CHECK`). The **HTTP accounting e2e** (`accounting-http.e2e.spec.ts`, 6 cases) boots the full Nest
app and proves: the chart **provisions (idempotent)**; a **balanced manual journal posts**, an
unbalanced one is **400**, and a posted entry **reverses** (a second reverse is 400); a **real credit
top-up AUTO-POSTS a VATable `credit_sale` journal** (the seam wired live — VAT £20 on £100); the
**trial balance, P&L, balance sheet (balances) and VAT return render**; a **deferred schedule recognises
monthly** (≥2 periods released as `deferral` journals, the schedule advances £10/month); and the surface
is **permission-gated** (Manager reads but is 403 on posting/provision via the view/manage split;
Technician 403 on viewing) and **tenant-isolated** (Org B un-provisioned, sees no journals). The
**module-20 suppliers e2e was updated** to assert the now-REAL purchase/payment journal entries (the
rebinding is observable) — it stays green. Full db suite (**38 suites / 339 tests**, incl. accounting
integration) + api suite (incl. accounting e2e) pass against the live DB — no cross-module regressions.
Seed idempotent (chart + 6 journals + 1 deferred schedule Org A first run, 0 on re-run; Org B 0). Live
screenshot pending deploy. **Ends at `[CHECKPOINT]`.**

### 20 — Suppliers & purchasing — APPROVED 2026-06-18 (operator instruction)
Approved by the human operator (soft checkpoint). See entry below for build detail.

### 20 — Suppliers & purchasing — DONE (pending approval), 2026-06-18
Where parts come from and what the garage owes: suppliers (distributors), purchase invoices with
per-line parts (which increment stock via the module-19 ledger and **stage the accounts-payable
posting behind the spec-21 accounting seam** — no parallel ledger invented), per-supplier
balances/statements, payments and returns/credits. Built on the module-19 parts/stock backbone,
the module-12 customers/tax codes, the module-14 Job pipeline (cost attribution) and the module-10
audit trail. Money in MINOR units (pence) end-to-end.

- **Data model** (`packages/db`): five TENANT-SCOPED tables (RLS) — `Supplier` (searchable
  company/contact/address/phone/email/`accountingCode`, `enabled`), `SupplierDepot` (Pro multi-depot),
  `PurchaseInvoice` (supplier's `invoiceNo` unique per `(org, supplier)`, `lines` JSON, UK-VAT
  `netTotalMinor`/`vatTotalMinor`/`invoiceTotalMinor`, `status` `unpaid|partial|paid`, plain nullable
  `branchId`/`jobId`/`customerId`), `SupplierPayment` (`amountMinor`, `method`, optional
  `purchaseInvoiceId` else on-account) and `SupplierReturn` (returned `lines`, `creditTotalMinor`).
  New enums `PurchaseInvoiceStatus`/`SupplierPaymentMethod`. **Extended** the module-19
  `StockMovementReason` enum with `supplier_return` (goods back to a supplier LEAVE stock —
  outbound, distinct from the inbound `return`). Migration `20260618300000_suppliers_purchasing`
  (standard tenant-isolation RLS on all five + the enum `ADD VALUE`).
- **Source of truth + pure maths** (`packages/db/src/suppliers.ts`): `SUPPLIER_PAYMENT_METHODS` +
  `PURCHASE_INVOICE_STATUSES` catalogues, `normalisePurchaseLines` (drop empty, clamp, validate
  tax code, price each line's net), **`computePurchaseTotals`** (the UK-VAT totals — REUSES the
  module-18 `computeEstimateTotals`, the ONE money path, over each line's net cost),
  **`supplierBalanceMinor`** (the ONE payables sum: invoices − payments − credits),
  **`purchaseInvoiceStatusFor`** (the ONE unpaid/partial/paid rule), `buildSupplierSearchTerms`/
  `supplierMatchesQuery`, `SUPPLIER_AUDIT_ACTIONS`. `suppliers-provision.ts`
  (`seedSupplier`/`seedPurchaseInvoice`/`seedSupplierPayment`/`seedSupplierReturn`, edit-preserving).
  Added `supplier.view`/`supplier.edit`/`purchase.manage` permissions (RBAC catalogue + deps +
  Owner/Manager/Accountant grants) and the `suppliers` mode-registry entry (Simple/Pro).
- **Accounting seam** (`apps/api/src/accounting/`): the spec-21 forward-reference. Every money event
  is a balanced typed `LedgerPosting` (DR/CR lines on stable nominal accounts — see
  `accounting-posting.port.ts` with `buildPurchaseInvoicePosting`/`buildSupplierPaymentPosting`/
  `buildSupplierReturnPosting`) handed to the `ACCOUNTING_POSTING` port. Phase-1 binding
  `StagedAccountingPosting` records the posting on the module-10 audit trail (`accounting.posting.staged`
  — a record, NOT a parallel ledger). Spec 21 rebinds the token to the real engine with NO caller change.
- **API** (`apps/api/src/suppliers/`): `SuppliersService` — `overview` (supplier list + search +
  payables summary), `getSupplier` (full statement: invoices/payments/returns + balance),
  supplier + depot CRUD, `purchaseContext` (record-a-bill data + each product's last-purchase price),
  **`recordPurchaseInvoice`** (normalise → totals → unique `(supplier, invoiceNo)` → stock-in via the
  ONE module-19 `PartsService.adjustStock` `purchase` movement + update product cost → stage the
  payables posting → audit), `recordPayment` (allocate + re-derive invoice status + post), `recordReturn`
  (`supplier_return` stock-out + credit + re-derive status + post the reversal). `SuppliersController`
  (`SessionGuard`+`PermissionGuard`): reads `supplier.view`, supplier/depot edits `supplier.edit`,
  purchases/payments/returns `purchase.manage`. `SuppliersService` EXPORTED for later modules.
- **Web** (`apps/web`): `/suppliers` — staff surface (`useMode('suppliers')`, Simple/Pro): Simple =
  add a supplier + record a bill + the payables summary ("Total owed to suppliers £X") + supplier list;
  Pro adds the supplier statement, payment allocation, returns/credits, depots, accounting codes and
  last-purchase price history. i18n extended (`suppliers.*`, no hardcoded strings); money via
  `formatCurrency`; dashboard "Suppliers" link gated by `supplier.view`; a shared `.input` form-control
  class added to `globals.css`. Contract: `packages/ui/SUPPLIERS_CONTRACT.md`.
- **Seed**: Org A seeds two distributors (GSF, Euro Car Parts), a purchase invoice that restocks the
  brake pads + discs (goods-in + a £264 payable), a £200 part payment (the bill shows `partial`) and a
  1-disc return/credit (the `supplier_return` stock-out + a credit against the balance). Org B left
  empty (empty-state + isolation proof). Idempotent.

**Gates:** `lint` ✓ (0 warnings) · `typecheck` ✓ (db/types/api/web) · `build` ✓ (`/suppliers`
7.33 kB) · `test` ✓ (db 8 new pure cases; api unit; DB-backed suites skip cleanly with no DB).
**Verified end-to-end against a real Postgres** (embedded, via `scripts/verify-db.mjs`): all **19
migrations apply with zero drift** (`migrate diff` → "No difference detected"); app role
`rolsuper=false, rolbypassrls=false`; RLS **enabled on all five new tables**. The new
`suppliers.integration.spec.ts` (4 cases) proves suppliers + a purchase invoice seed
edit-preservingly (invoiceNo unique per (org, supplier)), recording a purchase **increments stock**
(a `purchase` movement — the ledger reconciles), a part payment flips the invoice to `partial` and the
**balance reconciles** (invoices − payments − credits), a return writes a **`supplier_return` movement
(stock LEAVES)** + credits the balance, and all are **RLS-isolated** (Org A invisible to Org B;
cross-tenant write rejected by `WITH CHECK`). The **HTTP suppliers e2e** (`suppliers-http.e2e.spec.ts`,
5 cases) boots the full Nest app and proves: a **searchable supplier** is created; **recording a
purchase invoice** computes exact UK-VAT totals (£264), **increments stock** (`purchase` movement,
product cost updated), **stages the payables posting** (audited `accounting.posting.staged`) and shows
the £264 balance, with a **duplicate invoice number rejected**; a **payment** flips the invoice to
`partial` + reconciles the balance (£64) + stages its posting; a **return** writes a `supplier_return`
movement (stock −2) + a £52.80 credit (balance £11.20); and the surface is **permission-gated**
(technician 403 on the list; receptionist 403 on recording a bill) and **tenant-isolated** (Org B sees
no suppliers, 404 on Org A's). Full db suite (**36 suites / 316 tests**, incl. suppliers integration)
+ api suite (**27 suites / 157 tests**, incl. suppliers e2e) pass against the live DB — no cross-module
regressions. Seed idempotent (2 suppliers / 1 purchase invoice Org A first run, 0 on re-run; Org B 0).
Live screenshot pending deploy. **Ends at `[CHECKPOINT]`.**

### 19 — Parts & inventory — APPROVED 2026-06-18 (operator instruction)
Approved by the human operator (soft checkpoint). See entry below for build detail.

### 19 — Parts & inventory — DONE (pending approval), 2026-06-18
The parts/stock backbone: a two-level product catalogue, per-(product, branch) stock levels
maintained alongside an **append-only movement ledger**, inter-branch transfers, the
**technician → admin → arrived parts-request loop** (which drives the module-15 board) and a
walk-in **parts-only counter sale**. Built on the module-14 Job pipeline, the module-15 realtime
board, the module-11 notifications hub and the module-12 customers/tax codes. Money in MINOR
units (pence) end-to-end.

- **Data model** (`packages/db`): seven TENANT-SCOPED tables (RLS) — `ProductCategory`
  (two-level tree), `Product` (per-org `productNo`, cost/sale/trade pence, `taxCode`, `ean`,
  `crossReference`, `addToEveryJob`, nullable `supplierId` for module 20), `StockLocation`,
  `StockLevel` (`qtyOnHand`/min/max per (product, branch); low = `qtyOnHand < minLevel`),
  `StockMovement` (APPEND-ONLY signed `delta` + `reason` + polymorphic `reference`),
  `StockTransfer` (Pro: `requested→in_transit→received`, `goodsReceivedBy`) and `PartsOnlySale`
  (per-org `saleNo`, lines JSON, UK-VAT totals, nullable `invoiceId` for module 22). New enums
  `StockMovementReason`/`StockTransferStatus`. **Extended** the module-15 `JobPartRequest` with
  `productId`/`partNumber`/`orderedBy`/`orderedAt`/`arrivedAt` (the order→arrive loop). Migration
  `20260618290000_parts_inventory` (standard tenant-isolation RLS on all seven + the ALTER).
- **Source of truth + pure maths** (`packages/db/src/parts.ts`): `STOCK_MOVEMENT_REASONS`
  (+ each reason's direction), **`movementDelta`** (the ONE place a movement's signed delta is
  derived — inbound adds / outbound subtracts / `adjustment` keeps the sign), **`isLowStock`**
  (the ONE reorder rule), `applyDelta` (never below zero), `STOCK_TRANSFER_STATUSES` +
  `normaliseTransferLines`, `effectivePriceMinor`, `buildProductSearchTerms`/`productMatchesQuery`,
  `PARTS_AUDIT_ACTIONS`, `PARTS_ARRIVED_TEMPLATE_KEY`. Sale lines REUSE the module-18
  `normaliseLineItems` + `computeEstimateTotals` (ONE money path). `parts-provision.ts`
  (`seedProductCategory`/`seedProduct`/`seedStockLocation`/`seedStockLevel`, edit-preserving).
  Added the `parts.sell` permission (RBAC catalogue + dep + Owner/Manager/Receptionist grants —
  the front-desk counter sale) and the `parts.arrived` (in-app) notification template (module 11).
  The `inventory` mode-registry entry + the `alerts.lowStock` setting already existed.
- **API** (`apps/api/src/parts/`): `PartsService` — `overview`/`productDetail`, product +
  category + location CRUD, **`adjustStock`** (the ONE stock-change path: derive delta → upsert
  level → write the append-only ledger row → audit → low-stock alert on crossing), `setReorder`,
  `useOnJob`, transfers (`create`/`dispatch`[transfer_out]/`receive`[transfer_in + goods-received]),
  the parts-only counter `createSale` (exact UK-VAT totals, decrements stock via `sale` movements,
  `invoiceId` stub) and `defaultJobLines` (the add-to-every-job helper for jobs/estimates).
  `PartsRequestService` — the loop: `queue`/`order` (records part number, advances the job to
  `awaiting_parts` via the ONE validated `JobsService.transition` so the board recolours)/`arrive`
  (advances to `parts_arrived`, notifies the technician over the hub, surfaces the part number)/
  `cancel`; every step fans the module-15 realtime board event. `PartsController`
  (`SessionGuard`+`PermissionGuard`): reads `parts.view`, catalogue/stock/transfers/loop
  `parts.edit`, counter sale `parts.sell`. Both services EXPORTED for later modules.
- **Web** (`apps/web`): `/parts` — staff surface (`useMode('inventory')`, Simple/Pro): Simple =
  search a part + in-stock/low + quick counter sale; Pro adds the parts desk (order→arrive), the
  inter-branch transfers tab, the full catalogue editor with per-branch stock + ledger + min/max
  reorder editor, and the low-stock report. i18n extended (`parts.*`, no hardcoded strings); money
  via `formatCurrency`; dashboard "Parts" link gated by `parts.view`. Contract:
  `packages/ui/PARTS_CONTRACT.md`.
- **Seed**: Org A seeds two 2-level categories, four products (incl. an `addToEveryJob` sundry and
  a deliberately LOW-stock wiper) and opening stock across **two branches** (a second "Apex Retail"
  branch is upserted for the multi-branch + transfer proof). Org B left empty (empty-state +
  isolation proof). Idempotent.

**Gates:** `lint` ✓ (0 warnings) · `typecheck` ✓ (db/types/api/web) · `build` ✓ (`/parts`
8.13 kB) · `test` ✓ (db 211 unit incl. the 12 pure delta/low-stock/transfer/pricing/search cases;
api 46 unit; DB-backed suites skip cleanly with no DB). **Verified end-to-end against a real
Postgres** (embedded, via `scripts/verify-db.mjs`): all **18 migrations apply with zero drift**
(`migrate diff` → "No difference detected"); app role `rolsuper=false, rolbypassrls=false`; RLS
**enabled on all seven new tables**. The new `parts.integration.spec.ts` (3 cases) proves the
per-org `product_no` sequences independently (Org A 1,2 · Org B 1), a 2-level category tree +
products + stock levels seed **edit-preservingly**, an opening level records a `purchase` movement
(the ledger reconciles) and the **low-stock rule holds** for a level below its reorder point, and
products/categories/levels/movements are **RLS-isolated** (Org A invisible to Org B; cross-tenant
write rejected by `WITH CHECK`). The **HTTP parts e2e** (`parts-http.e2e.spec.ts`, 8 cases) boots
the full Nest app + the Socket.io gateway and proves: a **2-level catalogue** builds with a per-org
`product_no`; **stock moves in/out** maintaining the level + the append-only ledger; the
**low-stock alert fires** on the crossing below reorder (audited `stock.low` + the low-stock
report); the **parts-request loop DRIVES the board** (technician raises → office orders → job →
`awaiting_parts`; arrive → job → `parts_arrived`, the part number sticks, the technician is
**notified over the hub** [`parts.arrived`]); **using a part on a job decrements stock**
(`used_on_job` movement references the job); an **inter-branch transfer** moves qty with
**goods-received** (transfer_out at source, transfer_in at destination); a **walk-in counter sale**
computes exact UK-VAT totals (£54 inc VAT) + decrements stock (`invoiceId` null until module 22);
and the surface is **permission-gated** (receptionist views + sells but 403 on catalogue edit;
technician views but 403 on sell) and **tenant-isolated** (Org B sees no products, 404 on Org A's
product). Full db suite (**34 suites / 304 tests**, incl. parts integration) + api suite (**26
suites / 152 tests**, incl. parts e2e) pass against the live DB — no cross-module regressions.
Seed idempotent (3 categories / 4 products / 4 stock levels Org A first run, 0 on re-run; Org B 0).
Live screenshot pending deploy. **Ends at `[CHECKPOINT]`.**

### 18 — Estimates & follow-ups — APPROVED 2026-06-18 (operator instruction)
Approved by the human operator (soft checkpoint). See entry below for build detail.

### 18 — Estimates & follow-ups — DONE (pending approval), 2026-06-18
The quoting flow (estimate → send → track viewed → authorise online → **convert-to-job**) plus
a **system-wide Follow-Up / Task engine** (due-dated callbacks attached to any record, with an
overdue flag — a real sales-recovery tool). Built on the module-14 Job pipeline, the module-11
notifications hub, module-12 customers/tax codes and the module-17 no-login authorisation
mechanism.

- **Data model** (`packages/db`): two TENANT-SCOPED tables (RLS) — `Estimate` (per-org
  `estimateNo` sequence, `lineItems` JSON, UK-VAT totals in **MINOR units (pence)** —
  `subtotalMinor`/`vatMinor`/`totalMinor`, `status` `open→authorised|declined|closed|converted`,
  `sentAt`/`sentVia`, `emailViewedAt`/`smsViewedAt` read tracking, `authorisedAt`/`authorisedVia`,
  `convertedJobId`, optional `evhcId`) and `FollowUp` (`type` estimate|evhc|job|customer|general,
  `relatedEntityType`/`Id`/`Label`, `dueAt`, `assigneeUserId`, `status` open|done|dismissed,
  completion log). New enums `EstimateStatus`/`FollowUpType`/`FollowUpStatus`. Migration
  `20260618280000_estimates_followups` (standard tenant-isolation RLS on both).
- **Source of truth + pure maths** (`packages/db/src/estimates.ts`, `followups.ts`):
  `ESTIMATE_STATUSES`, `normaliseLineItems`, **`computeEstimateTotals`** (the ONE place UK VAT
  is computed — per-line tax code, exact integer pence, never on a re-rounded subtotal),
  `lineTotalMinor`, `ESTIMATE_TEMPLATE_KEY` + `ESTIMATE_AUDIT_ACTIONS`; `FOLLOW_UP_TYPES`/
  `STATUSES`, **`followUpBucket`/`isFollowUpOverdue`** (the ONE overdue/today/upcoming rule),
  `FOLLOW_UP_AUDIT_ACTIONS`/`FOLLOW_UP_TEMPLATE_KEY`. `estimate-provision.ts`/`followup-provision.ts`
  (edit-preserving seeds). Added `estimate.*`/`followup.*` permissions (RBAC catalogue + deps +
  Owner/Manager/Receptionist grants — the front-desk sales-recovery flow), `estimates`+`followups`
  mode-registry entries, the `reminders.estimateFollowUpDays` setting (module-10 catalogue) and
  the `estimate.sent` (email/SMS/WhatsApp) + `followup.reminder` (in-app/email) notification
  templates (module-11 catalogue).
- **API** (`apps/api/src/estimates/`, `followups/`): `EstimatesService` — list/get/create/update
  (recompute totals; settled estimate locked)/`send` (stamp sentAt/sentVia, send the branded
  no-login link over the module-11 hub, **auto-suggest a chase follow-up** via FollowUpsService)/
  `convert` (authorised → create the Job via the ONE validated `JobsService.create`, carry the
  line summary, mark `converted`+link)/`remove`, plus the PUBLIC no-login `authorisePage`
  (read-tracking on the sent channel) + `submitAuthorisation` (approve/decline, system-actor
  audit). `FollowUpsService` — list (today/overdue/upcoming + counts + assignees)/create/update/
  complete/dismiss/reopen (who-when logging), optional in-app assignee reminder. Controllers
  (`SessionGuard`+`PermissionGuard`): estimate view/create/edit/send/convert/delete + follow-up
  view/manage; `PublicEstimatesController` UNAUTHENTICATED page+submit (resolves by opaque id on
  `ADMIN_DB`). Both services EXPORTED for later modules.
- **Web** (`apps/web`): `/estimates` — staff surface (`useMode('estimates')`, Simple/Pro): quick
  quote / line-item editor with tax codes, send, copy-link, **convert-to-job**, read/authorise
  tracking + a follow-up queue tab. `/e/[token]` — the PUBLIC, no-login, **branded** view/approve
  page. `/follow-ups` — `useMode('followups')`: Simple = "Callbacks today" + done; Pro = the full
  chase queue with type/status/assignee filters + the overdue flag. i18n extended
  (estimates/estimateAuthorise/followUps, no hardcoded strings); money via `formatCurrency`;
  dashboard "Estimates"+"Follow-ups" links gated. Contract: `packages/ui/ESTIMATES_CONTRACT.md`.
- **Seed**: Org A captures a sent (read-trackable) brake estimate (labour+parts, £312 inc VAT)
  with an **OVERDUE** chase follow-up linked to it + a general callback due today. Org B left
  empty (empty-state + isolation proof). Idempotent.

**Gates:** `lint` ✓ (0 warnings) · `typecheck` ✓ (db/types/api/web) · `build` ✓ (`/estimates`
6.01 kB, `/e/[token]` server-rendered, `/follow-ups` 3.89 kB) · `test` ✓ (db 199 unit incl. the
new estimate/follow-up pure cases; api 46 unit; DB-backed suites skip cleanly with no DB).
**Verified end-to-end against a real Postgres** (embedded, via `scripts/verify-db.mjs`): all
**17 migrations apply with zero drift** (`migrate diff` → "No difference detected"); app role
`rolsuper=false, rolbypassrls=false`; RLS **enabled on `estimate` + `follow_up`**. The new
`estimates.integration.spec.ts` (3 cases) proves the per-org `estimateNo` sequences
independently, an estimate seeds edit-preservingly with **exact UK-VAT totals** round-tripping
(£312.00 = £260 + 20%), a follow-up seeds incl. an **overdue** one (the red-flag rule holds),
and both tables are **RLS-isolated** (Org A invisible to Org B; cross-tenant write rejected by
`WITH CHECK`). The **HTTP estimates e2e** (`estimates-http.e2e.spec.ts`, 6 cases) boots the full
Nest app and proves: build an estimate with **exact VAT totals** (#1, £312); **send** notifies
the customer over the hub (`estimate.sent` row), **auto-suggests a chase follow-up**, returns the
`/e/:id` link; the **public page loads with NO login** (read-tracking stamps `emailViewedAt`;
invalid token 404s), **approve writes back** (`authorised`, £312) and a second response is 400;
**convert** an authorised estimate creates a Job carrying the lines + marks it `converted`
(non-authorised convert is 400); the **follow-up engine** creates/completes (who-when) + the
**overdue count** flags; and the surface is **permission-gated** (Technician 403 on estimates +
follow-ups) and **tenant-isolated** (Org B 404 on Org A's estimate). Full db suite (289, incl.
estimates integration) + api suite (144, incl. estimates e2e) pass against the live DB — no
cross-module regressions. Seed idempotent (1 estimate / 2 follow-ups Org A first run, 0 on
re-run; Org B 0). Live screenshot pending deploy. **Ends at `[CHECKPOINT]`.**

### 17 — EVHC authorisation — APPROVED 2026-06-18 (operator instruction)
Approved by the human operator (soft checkpoint). See entry below for build detail.

### 17 — EVHC authorisation — DONE (pending approval), 2026-06-18
The technician's inspection becomes a structured **Electronic Vehicle Health Check** (RAG
checklist + measurement grids + photo/video evidence + curated customer-facing
recommendations), the customer **approves/declines the recommended work online** on a
branded no-login page (the link travels over the module-11 hub), and the day's work prints
as a **Daily Job Sheet** in three role-scoped views. Built on the module-14 Job pipeline
(`awaiting_authorisation` → `approved`) and the module-15 realtime board.

- **Data model** (`packages/db`): one TENANT-SCOPED table (RLS) — `Evhc` (`sections` JSON
  checklist with a RAG per item, `measurements` JSON tyres/brakes/fluids grids,
  `recommendations` JSON customer-facing list `{key,title,detail,rag,price,mediaKeys,
  decision,declineReason}`, `media` JSON evidence referencing StoredObject blobs, `notes`/
  `nextServiceNote`, derived `authorisationState` + `sentAt`/`viewedAt`/`respondedAt` read
  tracking). New enum `EvhcAuthState`. Migration `20260618270000_evhc` (standard
  tenant-isolation RLS).
- **Source of truth + pure maths** (`packages/db/src/evhc.ts`): `EVHC_SECTIONS` (the real
  service-schedule structure — Initial checks / On the ground / Tyres / Raised / Brakes /
  Final checks), `RAG_STATUSES`, `normaliseSections`/`normaliseMeasurements`/
  `normaliseRecommendations` (clamp + the `preserveDecisions` rule), `deriveAuthState` (the
  ONE place the draft→sent→partial/authorised/declined lifecycle is decided), `recommendedTotal`,
  `summariseRag`, `DAY_SHEET_SCOPES`, `EVHC_TEMPLATE_KEY` + `EVHC_AUDIT_ACTIONS`.
  `evhc-provision.ts` (`seedEvhc`, edit-preserving by (org, job)). Added the `evhc`
  mode-registry entry, the `documents.healthCheckIntro` setting (module-10 catalogue) and the
  `evhc.authorisation` notification templates (email/SMS/WhatsApp, module-11 catalogue). No
  RBAC change — the existing `evhc.perform` (Technician+Owner) + `job.view`/`report.view` gate it.
- **API** (`apps/api/src/evhc/`): `EvhcService` — `jobContext`/`create`/`get`/`update`
  (normalise + save; a staff edit resets customer decisions), `addMedia` (camera-first
  base64 behind the OBJECT_STORAGE seam), `send` (completes + sends the branded link over the
  module-11 `NotificationsService`, advances the job to `awaiting_authorisation` via the ONE
  validated `JobsService.transition`, audits `evhc.sent`), `authorisePage`/`submitAuthorisation`
  (the PUBLIC no-login flow — resolves the EVHC by its opaque id on the `ADMIN_DB` client,
  records the view, writes the per-item decisions back, advances `awaiting_authorisation →
  approved` when any item is authorised, audits `evhc.responded`, system-actor) and `daySheet`
  (three scopes; branch/garage need `report.view` via `RolesService.effectivePermissions`).
  `EvhcController` (`SessionGuard`+`PermissionGuard`): context/get/day-sheet `job.view`,
  create/update/media/send `evhc.perform`. `PublicEvhcController` — UNAUTHENTICATED page +
  submit. `EvhcService` EXPORTED for later modules. GBP "Customer Explanation" slots into
  `recommendation.detail` in Phase 2 with no shape change.
- **Web** (`apps/web`): `/evhc` — staff surface (`useMode('evhc')`, Simple/Pro): RAG checklist
  with big green/amber/red buttons, Pro tyre/brake/fluid measurement grids (sub-legal tread
  warning), recommendation editor with per-item price + camera-first evidence, completion +
  one-tap send, and the authorisation tracking + copyable link. `/a/[token]` — the PUBLIC,
  no-login, **branded** authorisation page (plain-English items + cost + photos + big
  Approve/Decline per item, decline reason, thank-you done-state). `/day-sheet` — the daily
  job sheet (`useMode('evhc')`): Simple = today's list; Pro = scope tabs (only those allowed)
  + date + **print/PDF**. i18n extended (evhc/authorise/daySheet, no hardcoded strings);
  dashboard "Health checks" + "Job sheet" links gated by `job.view`. Contract:
  `packages/ui/EVHC_CONTRACT.md`.
- **Seed**: Org A captures a completed + sent EVHC on the in-progress service job — a
  sub-legal rear tyre (red, **approved** by the customer, £89.99) + worn front pads (amber,
  **pending**, £120), each with an evidence photo — so the authorisation tracking, the
  write-back and the day sheet have content. Org B left empty (empty-state + isolation proof).
  Idempotent.

**Gates:** `lint` ✓ (db/api/types/web, 0 warnings) · `typecheck` ✓ (db/types/api/web) ·
`build` ✓ (`/evhc` 5.88 kB, `/a/[token]` server-rendered, `/day-sheet` 3.41 kB) · `test` ✓
(db 179 unit incl. the 15 pure RAG/section/measurement/recommendation/auth-state cases; api
46 unit; DB-backed suites skip cleanly with no DB). **Verified end-to-end against a real
Postgres** (embedded, via `scripts/verify-db.mjs`): all **16 migrations apply with zero
drift** (`migrate diff` → "No difference detected"); app role `rolsuper=false,
rolbypassrls=false`; RLS **enabled on `evhc`**. The new `evhc.integration.spec.ts` (2 cases)
proves the EVHC seeds edit-preservingly by (org, job) (recommendations + media + the derived
`partially_authorised` state round-trip, blobs resolve) and is **RLS-isolated** (Org A
invisible to Org B; cross-tenant write rejected by `WITH CHECK`). The **HTTP EVHC e2e**
(`evhc-http.e2e.spec.ts`, 4 cases) boots the full Nest app and proves: a technician **builds
+ sends a health check** (6 sections + 3 RAG levels in context; evidence photo attached;
**the customer is notified over the hub** [`evhc.authorisation` notification row], the job
advances to `awaiting_authorisation`, audited `evhc.sent`, the branded `/a/:id` link
returned); the **public authorisation page loads with NO login** (recommended items + intro
+ photos; invalid token 404s; the view is recorded), **per-item approve/decline writes back**
(state → `partially_authorised`, approved total £89.99, **the job advances to `approved`**,
audited `evhc.responded`), and a re-submit is 400; the **daily job sheet** serves
**role-correct scopes** (Owner gets technician/branch/garage; Receptionist is clamped to the
technician-own scope); and the surface is **permission-gated** (Receptionist views context
but is 403 on starting a check) and **tenant-isolated** (Org B 404 on Org A's EVHC). Full db
suite (incl. EVHC integration) + api suite (incl. EVHC e2e) pass against the live DB — no
cross-module regressions (**24 suites / 138 api tests green**). Seed idempotent (1 EVHC/Org A
first run, 0 on re-run; Org B 0). Live screenshot pending deploy. **Ends at `[CHECKPOINT]`.**

### 16 — QR drop-off & VCR — APPROVED 2026-06-18 (operator instruction)
Approved by the human operator (soft checkpoint). See entry below for build detail.

### 16 — QR drop-off & VCR — DONE (pending approval), 2026-06-18
Two linked, job-attached, **legally-significant** capture flows over the module-14 Job
pipeline: (A) the **QR-code customer check-in** with a signed T&C consent and (B) the
**Vehicle Condition Report** (tyre treads, tap-to-mark damage markers, photos and dual
signatures). Realtime over the module-15 Socket.io channel — no polling. Both attach to the
Job and write the module-10 audit log.

- **Data model** (`packages/db`): two TENANT-SCOPED tables (RLS) — `BookingConsent` (the
  signed check-in: `confirmations` JSON array of `{key,value}`, `signatureKey`, `signedName`,
  `ip`/`userAgent` for the legal record) and `VehicleConditionReport` (`variant`
  dropoff|collection, snapshotted vrm/make/model/colour/mileage, `lockingWheelNut`,
  `interiorNote`, `treads` JSON, `damageMarkers` JSON array, `photoKeys` JSON, dual
  `customer/technicianSignatureKey`, `createdBy`). New enum `VcrVariant`. Migration
  `20260618260000_qr_dropoff_vcr` (standard tenant-isolation RLS on both).
- **Source of truth + pure maths** (`packages/db/src/dropoff.ts`): `DROPOFF_CONFIRMATIONS`
  (the real garage tick-list — `ack` items block submission, `yesno` items record an answer),
  `validateConfirmations` (the ONE consent rule), `TREAD_POSITIONS`/`normaliseTreads`,
  `DAMAGE_VIEWS`/`normaliseDamageMarkers` (clamp 0..1 + re-number 1..n + cap), `VCR_VARIANTS`/
  `isVcrVariant`, the image/photo caps and `DROPOFF_AUDIT_ACTIONS`. `dropoff-provision.ts`
  (`seedBookingConsent`/`seedConditionReport`, edit-preserving). Added the `documents.dropoffTerms`
  setting (module-10 catalogue), a `vcr` mode-registry entry and the `dropoff.presented`
  realtime event vocabulary (module-15 channel). No RBAC change — `job.view`/`job.edit` gate it.
- **API** (`apps/api/src/dropoff/`): `DropoffService` — `reception` (pending drop-offs +
  branded QR targets), `present` (audited + realtime nudge), `consentPage`/`submitConsent`
  (the PUBLIC no-login flow — validates, stores the signature, advances `booking → on_site`
  via the module-14 `JobsService.transition` [ONE validated/audited path, fires the board
  event], audits `consent.signed`), `listConsents`, `vcrContext` and `createVcr` (markers +
  treads + photos + dual signatures, audits `vcr.created`). `DropoffController`
  (`SessionGuard`+`PermissionGuard`): reception/present/reads gated by `job.view`, VCR capture
  by `job.edit`. `PublicDropoffController` — UNAUTHENTICATED consent page + submit (resolves
  the job by its opaque id on the `ADMIN_DB` client, then every write scoped to that job's
  org; captures caller IP/user-agent). Blobs behind the module-09 `OBJECT_STORAGE` seam; JSON
  body limit raised to 12 MB. `DropoffService` EXPORTED for later modules.
- **Web** (`apps/web`): `/reception` — the realtime QR display (`usePermission('job.view')`,
  branded idle/QR states via the garage's own branding, `QRCodeSVG` of the consent URL, live
  over Socket.io with a "Present" nudge); `/c/[jobId]` — the PUBLIC, no-login, **branded**
  consent page (booking details + house terms + the confirmation tick-list + a finger-friendly
  signature pad → check-in + "vehicle arrived" confirmation; `alreadySigned` done-state);
  `/dropoff` — staff VCR capture (`useMode('vcr')`, Simple/Pro): tap-to-mark damage diagram,
  tyre-tread grid (sub-legal warning), camera-first photos, dual signature pads, history.
  Reusable `signature-pad.tsx` + `damage-diagram.tsx`. i18n extended (reception/consent/vcr,
  no hardcoded strings); dashboard "Reception" + "Condition report" links gated by `job.view`.
  Contract: `packages/ui/DROPOFF_CONTRACT.md`.
- **Seed**: Org A attaches a signed booking consent (Sam Driver) to the service job and
  captures a drop-off VCR (damage markers + low rear treads + a photo + dual signatures) plus
  a collection-variant VCR on the clutch job (both variants proven). Org B left empty (the
  empty-state + isolation proof). Idempotent.

**Gates:** `lint` ✓ (db/api/types/web, 0 warnings) · `typecheck` ✓ (db/types/api/web) ·
`build` ✓ (`/reception` 9.62 kB, `/c/[jobId]` server-rendered, `/dropoff` built) · `test` ✓
(db 164 unit incl. the 10 pure confirmation/tread/marker/variant cases; api 46 unit;
DB-backed suites skip cleanly with no DB). **Verified end-to-end against a real Postgres**
(embedded, via `scripts/verify-db.mjs`): all **15 migrations apply with zero drift**
(`migrate diff` → "No difference detected"); app role `rolsuper=false, rolbypassrls=false`;
RLS **enabled on `booking_consent` + `vehicle_condition_report`**. The new
`dropoff.integration.spec.ts` (3 cases) proves a signed consent seeds edit-preservingly
(answers + signature blob round-trip), a condition report's treads/markers/photos/dual
signatures round-trip and the dropoff/collection variants are DISTINCT rows, and both tables
are **RLS-isolated** (cross-tenant write rejected by `WITH CHECK`). The **HTTP drop-off e2e**
(`dropoff-http.e2e.spec.ts`, 5 cases) boots the full Nest app + the Socket.io gateway and
proves: the reception display lists pending drop-offs and **presenting one fans a realtime
`dropoff.presented` event to a SECOND socket.io-client session** (no polling); the **public
consent page loads with NO login** (booking details + confirmations + terms; an invalid token
404s); **signing the consent (no login) records it, advances `booking → on_site`
(`vehicleArrived`), fans the board `job.status` event to a second session, audits
`consent.signed`**, the office reads it back + the signature serves over the asset URL, and a
re-submit / a missing acknowledgement are both 400; **VCR capture stores damage markers
(coords+notes), tyre treads, a photo blob (served) + dual signatures, audits `vcr.created`,
the collection variant works** and an unknown variant 400s; and the surface is
**permission-gated** (Receptionist views the reception display but is 403 on VCR capture) and
**tenant-isolated** (Org B's reception empty, 404 on Org A's consents). Full db suite (249,
incl. drop-off integration) + api suite (134, incl. drop-off e2e) pass against the live DB —
no cross-module regressions. Seed idempotent (1 consent + 2 condition reports/Org A first run,
0 on re-run; Org B 0). Live screenshot pending deploy. **Ends at `[CHECKPOINT]`.**

### 15 — Workshop / technician — APPROVED 2026-06-18 (operator instruction)
Approved by the human operator (soft checkpoint). See entry below for build detail.

### 15 — Workshop / technician — DONE (pending approval), 2026-06-18
Two realtime surfaces over the module-14 Job pipeline: the glanceable big-screen
**workshop board** (Simple-only by nature) and the ultra-minimal **technician PWA**, both
updating live over **Socket.io** (no polling). The board mirrors the familiar WIP layout;
the PWA is do-the-task-and-go on the shop floor.

- **Realtime fan-out** (`apps/api/src/realtime`): a Socket.io `RealtimeGateway` that
  authenticates a connection by its `ax_session` cookie through the SAME module-03 suite
  seam, then joins it to an `org:<orgId>` room — realtime respects tenant isolation just
  like RLS. ONE channel (`WORKSHOP_CHANNEL = 'workshop'`) carries a typed
  `WorkshopEventPayload`; `RealtimeService.emit(orgId, type, jobId)` is the emit surface.
  `JobsService` now emits on create/update/status/clock; `WorkshopService` on
  allocate/parts/attachment. Modules 16 (QR arrival) + 19 (parts-arrived) reuse the channel.
- **Data model** (`packages/db`): two TENANT-SCOPED tables (RLS) — `JobAttachment`
  (the camera-first uploads: photo/advisory/scan/document; blob behind the module-09
  object-storage seam, referenced by `storageKey`; advisory may be text-only) and
  `JobPartRequest` (the lightweight technician parts request — the on-ramp to module 19).
  Allocation reuses the existing `job.assigned_technician_ids` JSON (no schema change). New
  enums `JobAttachmentKind`/`PartRequestStatus`. Migration `20260618250000_workshop`
  (standard tenant-isolation RLS on both).
- **Source of truth + pure maths** (`packages/db/src/workshop.ts`): `BOARD_BUCKETS` +
  `bucketForStatus` (the familiar WIP layout; `booking`→Booked column, `collected`→off
  board), `classifyBooking` (the booked-column breakdown — flags win over reason text),
  `ATTACHMENT_KINDS`/`PART_REQUEST_STATUSES` catalogues + validators, the `WORKSHOP_EVENTS`
  realtime vocabulary and `WORKSHOP_AUDIT_ACTIONS`. `workshop-provision.ts`
  (`allocateJobToTechnicians`/`seedPartRequest`/`seedAttachment`/`findSeededJobId`,
  edit-preserving). Waiting-parts vs parts-arrived stay DISTINCT colours (module-01 tokens).
- **API** (`apps/api/src/workshop/`): `WorkshopService` — `board` (booked breakdown + WIP
  buckets + technicians [= users holding `workshop.clock`] + unallocated), `myJobs` (the
  tech's queue), `technicians`, `allocate` (audited + realtime), `requestPart`/
  `updatePartRequest` (office alert + realtime), `addAttachment` (camera-first, base64,
  object-storage seam) and `advance` (delegates to the module-14 `JobsService.transition`
  — ONE validated/audited path, gated for the floor by `workshop.clock`). `WorkshopController`
  (`SessionGuard`+`PermissionGuard`): board/my-jobs/technicians/reads (`workshop.view`),
  allocate (`job.assign`), advance/upload (`workshop.clock`), part-request (`parts.request`),
  part-status (`parts.edit`). No RBAC catalogue change. JSON body limit raised to 6 MB.
- **Web** (`apps/web`): `/board` — the realtime board (`useMode('workshop')`, Simple-only):
  Booked column + breakdown, WIP buckets (status-token colours, waiting-vs-arrived distinct),
  Technicians column, Unallocated area; allocation popover gated by `job.assign`; live via
  Socket.io with a debounced reload + a "Live / as of" indicator. `/tech` + `/tech/[jobId]`
  — the installable technician PWA (`manifest.webmanifest`, `tech-sw.js` network-first SW,
  `PwaRegister`): big-card my-jobs, one-big-button clock, validated status advance, request-
  part, camera-first photo/advisory/scan/document uploads. i18n extended (board + tech
  catalogues, no hardcoded strings). Dashboard "Workshop" + "Technician" links gated.
  Contract: `packages/ui/WORKSHOP_CONTRACT.md`.
- **Seed**: Org A allocates the in-progress service job to Tariq Tech + the clone-reg
  investigation to Tina Tech (two new technician users), leaves the on-stop clutch job
  UNALLOCATED, raises a tyre parts request and records an advisory + a tyre photo. Org B
  left empty (empty-board + isolation proof). Idempotent.

**Gates:** `lint` ✓ (0 warnings) · `typecheck` ✓ (db/types/api/web) · `build` ✓ (`/board`
3.59 kB, `/tech` 1.59 kB, `/tech/[jobId]` 3.83 kB) · `test` ✓ (db 236 unit incl. the 10
pure bucket/booked/catalogue cases; api 46 unit; DB-backed suites skip cleanly with no DB).
**Verified end-to-end against a real Postgres** (embedded, via `scripts/verify-db.mjs`):
all **14 migrations apply with zero drift** (`migrate diff` → "No difference detected"); app
role `rolsuper=false, rolbypassrls=false`; RLS **enabled on `job_attachment` +
`job_part_request`**. The new `workshop.integration.spec.ts` (4 cases) proves allocation
round-trips the JSON list (idempotent), the part-request + advisory seeds are edit-preserving,
a photo blob round-trips through the object store, and attachments + part requests are
**RLS-isolated** (cross-tenant write rejected by `WITH CHECK`). The **HTTP workshop e2e**
(`workshop-http.e2e.spec.ts`, 8 cases) boots the full Nest app + the Socket.io gateway and
proves: the board renders buckets/booked-breakdown/technicians/unallocated; **allocation
fans a realtime `job.allocated` event to a SECOND socket.io-client session** (live across
two sessions, no polling); the technician PWA queue shows ONLY that tech's jobs; a technician
**advances status (validated — illegal move 400s), clocks on, but is 403 on allocate**; a
technician **raises a parts request (audited `job.part.request`) the office reads back** (the
board card shows the open count); uploads an **advisory (text-only) + a photo blob served
over the asset URL** (a file-less photo 400s); and the surface is **permission-gated**
(Receptionist 403 on the board + allocate) and **tenant-isolated** (Org B's board empty).
Full db suite (236, incl. workshop integration) + api suite (129, incl. workshop e2e) pass
against the live DB — no cross-module regressions. Seed idempotent (3 jobs / 2 allocations /
1 part request / 2 uploads first run, all 0 on re-run; Org B 0). Live screenshot pending
deploy. **Ends at `[CHECKPOINT]`.**

### 14 — New job pipeline — APPROVED 2026-06-18 (operator instruction)
Approved by the human operator (soft checkpoint). See entry below for build detail.

### 14 — New job pipeline — DONE (pending approval), 2026-06-18
The canonical **Job** entity — the spine of the whole system — plus the reg-first New Job
wizard (reg → live DVLA → customer → editable prefs → create) and the full, validated,
audited status pipeline. Everything downstream (workshop, EVHC, parts, invoicing, P&L)
hangs off Job.

- **Data model** (`packages/db`): two TENANT-SCOPED tables (RLS) — `Job` (per-org human
  `jobNo`, nullable branch/vehicle/customer, frozen `dvlaSnapshot` JSON, `JobStatus`
  enum, reason/requestedWork, `assignedTechnicianIds`/`flags`/`notificationPrefsSnapshot`
  JSON, `onStopFlagged`, bookingDate/onSiteAt, `clockStartAt`/`clockEndAt`/`clockSeconds`
  clocking, createdBy) and append-only `JobStatusEvent` (from→to/byUser/note/at). New
  `JobStatus` enum (16 canonical states). Migration `20260618240000_jobs_pipeline`
  (standard tenant-isolation RLS on both).
- **Source of truth + pure maths** (`packages/db/src/jobs.ts`): the `JOB_STATUSES`
  catalogue (Pro label + Simple bucket + workshop-status colour `token` + side-state/
  terminal flags), the validated transition graph (`canTransition`/`allowedTransitions` —
  the ONE place the pipeline rules live; any active job → a side state; `collected`
  terminal; a quick `booking → collected` drop-off allowed), `primaryNextStatus` (the
  Simple "one obvious next action"), `simpleStatusFor` + `SIMPLE_STATUS_LABELS` (the
  **lossless** Simple collapse), the `JOB_FLAGS` familiar-tag catalogue + `normaliseJobFlags`,
  `totalClockSeconds`/`isClockedOn` clocking, `buildJobSearchTerms` and `JOB_AUDIT_ACTIONS`.
  `job-provision.ts` (`seedJob`, edit-preserving by (org, vehicle, reason); allocates jobNo
  + seeds the initial event). No RBAC/mode change — `job.view/create/edit/assign/delete`,
  `workshop.clock` + the `jobs` mode entry already exist.
- **API** (`apps/api/src/jobs/`): `JobsService` — list/search (status/simple-bucket/
  active filters), get (incl. status history), **`lookup`** (the reg-first DVLA preview),
  **`create`** (THE wizard: resolve/inline-add customer → find-or-create vehicle → LIVE
  DVLA fetch [free, module 13] freezing the snapshot → resolver-prefilled prefs →
  on-stop warn-but-flag → allocate jobNo → open in `booking` + initial event),
  update, **`transition`** (validated + writes a JobStatusEvent + audits), `clockOn`/
  `clockOff` (accumulate) and remove. Composes the module-13 `VehiclesService`, module-12
  `CustomersService`, module-10 `AuditService` + `SettingsService`. `JobsController`
  (`SessionGuard`+`PermissionGuard`): `GET /jobs` + `GET /jobs/:id` (`job.view`),
  `POST /jobs/lookup` + `POST /jobs` (`job.create`), `PATCH /jobs/:id` + `POST
  /jobs/:id/transition` (`job.edit`), `POST /jobs/:id/clock-{on,off}` (`workshop.clock`),
  `DELETE /jobs/:id` (`job.delete`). `JobsService` EXPORTED for later modules. Added a
  `findByVrm` helper to `VehiclesService`.
- **Web** (`apps/web`): `/jobs` page dogfoods the mode framework (`useMode('jobs')`):
  Simple = the reg-first wizard (fewest fields, friendly status, one obvious next action),
  Pro adds the full pipeline picker, flags, clocking detail and the audited status
  history. Status chips use the module-01 workshop-status tokens. Familiar primary-action
  set present (New Job / New Estimate [stub] / New Customer / Parts-Only Sale [stub]).
  i18n catalogue extended (no hardcoded strings); discreet dashboard "Jobs" link gated by
  `job.view`. Contract: `packages/ui/JOBS_CONTRACT.md`.
- **Seed**: Org A gets 3 proof jobs across the pipeline — an in-progress service, a
  **clone-reg** investigation (`CL0 NE1`), and an **on-stop** account-customer clutch job
  (flagged) — each with its initial status event. Org B left empty (isolation proof).
  Idempotent (edit-preserving by (org, vehicle, reason)).

**Gates:** `lint` ✓ (0 warnings) · `typecheck` ✓ (db/types/api/web) · `build` ✓
(`/jobs` route compiles, 5.85 kB) · `test` ✓ (db 144 unit incl. the 24 pure
status/transition/flag/clock cases; api 46 unit; DB-backed suites skip cleanly with no
DB). **Verified end-to-end against a real Postgres** (embedded): all **13 migrations apply
with zero drift** (`migrate diff` → "No difference detected" against a separate shadow
DB); app role `rolsuper=false, rolbypassrls=false`; RLS **enabled on `job` +
`job_status_event`**, **OFF** on the platform `plan`. The new `jobs.integration.spec.ts`
(6 cases) proves the seed allocates a per-org `jobNo` + default branch + an **initial
status event**, is **edit-preserving by (org, vehicle, reason)** (no dup, no jobNo
reshuffle), jobNo **sequences independently per org** (Org B restarts at 1), and jobs +
status events are **RLS-isolated** (cross-tenant write rejected by `WITH CHECK`). The
**HTTP jobs e2e** (`jobs-http.e2e.spec.ts`, 8 cases) boots the full Nest app and proves:
the **reg-first wizard** (live DVLA preview → create with jobNo #1 + frozen snapshot +
initial event); an **inline new-customer** create links the customer; the **clone reg
raises the mismatch alert at creation but still opens the job** (never blocks) + audits
`vehicle.dvla.mismatch`; an **on-stop customer warns + flags** (job created, `onStopFlagged`);
status transitions are **validated** (legal `booking → on_site` stamps `onSiteAt` + logs
an event + audits `job.status`; illegal `on_site → collected` is 400); **Simple collapses
`awaiting_parts` → "waiting_parts" while the full status stays stored** (lossless);
**clocking** on→off accumulates `clockSeconds`; **tenant isolation** (Org B sees none);
and the surface is **permission-gated** (Receptionist creates + views but is **403 on
transition / clock / delete**; Technician **can clock**). Full db suite (222, incl. jobs
integration) + api suite (121, incl. jobs e2e) pass against the live DB — no cross-module
regressions. Seed is idempotent (3 new jobs/Org A first run, 0 on re-run; Org B 0). Live
screenshot pending deploy. **Ends at `[CHECKPOINT]`.**

### 13 — Vehicles & DVLA — APPROVED 2026-06-18 (operator instruction)
Approved by the human operator (soft checkpoint). See entry below for build detail.

### 13 — Vehicles & DVLA — DONE (pending approval), 2026-06-18
Vehicle records keyed by registration, with a **LIVE DVLA fetch on every new job**
(never the cache — plate cloning), a cached DVSA MOT history, and the per-job
**notification-preference resolver** (the smart pre-fill chain spec 14 consumes). DVLA +
DVSA are FREE — never metered.

- **Data model** (`packages/db`): three TENANT-SCOPED tables (RLS) — `Vehicle`
  (keyed by normalised `vrm`, nullable `customerId` [a vehicle changes owner],
  make/model/colour/fuelType/year/engineSize/vin/mileageLast, `dvlaSnapshot` JSON,
  `notes`), `MotRecord` (DVSA cache, upsert by `(org, vehicle, testDate)`;
  result/expiry/odometer/`advisories`/`failures` JSON) and `DvlaFetchLog` (every live
  fetch; `mismatch` flag + JSON `result` — the cloning trail). Migration
  `20260618230000_vehicles_dvla` (standard tenant-isolation RLS on all three).
- **Source of truth + pure maths** (`packages/db/src/vehicles.ts`): `normaliseVrm`
  (uppercase/no-space — the ONE lookup key), `formatVrmDisplay`, **`detectMismatch`**
  (only a genuine contradiction between two KNOWN make/model values counts — a
  previously-blank field being filled is NOT a mismatch), `motStatus(expiry, now,
  dueSoonDays)` (valid/due_soon/expired/unknown + days), `vehicleMatchesQuery`/
  `buildVehicleSearchTerms`, **`garageDefaultConsent`** (resolver step 3 — operational/
  reminders default allowed, marketing follows `customer.marketingOptInDefault`), the
  `DVLA_ACTION_KEY`/`DVSA_ACTION_KEY` (free) and `VEHICLE_AUDIT_ACTIONS`.
  `vehicle-provision.ts` (`seedVehicle`/`seedMotRecord`, edit-preserving by reg/test
  date). No RBAC/mode change — `vehicle.view/create/edit` + the `vehicles` mode entry
  already exist; `dvla.lookup`/`dvsa.mot` are already on the free allowlist.
- **Provider seam** (`apps/api/src/vehicles/vehicle-data-provider.ts`): `VehicleDataProvider`
  (`lookupVehicle`/`fetchMotHistory`) behind `VEHICLE_DATA_PROVIDER` →
  `MockVehicleDataProvider` (Phase 1, deterministic — a known book + a CLONE case
  [returns Vauxhall Astra for a reg stored as Ford Focus] + a not-found set + a stable
  pseudo-record fallback so a lookup "works live" every time). Swap the one binding for
  the real DVLA/DVSA adapters later.
- **API** (`apps/api/src/vehicles/`): `VehiclesService` — list/search, get (incl. MOT
  history + fetch log)/create/update/remove, **`lookupDvla`** (THE live fetch — FREE, no
  preflight; stores the snapshot ALWAYS; on a mismatch raises an alert + flags the log +
  audits, does NOT overwrite make/model; not-found is graceful + never blocks),
  **`refreshMot`** (DVSA cache, free, idempotent) and **`resolvePreferences`** (the
  3-step chain: reg-owner → named customer → garage default; always editable per job).
  Composes the module-10 `AuditService` + `SettingsService`. `VehiclesController`
  (`SessionGuard`+`PermissionGuard`): `GET /vehicles` + `GET /vehicles/resolve-preferences`
  + `GET /vehicles/:id` + `POST /vehicles/:id/{dvla-lookup,mot-refresh}` (`vehicle.view`
  — the New Job picker is used by reception), `POST /vehicles` (`vehicle.create`),
  `PATCH`/`DELETE /vehicles/:id` (`vehicle.edit`). `VehiclesService` EXPORTED for module 14.
- **Web** (`apps/web`): `/vehicles` page dogfoods the mode framework (`useMode('vehicles')`):
  Simple = reg-in/details-out, MOT shown plainly, a friendly **mismatch banner** + a free
  "Check with DVLA"; Pro adds the full record, MOT history (advisories/failures), the DVLA
  fetch log, snapshot and manual overrides. i18n catalogue extended (no hardcoded strings);
  discreet dashboard "Vehicles" link gated by `vehicle.view`. Contract:
  `packages/ui/VEHICLES_CONTRACT.md` (the live-DVLA + free + resolver recipe for spec 14).
- **Seed**: Org A gets a 3-vehicle fleet linked to its customer book — a Ford Fiesta +
  Vauxhall Astra (known DVLA + cached MOT incl. a fail) and a **clone proof** (`CL0 NE1`
  stored as a Ford Focus; mock DVLA returns a Vauxhall Astra → deterministic mismatch).
  Org B left empty (isolation proof). Idempotent.

**Gates:** `lint` ✓ (0 warnings) · `typecheck` ✓ (db/types/api/web) · `build` ✓
(`/vehicles` route compiles, 4.71 kB) · `test` ✓ (db 127 unit incl. the 25 pure
vrm/mismatch/mot/search/consent cases; api 46 unit; DB-backed suites skip cleanly with no
DB). **Verified end-to-end against a real Postgres** (embedded): all **12 migrations apply
with zero drift** (`migrate diff` → "No difference detected" against a separate shadow DB);
app role `rolsuper=false, rolbypassrls=false`; RLS **enabled on `vehicle` + `mot_record` +
`dvla_fetch_log`**, **OFF** on the platform `plan`. The new `vehicles.integration.spec.ts`
(5 cases) proves the vrm is stored **normalised**, the seed is **edit-preserving by (org,
vrm)**, the MOT cache is **refreshable by test date** (no dup), and vehicles + MOT records
+ fetch logs are **RLS-isolated** (cross-tenant write rejected by `WITH CHECK`; a mismatch
flag round-trips). The **HTTP vehicles e2e** (`vehicles-http.e2e.spec.ts`, 8 cases) boots
the full Nest app and proves: create **normalises the reg** + search finds by reg/make; a
**live DVLA lookup is FREE** (balance untouched, **0 `dvla.lookup` ledger rows**, snapshot
stored); the **CLONE case raises the mismatch alert, keeps the stored Ford Focus, stores
the fresh Vauxhall snapshot, flags the log + audits `vehicle.dvla.mismatch`**; a
**not-found reg is graceful** (200, `found:false`, never blocks); the **MOT cache** fetches
+ is idempotent; the **resolver returns all 3 paths** (reg-owner consent → named-customer
consent → garage default with marketing OFF); **tenant isolation** (Org B sees none); and
the surface is **permission-gated** (Receptionist views + adds + looks up but is **403 on
edit / delete**). Full db suite (200, incl. vehicles integration) + api suite (112, incl.
vehicles e2e) pass against the live DB — no cross-module regressions. Seed is idempotent
(3 new vehicles + 4 MOT records/Org A first run, 0 on re-run; Org B 0). Live screenshot
pending deploy. **Ends at `[CHECKPOINT]`.**

### 12 — Customers — APPROVED 2026-06-18 (operator instruction)
Approved by the human operator (soft checkpoint). See entry below for build detail.

### 12 — Customers — DONE (pending approval), 2026-06-18
The customer master every later module (jobs, estimates, invoicing, notifications,
plans/coverage) hangs off: UK trade-counter account controls, THREE independent
channel-selectable consent streams (GDPR-clean — marketing separate + opt-in), customer
groups, and fast multi-field search (the New Job picker).

- **Data model** (`packages/db`): two TENANT-SCOPED tables (RLS) — `Customer`
  (`customerNo` human-friendly per-org sequence, name/company/email/`phones` JSON +
  denormalised `phonesText` for search, flat address, account controls
  [`isAccountCustomer`/`onStop`/`paymentTerms`/accounting codes/`taxCode`/
  `requireCustomerReferenceForInvoice`], `priorityMessage`, `leadSource`,
  `identificationNumber`, and the three consent JSON columns) and `CustomerGroup`
  (`(org, name)` segmentation). Migration `20260618220000_customers` (standard
  tenant-isolation RLS on both).
- **Source of truth + pure maths** (`packages/db/src/customers.ts`): the
  `CUSTOMER_TAX_CODES` catalogue (standard/reduced/zero/exempt), the `CONSENT_STREAMS`
  metadata + `defaultConsent`/`normaliseConsent` (no-row-means-default, marketing
  defaults opted-OUT), and `resolveConsent` — **the ONE place suppression is decided**
  (explicit `no` ⇒ opted_out; marketing needs explicit opt-in; operational/reminders
  allow unless opted out; then the channel must match the stream's preference) +
  `streamForTemplateKey`, `customerDisplayName`, `buildCustomerSearchTerms`/
  `customerMatchesQuery`, `normalisePhones`, and the `CUSTOMER_AUDIT_ACTIONS` vocabulary.
  `customer-provision.ts` (`seedCustomer`/`seedCustomerGroup`, edit-preserving by email).
- **API** (`apps/api/src/customers/`): `CustomersService` — list/**multi-field fuzzy
  search** (name/company/email/postcode/`customerNo`/phone — the New Job picker),
  get/create (allocates the next `customerNo`, retried on a race)/update/
  `setAccountControls`/`setConsent`/`remove`/`createGroup`, plus `notifyCustomer` —
  **THE consent-gated send** (resolve consent → suppress [log a `failed` row + audit] OR
  hand off to the module-11 `NotificationsService.send`). Composes the module-10
  `AuditService` (privileged-change trail) + `SettingsService` (locale) + module-11
  `NotificationsService`. `CustomersController` (`SessionGuard`+`PermissionGuard`):
  `GET /customers` (`customer.view`), `POST /customers` (`customer.create`),
  `GET /customers/:id` (`customer.view`), `PATCH /customers/:id` + `PUT
  /customers/:id/{account,consent}` + `POST /customers/groups` (`customer.edit`),
  `POST /customers/:id/notify` (`notifications.send`), `DELETE /customers/:id`
  (`customer.delete`). `CustomersService` EXPORTED for later modules. No RBAC/mode
  changes needed (the `customer.*` permissions + `customers` mode entry already exist).
- **Web** (`apps/web`): `/customers` page dogfoods the mode framework
  (`useMode('customers')`): Simple = find/add with the essentials + a detail panel with
  the **priorityMessage banner**, **on-stop warning**, contact and a consent-gated send;
  Pro adds filters (group/account-only/on-stop), groups management, the full account-
  controls editor (incl. the tax-code select) and the **3×channel consent matrix**.
  i18n catalogue extended (no hardcoded strings); discreet dashboard "Customers" link
  gated by `customer.view`. Contract: `packages/ui/CUSTOMERS_CONTRACT.md`.
- **Seed**: Org A gets a 3-customer book — an account-customer **on stop** (JT Haulage),
  one with a **priorityMessage** (Sam Driver), and one **marketing-opted-out** (Priya
  Patel) — so the suppression proof is deterministic. Org B left empty (the empty-state
  + isolation proof). Idempotent (edit-preserving by email).

**Gates:** `lint` ✓ (0 warnings) · `typecheck` ✓ (db/types/api/web) · `build` ✓
(`/customers` route compiles, 8.88 kB) · `test` ✓ (db 108 unit incl. the 23 pure
consent/tax/search cases; api 46 unit; DB-backed suites skip cleanly with no DB).
**Verified end-to-end against a real Postgres** (embedded): all **11 migrations apply
with zero drift** (`migrate diff` → "No difference detected"); app role
`rolsuper=false, rolbypassrls=false`; RLS **enabled on `customer` + `customer_group`**,
**OFF** on the platform `plan`. The new `customers.integration.spec.ts` (5 cases) proves
the per-org `customerNo` sequences independently (Org A 1,2 · Org B 1), the seed is
edit-preserving by email, customers + groups are **RLS-isolated** (cross-tenant write
rejected by `WITH CHECK`), and on-stop/priorityMessage round-trip with the stored
consent suppressing marketing while allowing operational. The **HTTP customers e2e**
(`customers-http.e2e.spec.ts`, 7 cases) boots the full Nest app and proves: create
**sequences the customer number** (#1, #2); **search finds by name / phone / postcode**;
the on-stop + priorityMessage surface and an **account change is audited** (before/after);
a **marketing send to an opted-out customer is suppressed + logged** (a `failed` row +
`customer.notify.suppressed` audit) **while an operational send proceeds**; marketing
proceeds once the customer **opts in**; **tenant isolation** (Org B sees none of Org A's
customers, its own sequence restarts at #1); and the surface is **permission-gated**
(Receptionist views + adds but is **403 on edit / account / delete**). Full db suite
(176, incl. customers integration) + api suite (104, incl. customers e2e) pass against
the live DB — no cross-module regressions. Seed is idempotent (3 new customers/Org A
first run, 0 on re-run; Org B 0). Live screenshot pending deploy. **Ends at `[CHECKPOINT]`.**

### 11 — Notifications — APPROVED 2026-06-18 (operator instruction)
Approved by the human operator (soft checkpoint). See entry below for build detail.

### 11 — Notifications — DONE (pending approval), 2026-06-18
The unified outbound comms hub every later module sends through (`notify.send`):
per-garage sender identity, template management (with platform defaults), credit
metering with cross-channel fallback, and delivery/read tracking. Channels:
email/SMS/WhatsApp (metered, module 07) + in-app/push (free). Providers MOCKED in
Phase 1 behind a swappable seam.

- **Data model** (`packages/db`): three TENANT-SCOPED tables (RLS) — `NotificationChannelConfig`
  (one per `(org, channel)`, JSON sender identity + `verifiedStatus`), `NotificationTemplate`
  (`(org, key, channel, locale)`; a NULL `org_id` is a PLATFORM DEFAULT readable by every
  garage), and `Notification` (the send log — rendered content, status lifecycle,
  `providerMessageId`, `creditCost`/`creditTxnId` link, `relatedEntity*`, `sentAt`/
  `deliveredAt`/`readAt`). New enums `NotificationChannel`/`ChannelVerifyStatus`/
  `NotificationStatus`. Migration `20260618210000_notifications`. **Templates use a special
  RLS policy**: the read `USING` clause ALSO matches `org_id IS NULL` (garages read platform
  defaults), while `WITH CHECK` only ever permits a garage to write its OWN rows.
- **Source of truth + pure maths** (`packages/db/src/notifications.ts`): `METERED_CHANNEL_ACTION`
  (email→`email.send`/sms→`sms.send`/whatsapp→`whatsapp.send`; in-app/push free),
  `channelActionKey`/`isMeteredChannel`, `renderTemplate` (`{{var}}` interpolation — an
  unfilled var is left verbatim + reported, never silently blanked) + `templateVariables`,
  `resolveSender` (the ONE place the sender rules live: email own-verified-domain-else-default,
  sms own-registered-id-else-default, whatsapp connected-else-UNAVAILABLE), the cross-channel
  fallback order (`whatsapp→sms→email`, `sendChannelOrder`), the `DEFAULT_TEMPLATES` catalogue
  (booking.confirmation/mot.reminder/estimate.sent/job.ready) and the `NOTIFICATION_AUDIT_ACTIONS`
  vocabulary. `notification-provision.ts` (`seedDefaultTemplates` edit-preserving, `seedChannelConfig`).
- **API** (`apps/api/src/notifications/`): `NotificationsService` — THE `notify.send` pipeline
  (resolve template → render → resolve sender → preflight → dispatch → log → charge, walking
  the fallback chain; an undeliverable message is QUEUED + audited, never dropped) plus
  `viewForOrg`/`setChannelConfig`/`upsertTemplate`/`markDelivered`/`markRead`. Composes the
  module-07 `MeteringService` (preflight/charge metered channels, links the `CreditTransaction`),
  the module-10 `SettingsService` (locale + the new `alerts.notificationFallback` toggle) +
  `AuditService` (fallback/queued/config/template events), behind a swappable provider seam
  (`NOTIFICATION_DISPATCHER` → `MockNotificationDispatcher` in Phase 1). `NotificationsController`
  (`SessionGuard`+`PermissionGuard`): `GET /notifications` (`notifications.view`), `PUT
  /notifications/channels/:channel` + `PUT /notifications/templates/:key/:channel` + `POST
  /notifications/:id/{delivered,read}` (`notifications.manage`), `POST /notifications/send`
  (`notifications.send`). `NotificationsService` EXPORTED so every later module sends through it.
- **RBAC + mode + settings extended**: added `notifications.view`/`notifications.manage`/
  `notifications.send` (Settings group; manage+send → view; Owner `*` + Manager get all,
  Receptionist gets view+send but NOT manage). Added a `notifications` mode-registry entry
  (both faces). Added the `alerts.notificationFallback` setting (default on) — dogfoods the
  module-10 catalogue extension contract.
- **Web** (`apps/web`): `/notifications` page dogfoods the mode framework (`useMode('notifications')`):
  Simple = "Messages" (channel-status badges, recent sends, a live test-send), Pro = animated
  delivery analytics, per-channel sender-identity status + fallback rule, template list with
  variables + preview, and per-message mark-delivered/read. i18n catalogue extended (no hardcoded
  strings); discreet dashboard "Messages" link gated by `notifications.view`. Contract:
  `packages/ui/NOTIFICATIONS_CONTRACT.md` (the `notify.send` recipe for every later module).
- **Seed**: platform default templates (global, edit-preserving); Org A configured with a
  mock-verified email domain + registered SMS sender (WhatsApp left unconfigured → fallback),
  plus two sample messages (a delivered email + a read SMS); Org B left unconfigured (the
  needs-setup/upsell path). Idempotent.

**Gates:** `lint` ✓ (0 warnings) · `typecheck` ✓ (db/types/api/web) · `build` ✓ (`/notifications`
route compiles) · `test` ✓ (db 156, api 97 — DB-backed suites skip cleanly with no DB).
**Verified end-to-end against a real Postgres** (embedded): all **10 migrations apply with zero
drift** (`migrate diff` → "No difference detected" against a separate shadow DB); app role
`rolsuper=false, rolbypassrls=false`; RLS **enabled on all 3 notification tables**, **OFF** on
the platform `plan`/`credit_price_list`. The new `notifications.integration.spec.ts` (7 cases)
proves the platform-default templates seed edit-preservingly + are READABLE under tenant context
but a garage **cannot write a NULL-org template** (`WITH CHECK`), an org override wins + is
invisible to others, channel configs + the send log are **RLS-isolated** (cross-tenant write
rejected), and the garage audit path records the fallback event. The **HTTP notifications e2e**
(`notifications-http.e2e.spec.ts`, 7 cases) boots the full Nest app and proves: the catalogue-
driven view (own email identity, WhatsApp needs-setup); a **metered email send renders +
dispatches + logs + charges 2 credits and links the CreditTransaction** (`reference` = message id);
a **WhatsApp send falls back to SMS** when WhatsApp is unavailable (logged + audited, attempts
recorded); an **in-app send is FREE** (no charge, balance unchanged); an exhausted chain (Org B,
zero credits) is **QUEUED + alerted** (all 3 channels attempted, nothing dropped); delivery+read
tracking flips status + `readAt`; and the surface is **permission-gated** (Receptionist views +
sends but is **403 on manage**). Full db suite (156) + api suite (97, incl. notifications e2e)
pass against the live DB — no cross-module regressions. Seed is idempotent (7 platform templates
+ 20 settings/org incl. the new fallback toggle on first run, 0 templates on re-run). Live
screenshot pending deploy. **Ends at `[CHECKPOINT]`.**

### 10 — Settings, audit, i18n — APPROVED 2026-06-18 (operator instruction)
Approved by the human operator (soft checkpoint). See entry below for build detail.

### 10 — Settings, audit, i18n — DONE (pending approval), 2026-06-18
Three cross-cutting foundations later modules plug into: a per-garage **settings**
store, the system-wide **audit** trail (the garage-facing half + the `audit.record`
convention), and **i18n scaffolding** (English default). GDPR conventions established
first-class.

- **Data model** (`packages/db`): TENANT-SCOPED `Setting` (one row per
  `(orgId, category, key)`, JSON `value`, `updatedBy`; RLS exactly like every tenant
  table). The system-wide `AuditLog` already existed (module 08) with the module-10
  shape — **no new audit table**; this module adds the garage-scoped record/query path
  on top. Migration `20260618200000_settings_audit` (RLS only on `setting`).
- **Source of truth + pure maths** (`packages/db/src/settings.ts`): `SETTINGS_CATALOGUE`
  (20 conservative UK-garage defaults across `general`/`reminders`/`calendar`/`documents`/
  `alerts`/`customer`/`jobs`/`invoicing`/`gdpr`), each declaring type/default/Simple-face/
  bounds. Pure helpers `settingDefault`/`resolveSetting` (the no-row-means-default rule in
  ONE place) + `coerceSettingValue` (the single validator that keeps a stored value
  consistent with its declared type). `settings-provision.ts` (`seedDefaultSettings`,
  edit-preserving). `audit.ts` extended with the GARAGE path: `recordTenantAudit`/
  `queryTenantAudit` (via `withTenant`, RLS enforced — a garage sees ONLY its own rows;
  platform NULL-org entries invisible) + the shared `GARAGE_AUDIT_ACTIONS` vocabulary.
- **API** (`apps/api/src/settings/`): `SettingsService` (tenant-scoped via `withTenant`;
  `get`/`set`/`viewForOrg` — `set` validates, upserts, AND records an audit entry with
  before/after) + `AuditService` (the reusable garage `audit.record` path + the viewer).
  `SettingsController` (`SessionGuard`+`PermissionGuard`): `GET /settings` needs
  `settings.view`, `PUT /settings/:category/:key` needs `settings.edit`, `GET /settings/audit`
  needs `audit.view` — **the API guard, not the UI, is the real gate**. Both services
  EXPORTED so later modules read/write settings + record audited actions through the one
  engine. (DI bug caught by the real boot: `SettingsModule` must import `SuiteModule` —
  `SessionGuard` depends on the suite seam — fixed.)
- **RBAC catalogue extended**: added `audit.view` (Settings group, standalone), granted to
  Owner (`*`) + Manager. Settings dogfoods the existing `settings` mode-registry entry
  (`useMode('settings')`) — no registry change.
- **i18n scaffolding** (`apps/web`): a small, dependency-free layer consistent with the
  project's roll-our-own ethos — `lib/i18n` (`en` complete catalogue + `Messages` shape,
  locale registry, dotted-key `t()` with interpolation, locale-aware Intl date/number/
  **currency** formatters) + `I18nProvider`/`useI18n`/`LocaleSwitcher`. Locale resolution
  per-user (localStorage) → garage default (`general.locale`) → `en`. Adding a language is
  a `Messages`-shaped object + a `LOCALES` entry — no refactor. No hardcoded strings in the
  new UI.
- **Web** (`apps/web`): a `/settings` page that **dogfoods the mode framework** (Simple =
  the everyday handful of toggles; Pro = full categorised tree + search), seeding the
  i18n locale from the garage default, read-gated by `settings.view` and edit-gated by
  `settings.edit`; a `/audit` viewer (permission-gated by `audit.view`) with locale-aware
  timestamps that prove the i18n switch live. Discreet dashboard links; tokens, light/dark,
  skeletons — design bar met. Contract: `packages/ui/SETTINGS_AUDIT_CONTRACT.md`.
- **Seed**: default settings seeded per org (edit-preserving), plus a couple of Org A audit
  entries (a role edit + a settings change) so the live trail has content. Idempotent.

**Gates:** `lint` ✓ (0 warnings) · `typecheck` ✓ (db/types/api/web) · `build` ✓ (`/settings`,
`/audit` routes compile) · `test` ✓ (db 79 unit incl. the 14 pure settings/coercion cases;
api 46 unit; DB-backed suites skip cleanly with no DB). **Verified end-to-end against a real
Postgres** (embedded): all **9 migrations apply with zero drift** (`migrate diff` → "No
difference detected" against a separate shadow DB); app role `rolsuper=false,
rolbypassrls=false`; RLS **enabled on `setting`** and `audit_log`, **OFF** on the platform
`plan`. The new `settings.integration.spec.ts` (6 cases) proves the catalogue seeds
edit-preservingly (idempotent), a setting round-trips, settings are **RLS-isolated** (Org A
invisible to Org B, a cross-tenant write rejected by `WITH CHECK`), and the garage audit path
records + reads + searches under tenant context and **a garage sees ONLY its own rows**
(platform + other-tenant entries invisible). The **HTTP settings e2e** (`settings-http.e2e.spec.ts`,
7 cases) boots the full Nest app and proves: the catalogue-driven view with defaults
(`settings.view`); a save round-trips (`motLeadDays` 30→21) **and appears in the garage's
audit log** with before/after; an out-of-range/wrong-type value 400s; an unknown key 404s;
**tenant isolation** (Org B still sees the default + an empty trail); and the surface is
permission-gated (Receptionist 403 on view, save AND audit). Full db suite (135) + api suite
(90, incl. settings e2e) pass against the live DB — no cross-module regressions. Seed is
idempotent (19 new settings/org first run, 0 on the second). Live screenshot pending deploy.
**Ends at `[CHECKPOINT]`.**

### 09 — White-label — APPROVED 2026-06-18 (operator instruction)
Approved by the human operator (soft checkpoint). See entry below for build detail.

### 09 — White-label — DONE (pending approval), 2026-06-18
Per-garage white-label branding (logo, colours, name, custom domain) on customer-facing
surfaces, **gated by the `whitelabel` entitlement** (module 06). Branding overrides the
design tokens at runtime; a downgrade reverts to GarageAxon defaults WITHOUT data loss.

- **Data model** (`packages/db`): TENANT-SCOPED `Branding` (one per org — display name,
  logo refs, hex palette, custom domain, `enabled`; RLS exactly like every tenant table)
  + PLATFORM-level `StoredObject` (a Phase-1 object-store stand-in: opaque id → content
  type + BYTEA, **no RLS** — outside Postgres isolation like a real bucket, logos public).
  Migration `20260618190000_white_label` (RLS only on `branding`).
- **Source of truth + pure maths** (`packages/db/src/branding.ts`): `DEFAULT_BRAND`
  (mirrors `tokens.css`), `isValidHex`/`hexToRgbChannels`, WCAG `contrastRatio` +
  `contrastWarnings`, `validateLogoUpload` (type/size, 512 KB cap), and the resolver
  `resolveBrandTokens(branding, entitled)` — the ONE place the two locked rules live:
  (1) effective ONLY when entitled AND enabled; (2) fail-safe to the default per-field on
  any missing/invalid value. `branding-provision.ts` (`putObject`/`seedBrandingForOrg`,
  admin client) is the seed/super-admin path.
- **API** (`apps/api/src/branding/`): `BrandingService` (tenant-scoped via `withTenant`;
  view/update/uploadLogo/removeLogo/`resolveEffective`) behind the swappable
  `OBJECT_STORAGE` seam (`DbObjectStorage` in Phase 1, MinIO/S3 later — module-03 pattern).
  `BrandingController` (garage-facing, `SessionGuard`+`PermissionGuard`+`EntitlementGuard`):
  `GET /branding/me` needs only `settings.branding.edit` (so a non-entitled garage still
  sees the upsell); `PUT`/`POST logo`/`DELETE logo` additionally require the `whitelabel`
  entitlement — **the API guard, not the UI, is the real gate**. `PublicBrandingController`
  (UNAUTHENTICATED): `GET /branding/public/:orgId` (effective branding for customer
  surfaces) + `GET /branding/asset/:key` (logo served over the app domain, immutable).
  Logos upload as base64 JSON (no multipart); the JSON body limit was raised to 2 MB.
- **RBAC catalogue extended**: added `settings.branding.edit` (requires `settings.view`),
  granted to Owner (`*`) + Manager. **Mode registry extended**: a `branding` module (both
  faces) so the settings page dogfoods `useMode('branding')`.
- **Web** (`apps/web`): `/branding` settings page (Simple = logo + one colour + name; Pro
  = full palette, dark-mode logo, live **contrast warnings**, custom domain) with a LIVE
  preview that renders the real customer surface (`<BrandPreview>`) via runtime token
  overrides (`brandStyleVars` re-declares the `--brand-*` CSS vars on a scoped wrapper),
  entitlement upsell when not entitled, capability-aware. A public `/b/[orgId]` customer
  confirmation page applies the resolved branding. Discreet dashboard link; tokens,
  light/dark, skeletons — design bar met. Contract: `packages/ui/WHITELABEL_CONTRACT.md`.
- **Seed**: Org A (entitled enterprise) branded ON — teal/amber palette (NOT the default
  orange) + an SVG logo, name "Apex Auto"; Org B left unbranded (starter, no entitlement).
  Idempotent.

**Gates:** `lint` ✓ (0 warnings) · `typecheck` ✓ (db/types/api/web) · `build` ✓ (`/branding`,
`/b/[orgId]` routes compile) · `test` ✓ (db 65 unit incl. the pure resolver/contrast maths;
api 46 unit; DB-backed suites skip cleanly with no DB). **Verified end-to-end against a real
Postgres** (embedded): all **8 migrations apply with zero drift** (`migrate diff` → "No
difference detected"); app role `rolsuper=false, rolbypassrls=false`; RLS **enabled on
`branding`**, **OFF** on the platform `stored_object`. The new `branding.integration.spec.ts`
(6 cases) proves the logo blob round-trips + is readable by the restricted role with no
tenant context, the `Branding` row is **RLS-isolated** (Org A invisible to Org B, a
cross-tenant write rejected by `WITH CHECK`), and the resolver applies branding when
entitled / reverts to defaults when not **with the row retained** (downgrade-safe). The
**HTTP branding e2e** (`branding-http.e2e.spec.ts`, 8 cases) boots the full Nest app and
proves: an entitled garage saves a palette (`#0ea5a4` → `effective.primaryRgb "14 165 164"`),
an invalid hex 400s; a logo uploads, **serves over the app domain** as `image/png`, and
surfaces in effective branding; a non-image 400s; the **public surface shows Org A its brand
and Org B the GarageAxon defaults**; a non-entitled garage reads the page (`entitled:false`)
but is **403 on save** (entitlement gate); the page is **permission-gated** (Receptionist
403); and a **downgrade reverts the public surface to defaults while keeping the saved row**
(re-upgrade restores it). Full db suite (115) + api suite (84, incl. branding e2e) pass
against the live DB — no cross-module regressions. Seed is idempotent on re-run (2 platform
staff + 6 roles/org first run, 0 on the second). Live screenshot pending deploy. **Ends at
`[CHECKPOINT]`.**

### 08 — Super admin — APPROVED 2026-06-18 (operator instruction)
Approved by the human operator (soft checkpoint). See entry below for build detail.

### 08 — Super admin — DONE (pending approval), 2026-06-18
The platform owner's control room (Geek Axon staff) spanning ALL tenants — the one
place with a controlled, audited cross-tenant capability. A SEPARATE auth scope
(never a garage user), an audited admin DB context, and audited support
impersonation (read-only, time-boxed, never silent).

- **Data model** (`packages/db`): PLATFORM-level `PlatformUser` (`platform_role`
  superadmin|support, no RLS — staff identity store) + `ImpersonationSession`
  (audited support sessions, no RLS) + the system-wide append-only `AuditLog`
  (introduced here with the exact module-10 shape: nullable `org_id`, `actorType`,
  JSON `metadata`). `AuditLog` is **RLS-protected** — a garage sees only its own
  entries (platform NULL-org rows invisible), the admin client reads across
  tenants. Migration `20260617180000_super_admin` (new `platform_role`/
  `impersonation_mode`/`audit_actor_type` enums; RLS only on `audit_log`).
- **Platform source of truth + provisioning** (`packages/db/src`): pure
  `platform.ts` (`PLATFORM_CAPABILITIES`, `platformCan` — superadmin=all,
  support=read+impersonate, `AUDIT_ACTIONS`); `audit.ts` (`recordAudit`/`queryAudit`,
  append-only, admin client); `platform-provision.ts` — the controlled cross-tenant
  ops, each AUDITED: `createOrgWithAdmin` (org + default roles + first Owner, ties to
  module-04 seeding), `setOrgStatus`, `adjustOrgCredits` (grant/deduct, never
  negative, before/after), `startImpersonation`/`stopImpersonation` (validates the
  user belongs to the org; idempotent stop), `listOrganisations`/`platformMetrics`/
  `getOrganisationDetail`, `seedPlatformUsers`.
- **API** (`apps/api/src/admin/`): a SEPARATE auth scope — `PlatformAuthService`
  mints a `scope:'platform'` JWT into the dedicated `ax_admin` cookie (a garage
  token has no scope ⇒ rejected; a platform token has no org ⇒ fails the garage
  SessionGuard). `PlatformSessionGuard` (never sets tenant context) +
  `PlatformCapabilityGuard`/`@RequireCapability` (support is 403 on mutating
  actions). `AdminService` runs the one controlled cross-tenant DB context
  (`ADMIN_DB` = admin client, used ONLY here). `AdminController` (`/admin/*`):
  garages list/detail/create/suspend, credit adjust, plans CRUD (create/edit/
  archive) + assign, metrics, health, audit search, impersonation start/stop.
  **Impersonation** mints a
  short-lived garage session token (target user+org + `imp`/`mode` claims), plants
  it in the `ax_session` cookie; the module-03 `SessionGuard` now surfaces the
  impersonation context AND **blocks every mutating request** under a read-only
  session (uniform, app-wide) — read-only, time-boxed, audited, never silent.
- **Web** (`apps/web`): `/admin/login` (platform sign-in, `ax_admin` cookie) + a
  polished single-interface `/admin` console (tabs: Overview KPIs + health + usage
  chart · Garages list/detail with suspend, credit adjust, plan assign, create-org,
  impersonate · Plans (create + Hide/Publish + Archive/Restore) · Audit log search),
  capability-aware affordances. A persistent
  `<ImpersonationBanner>` on the garage dashboard ("viewing as … — read-only", End
  session). Tokens, light/dark, skeletons — design bar met. Discreet "Platform staff"
  link in the home footer.
- **Seed**: 1 super-admin + 1 support platform user (dummy staging creds),
  idempotent. No garage RBAC/mode changes (platform scope is separate).

**Gates:** `lint` ✓ (0 warnings) · `typecheck` ✓ · `build` ✓ (`/admin`, `/admin/login`
routes compile) · `test` ✓ (db 52 unit incl. the pure platform-capability maths; api
46 unit; DB-backed suites skip cleanly with no DB). **Verified end-to-end against a
real Postgres** (embedded PG): all **8 migrations apply with zero drift** (schema-vs-
migrations diff empty on a separate shadow DB); app role confirmed
`rolsuper=false, rolbypassrls=false`; RLS **enabled on `audit_log`**, **OFF** on the
platform `platform_user`/`impersonation_session` tables. The new
`platform.integration.spec.ts` (7 cases) proves `createOrgWithAdmin` seeds 6 roles +
a first Owner + audits `org.create`, suspend/reactivate + credit grant/deduct are
audited before/after (and a deduction can't go negative), an impersonation session is
recorded + stop is audited + idempotent (and a cross-tenant target is rejected), the
**AuditLog is RLS-isolated** (a garage sees only its own rows, a cross-tenant audit
insert is rejected by `WITH CHECK`) while the admin read spans tenants, and the
cross-tenant rollups span every tenant. The **HTTP admin e2e** (`admin-http.e2e.spec.ts`,
7 cases) boots the full Nest app and proves: a super-admin **lists BOTH orgs**
(cross-tenant); a **garage user is DENIED** the admin area; creating an org seeds its
default roles + a working first admin; assigning the **hidden Enterprise plan grants
white-label**; a credit grant is audited; **impersonation** sets the garage cookie,
`GET /auth/me` reports the banner context, a **GET works but a mutating request is 403
(read-only)**, and start/stop are audited; and the **support role is 403 on a credit
adjustment** (capability gate). Full db suite (96) + api suite (75, all
rbac/modes/plans/credits/admin e2e) pass against the live DB — no cross-module
regressions. Seed seeds the platform staff + is idempotent on re-run (2 new staff
first run, 0 on the second). Live screenshot pending deploy. **Ends at `[CHECKPOINT]`.**

### 07 — Credits — APPROVED 2026-06-17 (operator instruction)
Approved by the human operator (soft checkpoint). See entry below for build detail.

### 07 — Credits — DONE (pending approval), 2026-06-17
A single unified credit balance per org, a configurable per-action price list,
credit packages + pay-as-you-go top-ups, and a mandatory **pre-flight check before
any metered call** — the margin guard every paid third-party action goes through.
Free actions (DVLA/DVSA) are never metered; GBP diagnostic credits stay separate.

- **Source of truth + pure maths** (`packages/db/src/credits.ts`): the
  `FREE_ACTION_ALLOWLIST` (`dvla.lookup`/`dvsa.mot` — never metered), a seedable
  `CREDIT_PRICE_LIST` (sms=1/email=2/whatsapp=3/voip=2/ai=5) + `CREDIT_PACKAGES`
  (1k/£100 + 5k/£450 bulk), and the deterministic helpers `costForAction`,
  `resolvePreflight` (free ⇒ allowed cost 0; unpriced non-free ⇒ BLOCKED fail-closed;
  else balance ≥ cost), `creditsForTopUp` (PAYG 10p/credit), `vatMinor`/`grossMinor`
  (20% VAT on sales) and `isLowBalance`. Prices in MINOR units (pence).
- **Concurrency-safe ledger** (`packages/db/src/credit-ledger.ts`): `chargeCredits`
  is a single conditional `UPDATE … WHERE balance >= cost` inside the tenant tx —
  the row lock serialises parallel charges so the balance **can never go negative**;
  the usage row is appended in the SAME tx (balance + ledger never drift).
  `creditCredits` (runtime top-up), `getBalance`, `recentTransactions`, `usageByAction`.
- **Data model** (`packages/db`): PLATFORM-level `CreditPriceList` + `CreditPackage`
  (no RLS — shared catalogue the super-admin tunes at runtime; package reuses the
  module-06 `PlanVisibility`/`PlanStatus` enums) + TENANT-SCOPED `CreditBalance`
  (one unified balance/org) and append-only `CreditTransaction` ledger (new
  `CreditTxnKind` enum: usage/purchase/grant). RLS-protected exactly like every
  tenant table (migration `20260617170000_credits_metering`). `seedCreditPriceList`/
  `seedCreditPackages` (edit-preserving) + `grantCredits` (admin/bypass — the seed +
  included-allowance path) in `credit-provision.ts`.
- **API** (`apps/api/src/credits/`): `MeteringService` — THE gatekeeper later
  modules reuse (`preflight` before, `charge` after; `purchase` mock until payments;
  `viewForOrg`; `proofMeter`). Exported from `CreditsModule`. `CreditsController`
  behind `SessionGuard` + `PermissionGuard`: `GET /credits/me` + `/balance` +
  `/preflight/:actionKey` (`credits.view`), `POST /credits/purchase` (`credits.purchase`),
  and the proof `POST /credits/proof/meter`.
- **Separate GBP credits**: extended the module-03 suite seam with
  `getDiagnosticCredits` (mock impl) — the hub's own diagnostic balance, surfaced
  ALONGSIDE the unified balance and never merged.
- **RBAC catalogue extended** (`permissions.ts`): added `credits.view` +
  `credits.purchase` (purchase → view dependency), granted to Owner (`*`) + Manager.
- **Mode registry extended** (`modes.ts`): added a `credits` module (both faces).
- **Web** (`apps/web`): a `/credits` page that **dogfoods the mode framework**
  (`useMode('credits')`): Simple = "Credits left: N" + Buy more + plain-language
  recent usage; Pro = animated usage-by-action chart, price list (with a per-action
  "Meter once" proof), packages (VAT-inclusive), full ledger. GBP credits shown
  separately in both. Gated by `credits.view`, linked from the dashboard.
- **Metering contract documented** for every later paid-action module:
  `packages/ui/METERING_CONTRACT.md` (the two locked rules — metered IFF it costs
  us; pre-flight before / charge after / fail-closed — and the `preflight`→action→
  `charge` recipe; composes with permissions + entitlements).

**Gates:** `lint` ✓ (0 warnings) · `typecheck` ✓ · `build` ✓ (`/credits` route
compiles) · `test` ✓ (db 49 unit incl. the pure credit maths; api 44 unit; DB-backed
suites skip cleanly with no DB). **Verified end-to-end against a real Postgres**
(embedded): all **6 migrations apply with zero drift** (schema-vs-db diff empty); app
role confirmed `rolsuper=false, rolbypassrls=false`; RLS enabled on `credit_balance`
+ `credit_transaction`, **OFF** on the global `credit_price_list`/`credit_package`.
The new `credits.integration.spec.ts` (7 cases) proves the catalogue seeds, a charge
decrements + logs a usage row, an insufficient balance is blocked without going
negative, **25 parallel 1-credit charges against a 10-credit balance succeed exactly
10 times (never negative, exactly 10 usage rows)**, a runtime top-up credits + logs,
tenant isolation, and a cross-tenant ledger write rejected by `WITH CHECK`. The
**HTTP credits e2e** (`credits-http.e2e.spec.ts`) boots the full Nest app and proves:
a metered action does preflight then charge and decrements (whatsapp 3 → balance
97); an insufficient balance blocks (Org B at 0 → `allowed:false`, no charge); a
**DVLA-allowlisted action is free and never charged**; buying a package tops up (with
VAT); GBP diagnostic credits are shown SEPARATELY; and the surface is `credits.view`-
gated (Owner ok / Receptionist 403). Full db suite (87) + api suite (66, all
rbac/modes/plans/credits e2e) pass against the live DB — no cross-module regressions.
Seed seeds the credit catalogue + grants Org A 500 credits (Org B left at 0 for the
block proof), idempotent on re-run. Live screenshot pending deploy. **Ends at `[CHECKPOINT]`.**

### 06 — Plans & entitlements — APPROVED 2026-06-17 (operator instruction)
Approved by the human operator (soft checkpoint). See entry below for build detail.

### 06 — Plans & entitlements — DONE (pending approval), 2026-06-17
Dynamic subscription plans, a per-org resolved **entitlement** that gates features
across the whole system, and a **public pricing website** — billing of record stays
with the GBP hub, read through the module-03 suite-account seam.

- **Source of truth + resolver** (`packages/db/src/entitlements.ts`): canonical
  `FEATURE_FLAG_KEYS` / `LIMIT_KEYS`, a seedable `PLAN_CATALOGUE` (3 public tiers
  Starter/Standard/Pro + 1 hidden Enterprise), and the pure helpers
  `resolveEffectiveEntitlement` (plan flags ∪ add-on flags; add-ons only ever
  grant), `hasFlag`/`getLimit`/`isWithinLimit` (fail-closed; null limit = unlimited)
  and `planSlugForCrmPlan`. Prices stored in MINOR units (pence).
- **Data model** (`packages/db`): PLATFORM-level `Plan` (no RLS — shared catalogue
  the super-admin edits at runtime; `featureFlags`/`limits` JSON so new capabilities
  need no schema change) + TENANT-SCOPED `Entitlement` (one per org, cached
  `effectiveFlags`/`effectiveLimits`) and `PlanAssignment` (audit log). RLS-protected
  exactly like every tenant table (migration `20260617160000_plans_entitlements`;
  reuses new `PlanVisibility`/`PlanStatus`/`EntitlementStatus` enums).
  `seedPlans` (edit-preserving — only creates missing slugs) + `assignPlanToOrg`
  (the super-admin/system path: upserts the entitlement, recomputes effective
  flags/limits, logs the change) are the reusable provisioning hooks (`plan-provision.ts`,
  admin/bypass client — the same controlled exception as seed).
- **API** (`apps/api/src/plans/`): `PlansService` (public-plan reads — global, no
  tenant scope) + `EntitlementsService` (tenant-scoped via `withTenant`:
  `resolveForOrg` — super-admin assignment if present, else **lazy resolution from
  the suite interface**, cached + logged — plus `can`/`getLimit`/`assertWithinLimit`
  and the garage `viewForOrg`). `EntitlementGuard` + `@RequireEntitlement(flag)` is
  THE reusable feature gate later modules reuse. Controllers: `GET /plans/public`
  (UNAUTHENTICATED — drives the pricing site, hides hidden plans), `GET /entitlements/me`
  (gated by new `billing.view`), and proof endpoints `GET /entitlements/proof/whitelabel`
  (entitlement-gated) + `POST /entitlements/proof/seat` (seat-limit enforced).
- **RBAC catalogue extended** (`permissions.ts`): added `billing.view`, granted to
  Owner (`*`) + Manager — proving cross-module catalogue extensibility again.
- **Web** (`apps/web`): a public `/pricing` page (live public plans, monthly/annual
  toggle, "Contact us" path, branded, responsive, light/dark, skeletons) linked from
  the home header; a `/plan` page that **dogfoods the mode framework** (`useMode('settings')`:
  Simple = "Your plan: X — Upgrade", Pro = feature/limit breakdown + live usage),
  gated by `billing.view` and linked from the dashboard; `EntitlementsProvider` +
  `useEntitlement(flag)` + `<Entitled>` (the web gating contract, mirrors `<Can>`).
- **Entitlement contract documented** for every later gated module:
  `packages/ui/ENTITLEMENTS_CONTRACT.md` (the two locked rules — entitlements gate
  WHETHER the plan includes a feature, permissions gate WHO; fail-closed — the model,
  and the `@RequireEntitlement` / `<Entitled>` recipe).

**Gates:** `lint` ✓ (0 warnings) · `typecheck` ✓ · `build` ✓ (`/pricing`, `/plan`
routes compile) · `test` ✓ (db 38 unit incl. the pure entitlement maths; api 44 unit;
DB-backed suites skip cleanly with no DB). **Verified end-to-end against a real
Postgres** (embedded): all **5 migrations apply with zero drift** (schema-vs-db diff
empty); app role confirmed `rolsuper=false, rolbypassrls=false`; RLS enabled on
`entitlement` + `plan_assignment`, **OFF** on the global `plan` table. The new
`entitlements.integration.spec.ts` proves the plan catalogue seeds, a HIDDEN plan
assignment grants white-label, the Starter plan leaves it off with a seat limit of 2,
an add-on only ever grants, tenant isolation (Org A's entitlement invisible to Org B),
a cross-tenant write rejected by `WITH CHECK`, and an audit row per (re)assignment.
The **HTTP plans e2e** (`plans-http.e2e.spec.ts`) boots the full Nest app and proves:
`GET /plans/public` (no auth) returns the 3 public plans and **hides Enterprise**;
Org A (hidden Enterprise) sees white-label ON; the white-label gate is **200 for Org A
/ 403 for Org B**; the seat limit is enforced (**Org B at its 2-seat limit → 403**,
Org A → 200); the plan view is `billing.view`-gated (Owner ok / Receptionist 403); and
an org with no assignment **resolves its entitlement live from the suite hub** (caches
a `suite`-sourced row). All three HTTP e2e suites (rbac/modes/plans) now run serially
(`maxWorkers: 1`) so they don't clobber the shared DB. Seed seeds the catalogue + a
proof config (Org A → Enterprise, Org B → Starter), idempotent on re-run
(0 new plans/roles second time). Live screenshot pending deploy. **Ends at `[CHECKPOINT]`.**

### 05 — Mode framework (Simple/Pro) — APPROVED 2026-06-17 (operator instruction)
Approved by the human operator (soft checkpoint). See entry below for build detail.

### 05 — Mode framework (Simple/Pro) — DONE (pending approval), 2026-06-17
The system-wide progressive-disclosure framework every later UI module builds
its two faces on: a per-garage, per-module (and global) Simple↔Pro choice that is
**lossless** (full data always stored regardless of mode) and **role-aware**
(`effective = nature-clamp(min(garageMode, roleCeiling))`).

- **Source of truth + resolver** (`packages/db/src/modes.ts`): `MODULE_REGISTRY`
  (12 modules, each declaring `supportsSimple`/`supportsPro`/`defaultMode` — incl.
  the spec's single-face cases: `workshop` Simple-only, `accounting` Pro-only) and
  the pure helpers `resolveGarageMode` (override → global → registry default),
  `resolveEffectiveMode` (nature clamp then `min(garage, ceiling)`) and
  `effectiveCeiling` (max across a user's roles; `simple` if none). Mirrored
  client-side (`apps/web/lib/modes.ts`) so the web computes the same answer the
  API does.
- **Data model** (`packages/db`): GLOBAL `ModuleRegistry` (no RLS — shared
  reference data, seeded from code via `seedModuleRegistry`) + TENANT-SCOPED
  `GarageModeSetting` (`@@unique([orgId, moduleKey])`, null `moduleKey` = the
  garage's global default row, set = a per-module override; absence ⇒ registry
  default). RLS-protected exactly like every tenant table (migration
  `20260617150000_mode_framework`). Reuses the new `GarageMode` enum.
- **API** (`apps/api/src/modes/`): `ModesService` (tenant-scoped via `withTenant`;
  `context`, `getEffectiveMode`, `setGlobalMode`, `setModuleMode`,
  `clearModuleMode` — single-face modes refused with 400) + `ModesController` —
  `GET /modes` (open to any session — drives `useMode`), `GET /modes/me/:key`
  (authoritative effective mode), `PUT /modes`, `PUT/DELETE /modes/:key` (all
  guarded by the new `settings.mode.edit` permission via the module-04
  `PermissionGuard`). All behind `SessionGuard` → RLS.
- **RBAC catalogue extended** (module-04 `permissions.ts`): added
  `settings.mode.edit` (requires `settings.view`), granted to Owner (`*`) and
  Manager — proving the catalogue is extensible across modules without a data
  model change.
- **Web** (`apps/web`): `ModeProvider` + `useMode(moduleKey)` (THE contract hook)
  + a `/modes` settings page that **dogfoods the framework** (it picks its own
  face with `useMode('settings')`): a Simple face (one garage-wide "Keep it
  simple / Show advanced features" switch) and a Pro face (per-module matrix with
  nature-locked single-face modules, per-module override + "use default", the
  global default, and a live "you see: …" effective-mode hint). Dashboard gains a
  `<Can settings.mode.edit>` link. Tokens, light/dark, skeletons — design bar met.
- **Mode contract documented** for every later module:
  `packages/ui/MODE_CONTRACT.md` (the three locked rules, the effective-mode
  maths, the data model, and the "register → two faces → `useMode` → never gate
  access by mode" recipe).

**Gates:** `lint` ✓ (0 warnings) · `typecheck` ✓ · `build` ✓ · `test` ✓ (db 24
unit pass incl. the pure mode maths; api 44 unit pass; DB-backed suites skip
cleanly with no DB). **Verified end-to-end against a real Postgres** (embedded):
all 4 migrations apply with **zero drift** (schema-vs-db diff empty); app role
confirmed `rolsuper=false, rolbypassrls=false`; RLS enabled on
`garage_mode_setting`, OFF on the global `module_registry`. The new
`modes.integration.spec.ts` proves override > global > registry-default
precedence, the no-row fallback (Org B), tenant isolation (Org A's choices
invisible to Org B), a cross-tenant write rejected by `WITH CHECK`, the
Simple-ceiling clamp, single-face nature, and a **lossless** toggle (a
`tenant_demo` row survives simple→pro→simple). The **HTTP mode e2e**
(`modes-http.e2e.spec.ts`) boots the full Nest app and proves login → cookie →
`useMode`: Owner ceiling `pro` / Receptionist `simple`; garage-default write
gated (Owner 204 / Receptionist 403); the **Receptionist held to Simple on a
Pro garage** (`garageMode=pro`, `mode=simple`); per-module override wins;
`accounting` always Pro / `workshop` always Simple; a Pro setting on a Simple-only
module 400s; session-driven tenant isolation. Both DB-backed suites run in CI's
`rls` job (db `npm test` + api e2e) against the same Postgres. Seed now seeds the
registry + a proof config ("Org A: Simple everywhere except Inventory = Pro"; Org
B left on registry defaults), idempotent on re-run. Live screenshot pending
deploy. **Ends at `[CHECKPOINT]`.**

### 04 — Roles & permissions — APPROVED 2026-06-17 (operator instruction)
Approved by the human operator (soft checkpoint). See entry below for build detail.

### 04 — Roles & permissions — DONE (pending approval), 2026-06-17
Fully dynamic, per-tenant RBAC: a global permission catalogue + dependency DAG,
tenant-local editable roles seeded from templates, dependency-closure enforcement,
an API permission guard, and Simple/Pro faces over the same data.

- **Catalogue (source of truth)** (`packages/db/src/permissions.ts`):
  `PERMISSION_CATALOGUE` (40 keys across 12 groups), `PERMISSION_DEPENDENCIES`
  (the DAG — incl. the transitive chain `role.delete → role.edit → role.view` and
  the multi-parent `user.assign_role → user.view & role.view`), six
  `ROLE_TEMPLATES` (Owner/Manager/Receptionist/Technician/Driver/Accountant with
  `modeCeiling`), and `expandPermissions` (transitive closure, cycle-safe, sorted).
  The DB tables are seeded FROM this code; the API serves it to the web.
- **Data model** (`packages/db`): GLOBAL `Permission` + `PermissionDependency`
  (no RLS — shared reference data); TENANT-SCOPED `Role` (`@@unique([orgId,name])`,
  `isSystemDefault`, `templateKey`, `modeCeiling` enum), `RolePermission` (stores
  the ALREADY-expanded closure → O(1) checks) and `UserRole` — each carrying
  `org_id` and RLS-protected exactly like every tenant table (migration
  `20260617140000_roles_permissions`). `provisionOrgRoles` / `seedPermissionCatalogue`
  / `assignRoleToUser` (`rbac-provision.ts`) are the reusable org-creation hooks
  (admin/bypass client — the same controlled provisioning exception as seed).
  Seed now provisions the 6 roles per org + an Owner and a **Receptionist** user
  (Simple, deliberately no `customer.delete`) for the live proof.
- **API** (`apps/api/src/roles/`): `RolesService` (tenant-scoped via `withTenant`;
  create/update/duplicate/delete with closure expansion, prerequisite-stranding
  prevented structurally, system-default + in-use-role delete guards, effective-
  permission union), `@RequirePermission` + `PermissionGuard` (fail-closed 403),
  and `RolesController` — `GET /roles`, `GET /roles/catalogue`,
  `GET /roles/me/permissions`, `POST/PATCH/DELETE /roles[...]`, assignment
  endpoints, and `POST /roles/proof/customer-delete` (guarded by `customer.delete`,
  the headline proof). All behind `SessionGuard` → RLS.
- **Web** (`apps/web`): `PermissionsProvider` + `usePermission` + `<Can>`; a `/team`
  page with a Simple face (plain-language roles + people/role assignment) and a Pro
  face (permission matrix grouped by module with client-side dependency
  auto-tick/lock, create/duplicate/delete custom roles, `modeCeiling`, multi-role
  assignment) over the same data; dashboard gains a `<Can role.view>` Team link and
  a `<Can customer.delete>` permission-proof affordance. Tokens, light/dark,
  skeletons — design bar met.
- **Latent module-03 bugs fixed** (the auth HTTP stack had never actually booted —
  module 03 only tested the provider class directly): (1) `LoginRateLimitGuard`
  was registered as a provider but has plain-number constructor args, so DI could
  not resolve it and `AppModule` failed to instantiate → now applied as an
  instance (`@UseGuards(new LoginRateLimitGuard())`) and dropped from providers;
  (2) confirmed `SessionGuard`'s request-scoped `TenantContext` injection works
  under the real (metadata-emitting) build. Both proven by the new HTTP e2e.

**Gates:** `lint` ✓ (0 warnings) · `typecheck` ✓ · `build` ✓ · `test` ✓ (api 44
unit pass + 4 HTTP-e2e skip cleanly with no DB; db 14 catalogue/closure pass, RLS
+ roles integration skip). **Verified end-to-end against a real Postgres**
(embedded): all 3 migrations apply with zero drift; app role confirmed
`rolsuper=false, rolbypassrls=false`; RLS enabled on `role`/`role_permission`/
`user_role`, OFF on the global catalogue; no-context → 0 rows (fail-closed); Org A
sees its 6 roles and 0 of Org B's; **Owner has `customer.delete`, Receptionist does
not, and the Receptionist's closure pulled in `invoice.view` via `payment.take`**;
cross-tenant role-permission write rejected (Postgres 42501); editing Org A's
Receptionist leaves Org B's copy untouched. The **HTTP RBAC e2e** boots the full
Nest app and proves login → cookie → guard: Owner 200 / Receptionist 403 on the
guarded endpoint, role admin gated, and session-driven tenant isolation. New CI
step runs this e2e against the `rls` job's Postgres. Staging deploy seeds the roles
+ proof users; live screenshot pending deploy. **Ends at `[CHECKPOINT]`.**

### 03 — Auth + suite interface — APPROVED 2026-06-17 (operator instruction)
Approved by the human operator (FIXED checkpoint). See entry below for build detail.

### 03 — Auth + suite interface — DONE (pending approval), 2026-06-17 🛑 FIXED CHECKPOINT
Authentication + the **GarageBrainPro (GBP) suite-account seam**, with the hub
**mocked** in Phase 1 behind a swappable interface. Real auth now drives the
step-02 tenant context (RLS), replacing the temporary `x-org-id` header.

- **Swap point** (`packages/types/src/suite.ts`): `SuiteAccountProvider`
  interface (`authenticate` · `verifyToken` · `getEntitlements` · `getIdentity`
  · `issueSsoToken` · `verifySsoToken`) + its data shapes. The rest of
  GarageAxon depends ONLY on this interface (injected via the
  `SUITE_ACCOUNT_PROVIDER` token) — never on a concrete implementation. Phase 2
  swaps the single binding for `GbpSuiteAccountProvider` with no rewrite.
- **Phase-1 implementation** (`apps/api/src/suite/`): `MockSuiteAccountProvider`
  — local email/password against the identity mirror, signed-JWT session + SSO
  tokens, static mock entitlements (pro plan + add-ons). Behind the
  `IdentityStore` abstraction; `PrismaIdentityStore` uses the admin/bypass
  client for the one legitimately cross-tenant op — resolving an identity at
  login, before any org context exists (same controlled exception as seed).
  bcrypt (pure-JS `bcryptjs`, no native build) at work factor 12; secrets from
  env only (`JWT_SECRET`; prod refuses to boot without it, dev has an insecure
  fallback).
- **Data model** (`packages/db`): `User` local mirror — `app_user` table,
  `@@unique([orgId, email])`, RLS-protected exactly like every tenant table
  (migration `20260617130000_auth_users`). Seed provisions an owner user per org
  (dummy staging creds, hashed).
- **Auth surface** (`apps/api/src/auth/`): `POST /auth/login` (HTTP-only +
  SameSite=Lax + Secure-in-prod cookie; opaque "Email or password is
  incorrect"; in-memory login rate-limit), `POST /auth/logout`, `GET /auth/me`,
  `GET /auth/entitlements`, `POST /auth/sso/issue` + `GET /auth/sso/callback`
  (intra-app round-trip), and `GET /auth/me/workspace` (RLS-through-real-auth
  proof). `SessionGuard` reads the cookie → `verifyToken` → stamps
  `TenantContext.orgId` (driving RLS); fail-closed on missing/invalid token.
  `main.ts` adds `cookie-parser` + credentialed CORS (`WEB_ORIGIN`).
- **Web** (`apps/web`): universal `/login` (token-driven, light/dark, skeleton
  on submit, plain-language errors, large touch targets) + `/dashboard`
  (identity, entitlements, SSO round-trip, logout); cookie-only session via
  `lib/api.ts` (`credentials: 'include'`, no token in JS/localStorage).

**Gates:** `lint` ✓ (0 warnings) · `typecheck` ✓ · `build` ✓ · `test` ✓ (api 39
pass; db RLS suite skips cleanly with no DB). **Verified end-to-end against a
real Postgres** (embedded): migrations apply with zero drift; app role confirmed
`rolsuper=false, rolbypassrls=false`; RLS enabled on `app_user`; the 9-case RLS
suite (now incl. `app_user` isolation) passes; and the REAL
`MockSuiteAccountProvider` + `PrismaIdentityStore` proved login resolves the
correct org, the session token round-trips, wrong passwords are rejected, **Org
A login cannot reach Org B data/users via RLS**, SSO round-trips, and
entitlements read back. Staging deploy seeds owner users; live login + isolation
screenshot pending deploy. **Ends at `[FIXED_CHECKPOINT]`.**

### 02 — Tenancy / RLS — APPROVED 2026-06-17 (operator instruction)
Approved by the human operator. See entry below for build detail.

### 02 — Tenancy / RLS — DONE (pending approval), 2026-06-17 🛑 FIXED CHECKPOINT
Multi-tenant data foundation with **Postgres Row-Level Security** as the
isolation guarantee (app-layer scoping is defence-in-depth on top).

- **Data model** (`packages/db/prisma/schema.prisma`): `Organisation` (tenant
  root, lean), `Branch` (`@@unique([orgId, name])`), and `TenantDemo` (throwaway
  RLS proof table). Tenant tables carry `org_id`; convention established for all
  later modules. Migration `20260617120000_tenancy_rls` ships the DDL + RLS.
- **RLS**: every tenant table has `ENABLE ROW LEVEL SECURITY` + a `FOR ALL`
  policy `USING/WITH CHECK (org_id = current_setting('app.current_org_id', true))`
  (org table keys on `id`). `missing_ok=true` → no context yields zero rows
  (fail-closed). RLS not forced, so the owner/admin role keeps the migration +
  seed bypass the spec allows.
- **Two-role split** (`datasource.directUrl`): app connects as the RESTRICTED,
  RLS-subject role (`DATABASE_URL`); migrations/seed use the admin/owner role
  (`DATABASE_MIGRATION_URL`). Restricted role created by
  `packages/db/sql/bootstrap-roles.sql` (idempotent, `\gexec`).
- **Tenant context**: `withTenant(orgId, fn)` (`packages/db/src/tenant.ts`)
  opens a tx and stamps `app.current_org_id` transaction-locally via bound-param
  `set_config` (pooling-safe, injection-safe). NestJS `TenancyModule`:
  request-scoped `TenantContext`, fail-closed `TenantGuard` (temp `x-org-id`
  header until auth in 03), and staging-only `/tenancy/{whoami,demo}` proof
  endpoints gated by `StagingProofGuard` (`APP_ENV=staging` + `SCREENSHOT_TOKEN`).
- **Proof**: 8-case RLS integration test (`packages/db/src/rls.integration.spec.ts`)
  — per-tenant read, cross-tenant read/update/delete blocked, `WITH CHECK`
  INSERT rejected, org root isolated, fail-closed (no context → empty; empty
  orgId → throws). Self-skips with no DB; new CI `rls` job runs it for real
  against a Postgres 16 service. Seed (`prisma/seed.ts`) provisions Org A/Org B.

**Gates:** `lint` ✓ · `typecheck` ✓ · `build` ✓ · `test` ✓ (api 11 pass; db RLS
suite skips cleanly with no DB). **RLS verified end-to-end against a real
Postgres** (embedded): app role confirmed `rolsuper=false, rolbypassrls=false`,
RLS enabled on all 3 tables, migration applied with zero drift, seed ran via
admin bypass, all 8 isolation tests pass. Staging deploy seeds Org A/B; live
proof via `/tenancy/demo` (token-gated). **Ends at `[FIXED_CHECKPOINT]`.**

### 01 — Foundation — APPROVED 2026-06-17 (operator instruction)
Soft checkpoint approved by the human operator. See entry below for build detail.

### 01 — Foundation — DONE (pending approval), 2026-06-17
Turborepo monorepo stood up: `apps/api` (NestJS, `GET /health` → `{status:'ok', build}`,
no `/api` prefix), `apps/web` (Next.js 14 App Router, themed GarageAxon placeholder
with light/dark toggle + skeleton-loader demo + status palette, tokens only),
`packages/ui` (design tokens CSS + Tailwind preset + ThemeProvider + Button/Card/
Skeleton/ChartContainer), `packages/config` (tsconfig base, ESLint flat config,
Tailwind preset), `packages/types` (HealthResponse contract), `packages/db`
(Prisma/Postgres baseline + generate wired). CI workflow
(install→generate→lint→typecheck→build→test), `deploy.sh` (node-detection,
dev-deps install, prisma generate, migrations, fresh BUILD_ID, PM2 restart,
`/health` polling) and `ecosystem.config.cjs` added.

**Gates:** `npm ci --include=dev` ✓ · `npm run generate` ✓ · `lint` ✓ (0 warnings) ·
`typecheck` ✓ · `build` ✓ · `test` ✓. API `/health` smoke-tested (returns ok +
injected BUILD_ID); web placeholder smoke-tested (renders wordmark/tagline, token
classes). Live staging screenshot pending deploy.
