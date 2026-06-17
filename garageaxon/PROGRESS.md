# GarageAxon — PROGRESS.md

> **Current state tracker.** The agent reads this with `ARCHITECTURE.md` and `AGENT.md` at the start of every session and **updates it after every module/task**.
>
> **PUBLIC REPO WARNING:** pushed to a public repo — **NO secrets, credentials, tokens, or sensitive data, ever.**

---

## Status

- **Project:** GarageAxon (Geek Axon Ltd garage software family; sibling to GarageBrainPro)
- **Phase:** Phase 1 — not started
- **Current module:** none (awaiting Build Order step 1)
- **Last completed module:** none
- **Last deploy:** none
- **Build state:** initialised (spec only)
- **Staging:** garageaxon.geekaxon.com
- **Production:** garageaxon.com (human-gated)
- **Checkpoint policy:** maximum automation. Hard manual stops at **steps 2, 3, 22** only; all other modules soft-checkpoint (30s auto-approve); steps 28 & 30 are human-required infra hand-offs.

---

## Build-order checklist

Legend: `[ ]` not started · `[~]` in progress · `[x]` done
Checkpoint markers (maximum automation — hard-stop only where the operator is genuinely needed):
- 🛑 **HARD** `[FIXED_CHECKPOINT]` — agent stops, waits for your manual APPROVE.
- 🟢 **SOFT** `[CHECKPOINT]` — auto-approved after 30s; build continues automatically.
- ✋ **HUMAN-REQUIRED** `[HUMAN_REQUIRED]` — not an approve gate; needs an infra action only you can do (DNS/SSL, telephony server).

### Phase 1
- [ ] 1. Repo + Turborepo + NestJS/Next scaffolding + CI + deploy.sh + design tokens/theming
- [ ] 2. Database + Prisma baseline + multi-tenant (Org/Branch) + isolation guards  🛑 HARD
- [ ] 3. Auth + suite-account interface (mock hub) + SSO scaffolding  🛑 HARD
- [ ] 4. Dynamic Roles & Permissions (dependency graph, tenant-scoped default edits)
- [ ] 5. Simple↔Pro mode framework (per-module + global, role-aware) + UI contract
- [ ] 6. Dynamic Plans + Entitlements + public pricing site + hidden plans
- [ ] 7. Single unified Credits/metering engine (price list, packages, top-ups, pre-flight, free-service exclusion)
- [ ] 8. Super-admin console (garages, plans, billing, usage, health, audit, impersonation)
- [ ] 9. White-label branding (plan-gated runtime theming)
- [ ] 10. Settings + Audit log + i18n scaffolding
- [ ] 11. Notifications hub + per-garage sender identity + send metering
- [ ] 12. Customers (account controls, 3 consent streams, groups, priority message) + search
- [ ] 13. Vehicles + live DVLA (per-job, mismatch alert) + DVSA cache (free) + notification-preference resolver
- [ ] 14. New Job flow (reg-first) + Job entity + status pipeline + JobStatusEvent
- [ ] 15. Workshop board (realtime status buckets) + Technician PWA (queue, clocking, parts request, advisories, scan upload)
- [ ] 16. QR drop-off + signed T&C consent (realtime) + Vehicle Condition Report (damage markers, photos, dual signature)
- [ ] 17. EVHC / Service Schedule (RAG, measurements, photos) + customer online authorisation + Daily job sheet (tech/branch/garage)
- [ ] 18. Estimates (+ read/authorise tracking, convert-to-job) + Follow-Up/Task engine
- [ ] 19. Products (2-level cats, cost/sale, EAN, cross-ref, add-to-every-job) + Inventory (stock, locations, min/max, inter-branch transfer) + Parts-request flow + Parts-Only/counter sale
- [ ] 20. Suppliers + Purchase Invoices + supplier ledger/balances + returns
- [ ] 21. Accounting engine (double-entry, CoA, journals) + Simple/Full views + UK VAT + deferred revenue
- [ ] 22. Invoicing (+ insurance-excess split) + Payments (Stripe, PayPal, GoCardless, Dojo, Payment Assist + manual) → post to ledger  🛑 HARD
- [ ] 23. Plans & Coverage engine (items, primitives → packages/memberships, coverage-resolution, warranties)
- [ ] 24. Courtesy Cars (+ fuel records) + Collection & Delivery (drivers, parking info, photos, pre-call)
- [ ] 25. Staff (attendance, holidays, notes, productivity) + Assets & tools (calibration, assignment, depreciation)
- [ ] 26. Job P&L + KPI dashboard + report suite + exports
- [ ] 27. Customer app (PWA): bookings, approve/decline, invoices, wallet, packages
- [ ] 28. Smart Booking: JS embed widget + hosted page + microsite (custom domain + auto-SSL) + booking API  ✋ HUMAN-REQUIRED (DNS/SSL)
- [ ] 29. WordPress plugin (separate sub-project; WordPress.org-ready + direct download; garage-key auth) — directory submission human-gated
- [ ] 30. Call-center app-side: DID→tenant routing + screen-pop (AMI-bridge + WebSocket) + WebRTC click-to-call + recording link + book-from-call + missed-call recovery + auto-lead (needs human FreePBX runbook first)  ✋ HUMAN-REQUIRED (FreePBX server first)

