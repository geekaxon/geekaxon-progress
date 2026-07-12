# PROGRESS.md — Rihaish

**Current build-step:** 10 — storage-media
**Last completed:** 09 — user-account — DONE (2026-07-12)
**Next:** 10 — storage-media

### Recent steps

- **09 — user-account** — DONE (2026-07-12) — `core.account` registered (isCore, deps core.auth/core.forms). Schema (migration `20260712130000_user_account`): `UserProfile`, `NotificationPreference` (both keyed by `userId`, not tenant-scoped), `Society.cnicCapture`, `Session.ip`/`userAgent` (login now captures both). `lib/crypto.ts` AES-256-GCM at-rest (`ENCRYPTION_KEY`→sha-256 key, `v1:iv:tag:ct`, tamper→null) + `maskCnic` (`*****-****567-*`). `lib/account/*`: pure `notifications-policy` (in-app locked all categories, push locked for emergency), `service` (profile get/update with CNIC encrypt+re-auth+audit, changePassword→revoke other sessions, setGuardPin, sessions list/revoke/revoke-all, notification prefs get/put with availability+floor), `actor` (society-user-only guard). API `/api/me` GET/PATCH, `/password`, `/pin`, `/sessions` GET+DELETE-all, `/sessions/[id]` DELETE, `/notification-preferences` GET/PUT. UI `/app/account` (Simple/Pro): Profile (masked CNIC, re-auth to edit, PII hidden on shared device), Security (password/PIN + sessions list→table kit in Pro), Notifications matrix (locked switches), Preferences (lang/theme). i18n en+ur. Gates green: typecheck, lint(0), 222 unit tests, build (account EN+UR); runtime smoke — all `/api/me/*` 401 unauth, app-shell 404 off society host. e2e `account.spec` (unauth 401s + seeded session-revoke, skips unseeded).
- **08 — platform-console** — DONE (2026-07-12) — `platform.console` registered (platform-only, `isCore:false`, deps core.entitlements/tables/shell). `lib/platform/*` guard/list/societies/detail/plans/audit/dashboard + impersonation (apex-only, 30-min read-only, enable-write w/ reason, exit). API societies/entitlements/impersonate/plans/audit; neutral impersonate exit/write. UI platform shell + dashboard/societies/detail/plans/audit tables on the kits. i18n en+ur. Gates green (207 unit tests).
- **07 — data-table-export-kit** — DONE (2026-07-12) — `core.tables` registered (isCore, deps core.forms). Server contract `lib/tables/*`: `buildListQuery`/`runList` (society-scoped, pageSize cap 100), in-memory parity, export engine (CSV+guard, exceljs RTL, pdf-lib Urdu, PII→audit). `<DataTable>` (TanStack v8) full toolbar/states/RTL + chart primitives. Dev `/design-system/tables`. All gates green (197 unit tests).

> Full history: PROGRESS-HISTORY.md · Build order: ARCHITECTURE.md