### Phase 2
- [ ] 31. GBP real integration (replace mock): SSO + entitlements + button-driven two-way sharing (Org + VRM)
- [ ] 32. Parts catalogues: GSF + Euro Car Parts (metered if charged)
- [ ] 33. HaynesPro: labour times + Service Assist animations (metered)
- [ ] 34. Accounting export: Xero / Sage / QuickBooks
- [ ] 35. Migration importer: TechMan, Garage Hive, Dragon2000, GA4, My Garage CRM
- [ ] 36. Fleet/insurer accounts + insurance/warranty claim workflow
- [ ] 37. Tyre module depth (brands, focus brands, stock/sales/turnover)
- [ ] 38. Marketing & reviews (review funnel, lead-source analytics)
- [ ] 39. Advanced cross-branch BI

---

## Decisions log (locked)

- **Name/brand:** GarageAxon. Tagline "Run your whole garage." Colours: #f99d23 / #8730d7 / #00012f / #ffffff (GeekAxon family). G+A monogram logo.
- **Domains:** staging garageaxon.geekaxon.com; production garageaxon.com.
- **Repo:** separate from GBP (own Turborepo).
- **Stack:** NestJS + Postgres/Prisma + Next 14 + Tailwind/shadcn + PM2/Nginx/aaPanel + Socket.io + BullMQ/Redis + MinIO.
- **Suite/GBP integration:** Option B — GBP as identity/billing hub behind a swappable suite-account interface; SSO; add-on entitlements; **separate databases** joined on Org ID + VRM; button-driven two-way sharing. **Mocked in Phase 1**, real in Phase 2.
- **Simple↔Pro:** system-wide framework, per-module or global, role-aware (garage sets ceiling, role sets visibility), lossless, full data always captured. Some modules Simple-only/Pro-only.
- **Familiar concepts, modern execution:** match incumbent workflow/vocabulary, never copy UI.
- **Accounting:** one double-entry engine; Simple "Money In/Out" + Full views; switchable anytime; UK standard VAT (per-customer tax code); deferred revenue for plans; staff pay posts to ledger.
- **Plans:** fully dynamic (public + hidden), admin-managed, public pricing site.
- **Roles/permissions:** fully dynamic; dependency graph; tenant-scoped default-role edits; custom roles.
- **Credits:** single unified balance; per-action price list; packages + pay-as-you-go top-ups; pre-flight check; **DVLA/DVSA free, unmetered**.
- **Payments (Phase 1):** Stripe, PayPal, GoCardless, Dojo, Payment Assist + manual (cash/bank/card-machine/cheque); all post to one invoice→ledger pipeline; garages connect their own gateway accounts.
- **DVLA:** live fetch per job (never cached); mismatch alert vs stored make/model.
- **Notifications:** three independent consent streams (operational / reminders / marketing), each channel-selectable; per-job preference resolver (reg → customer's last reg → garage default); per-garage sender identity (email domain auth, SMS sender ID, WhatsApp business profile). Channels: Postmark / Twilio / WhatsApp Business API.
- **Smart Booking:** five surfaces — WordPress plugin (WordPress.org + direct download), JS embed, hosted page, microsite (custom domain + auto-SSL), API.
- **Telephony:** FreePBX/Asterisk on a dedicated VPS (human runbook); BYO SIP (plan-gated); WebRTC + softphone + desk-phone options; call-center app-side in Phase 1.
- **Servers:** two dedicated Oracle Cloud VPS (CRM with aaPanel/Nginx/PM2; telephony isolated).
- **Apps:** customer PWA + technician PWA (ultra-minimal).
- **White-label:** per-tenant branding as a plan feature; else GeekAxon default.
- **Migration:** start fresh (legacy Prestige PHP not migrated); importer for rival systems in Phase 2.
- **i18n:** scaffolding from day one; English default.
- **GDPR/data retention:** first-class concern (consent records, call recordings, deletion path).
- **Dropped:** timing-belt reminders.

---

## Decisions / assumptions to resolve — [DECIDE AT BUILD]

- SSO token format / exact mechanism for the suite-account interface (Phase 1 mock; finalise at step 3 / Phase 2 step 31).
- VoIP/call-center billing specifics (bundled per-seat + metered minutes vs BYO-SIP flat fee) — confirm at step 30.
- Wallet UI: two clearly-labelled balances (GarageAxon credits vs GBP credits) — confirmed direction; finalise UI at step 7/31.
- Default UK garage **chart of accounts** contents — finalise at step 21.
- VAT beyond standard (margin / flat-rate schemes) — Phase 1 standard only; revisit later.
- Plans & Coverage edge rules: coverage conflict/overlap stacking order, warranty per-instance vs per-category (direction: per-instance, fitment-dated), deferred-revenue specifics — finalise at step 23.
- Payment gateways: which of the five ship first vs fast-follow within Phase 1 — confirm at step 22.
- Parts-supplier API order (GSF vs Euro Car Parts first) — step 32.
- Labour/technical data vendor (HaynesPro assumed) — step 33.
- WordPress plugin: WordPress.org directory submission is human-gated (owner submits); auto-update feed for direct downloads points to our server — step 29.
- Custom-domain DNS-verification + Let's Encrypt automation: any server/DNS/cert action the agent can't perform → `[HUMAN_REQUIRED]` — step 28.
- Notification channel fallback rules when a garage runs out of credit — finalise at step 11.

---

## Checkpoint log

(empty — record each APPROVE/REJECT here with date + module as the build proceeds)

---

## Session notes

(empty — the agent appends brief notes per session: what was built, decisions made, anything raised for human review)
