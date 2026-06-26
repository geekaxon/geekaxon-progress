> ⚠️ **PUBLIC FILE** — This document is pushed to a **separate public progress repo** (per-project folder). It must contain **NO secrets, NO hostnames, NO real subdomains, NO API endpoints, NO credentials, NO keys — ever.** Placeholders only. The code repo is private; this file is not.

# KhundiConnect — Complete Project Architecture

> **Product name:** **KhundiConnect** (the platform's own brand; replaced by a tenant's own logo/brand only when that tenant has the paid White-Label module).
>
> **Purpose of this document:** This is the single source of truth for building **KhundiConnect** — a multi-tenant SaaS Community Management platform. It is written to be handed directly to an agentic AI development tool. Every section is intended to be unambiguous and buildable. Where a decision is deliberately deferred, it is marked **[DECIDE AT BUILD]**.

> **Numbering authority:** The **§27 Build Order (steps 1–18)** is the *canonical* numbering for this project. All branch names, checkpoints, and `PROGRESS.md` rows reference **build-step numbers** (e.g. "Module 4 = Permission Engine"). The module list in **§8 is "feature areas"** for description only — its numbers are **not** the build order. When this project says "Module N", it always means **build-step N from §27.**

---

## 1. Executive Summary

**KhundiConnect** is a **multi-tenant SaaS** that digitizes the operations of *Khundis* (family-based sub-groups) within the **Okhai Memon Jamat (OMJ)** community. Each Khundi currently manages members, fees, welfare, census, and events manually (registers, Excel, WhatsApp). This platform replaces all of that with one central system where each Khundi is an isolated tenant with its own portal, branding, roles, and subscription.

The platform serves **three access altitudes**:

1. **Vendor (Platform Owner)** — the business operator (you). Full control: create tenants, billing, module toggles, white-label, impersonation, platform analytics.
2. **OMJ Super-User Layer** — the supreme community body. Cross-Khundi oversight via dynamic, permission-scoped roles. **Not a tenant** and **not separately built as its own DB/accounting** — OMJ logs into *this* portal as a privileged layer above all tenants.
3. **Khundi Tenant** — the operating unit (e.g. *Jafrani-B*, *Toberiya-A*). Its own Super Admin, roles, members, money, welfare, census, events.

The flagship capability is **Census** — a crowdsourced, family-tree-based community data system feeding **AI Insights** that drive welfare and growth decisions.

> **PROGRAM STATUS.** MVP (build-steps 1–18), the v2 depth pass (19–37), and v3 (38–53) are **complete and on staging**. The current program is **v3.1 (build-steps 54+)** — a foundation-first experience-overhaul + new-capability program (shared UI primitives, member model + timeline, form engine + census, finance + community modules, communication + access + a new Polling/Elections module, PWA + Reporting). v3.1 runs **continuous development** (auto-approve → staging → continue; production stays manual). The cross-org welfare ledger remains deferred to **v4** (§27.1). See §27 for the canonical build order and `specs/README-v3.1.md` for the v3.1 decision register.

---

## 2. Cultural & Domain Model (Critical Context for the Builder)

The builder **must** understand this domain model; the data model and business rules derive directly from it.

### 2.1 Hierarchy

```
OMJ (supreme body)
 └── Khundi Family  (naming/label layer only — e.g. "Jafrani", "Toberiya", "Kath")
      └── Sub-Group  (the REAL operating unit — e.g. "Jafrani-B")  ← THIS IS THE TENANT
           └── Members (attached to a sub-group; identity owned centrally by OMJ)
```

- In everyday speech, people call a **Sub-Group** a "Khundi." Throughout this app, **"Khundi" = the tenant = the sub-group.**
- The **Khundi Family name** (Jafrani) + **Sub-Group letter** (B) are two fields on the tenant; display name = `{family}-{subgroup}` (e.g. `Jafrani-B`). A tenant with no subdivision is just `{family}` with a null sub-group.
- A member's chain of belonging: `Member → Sub-Group (tenant) → Khundi Family (label) → OMJ`.

### 2.2 Khundi = Patrilineal Family

A Khundi is fundamentally an **extended paternal family**. Membership flows **father → children**. A person's Khundi is their **father's** Khundi.
- Same Khundi as a male member: his father, brothers, grandfather, paternal uncles (chacha, taya), his sons.
- Different Khundi: maternal relatives (mamu, khaloo) — they belong to the mother's Khundi.
- **Women** are born into their father's Khundi but on marriage join the **husband's** household/Khundi.

### 2.3 Membership Rules (Constitutional — must be enforced)

> **v2 DATA-MODEL CORRECTION (build-step 19).** The original build stored a `member_type` enum and an `is_honorary` flag, and carried Khundi/Group-code fields on the member. **These are removed.** The corrected model stores only **raw facts** and **derives** every category. The corrected rules below supersede the original; build-step 19 migrates the schema and all derivations. See §7.3 for the corrected table.

**Stored raw facts (the ONLY member-classification inputs a human sets):**
- `gender` (male/female)
- `dob` (date of birth)
- `is_okhai_by_birth` (boolean — replaces `is_honorary` and the `member_type` enum). True = born into the Okhai community (this Khundi's patrilineal line).
- `omj_card_number` (nullable — present = OMJ-issued canonical identity; absent = a *record*, not a *user*).
- `status` (active/suspended/expired/inactive).
- `marital_residence` (light indicator: `lives_in_khundi` | `married_out`) — only meaningful for women; lets the tree/census show a daughter who married out without inventing a member-type.

**Everything else is DERIVED, never stored, never hand-set:**
- **Voting-eligible** = `status=active` AND `gender=male` AND `age≥18` AND `is_okhai_by_birth=true`. Computed on read (age moves with the calendar). Never a manual flag. **(v3.1 note: voting-eligibility is a simple derived filter used only for building audiences/segments and messaging — it carries no special module behaviour and is not OMJ-API-dependent. Its scope is OMJ-constitution franchise; Khundi-level polls/elections set their OWN per-poll eligibility rules — see the Polling module, build-steps 54+.)**
- **(v3.1 — "honorary" REMOVED.)** The term *honorary* is dropped entirely from the product. A female member is simply a member; where a demographic split is needed, **split by gender**, never by an "honorary" label. The old `isHonorary` derivation (= female) is removed from code, counts, badges, dashboards, and copy.
- **Married-In (v3.1 — replaces the old "honorary female married in" idea):** a **non-Okhai female married to an Okhai male** is recorded with full OMJ-facility access (she is NOT OMJ-issued a membership; her children ARE full OMJ members). She is marked **"Married-In"**, a marker **DERIVED by relationship** (linked as spouse to an Okhai male + `is_okhai_by_birth=false`) — not a stored flag. On **divorce**, removing the marriage link sets her to **Inactive** (after a confirmation dialog); history is preserved (the closed marriage link + a timeline entry), never deleted; a re-marriage opens a new link.
- **Child** = `age<18`. **Adult** = `age≥18`. Derived from `dob`.
- **Non-member Okhai-by-birth** = `is_okhai_by_birth=true` AND no active membership / no card — e.g. a deceased person needing a grave record (§2.6). Derived from the facts, not a stored type.
- **Khundi / Group Code are NOT member fields.** The tenant *is* the Khundi; Khundi family + sub-group + group code are read from the tenant. Auto-assigned from the selected/current Khundi — never entered on the member form. (Removed from the member table in build-step 19.)

**Children & the turning-18 flag:** children are recorded under their father; a **male child (Okhai-by-birth) turning 18 and active** becomes voting-eligible and counts toward thresholds — the system **flags this for human confirmation, never auto-activates.**

**Women & marriage (the corrected, fuller model — build-step 19):**
- A woman is **born into her father's Khundi** and stays a **member of that Khundi (the tenant)**. On marriage she joins her husband's household; KhundiConnect keeps her on her father's-Khundi roster with `marital_residence=married_out` rather than moving her.
- **Female married to an Okhai groom:** standard member record; husband resolvable by OMJ card.
- **Female married to a NON-Okhai husband (NEW — KhundiConnect registers her; the OMJ Lawazim software does not):** she is still `is_okhai_by_birth=true`, a member of her father's Khundi. We additionally capture **husband-side facts** (not a Khundi/Group code): husband name, husband father name, husband grandfather name, husband CNIC, husband community/biradari, surname, and his community-membership number if any. These are attached facts for census/relationship context — they never make her a member of a non-existent tenant.

**Thresholds & dissolution (unchanged):**
- **Khundi formation/split threshold:** a group must have **≥ 20 active 18+ male Okhai-by-birth voting-eligible members** to be recognized as a Khundi (applies at creation and when splitting off a new sub-group). Below 20 it is not a Khundi.
- **Falling below 20 does NOT auto-dissolve a Khundi.** Dissolution is a manual, authority-gated action (by OMJ Hon. General Secretary in the real world). The software **must never auto-dissolve or auto-flag dissolution** when headcount drops.
- **Suspension is an OMJ-level power**, not a Khundi power. (Suspension status is represented and respected; the act of suspending is an OMJ/Vendor-altitude permission.)

### 2.4 Identity: OMJ Card Number

- OMJ issues a **centralized Membership Card** with an **OMJ Card Number** to each member. This is the **canonical identity** and constitutionally meaningful ID. Master member identity is OMJ-owned.
- Khundi-printed cards have **no constitutional value** (cosmetic/internal only).
- **OMJ Card Number gates app login:** a member can log into the PWA **only if they hold an OMJ Card Number** — female members included. Members without a card are *records*, not *users*.
- **CNIC** (national ID) is the **primary match anchor** for data reconciliation; **OMJ Card Number** is the secondary/fallback anchor.

### 2.5 Fees & Lawazim

- **"Lawazim" = the yearly fee**, tracked **per-member, per-year**.
- **OMJ fee:** fixed **Rs 100/year per 18+ male member** (uniform, constitutional). Not collected from members directly — the **Councilor remits it in bulk** to OMJ on behalf of the whole Khundi.
- **Khundi fee:** **variable, set per-Khundi** (e.g. Jafrani-B = Rs 1500/year per 18+ male member). **Configurable per tenant.**
- **NO AUTO-DEDUCTION / NO AUTO-TRANSFER between the two.** They are two independent ledger entries:
  - Member pays Khundi → recorded as **income** (full amount, no split).
  - Councilor pays OMJ against an **OMJ-generated receipt** → recorded as a separate **expense** ("Lawazim paid to Jamat — {year}").
  - The system records reality as humans enter it; it never moves money between Khundi and OMJ automatically.

> **v2 LAWAZIM CORRECTIONS (build-step 25).** Refinements from real-world use, applied when the Lawazim module is deepened:
> - **Liable set is exactly the voting-eligible set:** active, male, 18+, Okhai-by-birth member. A man stays liable every year **until death** (e.g. liable at 79). Females (non-liable), children, non-members, suspended/inactive/expired are **never** in the collection register. The member picker and register list **only liable members** — never the full roster.
> - **The Lawazim collection screen shows ONLY Khundi-fee collection.** The Rs 100 OMJ fee is part of **expenses** — it must **not** appear as a "+ OMJ Rs 100/member" note on the collection screen, and the **OMJ remittance is recorded on the expense/accounting side**, not as a block on the Lawazim collection view. (The remittance data model is unchanged; only its placement is corrected — it belongs with expenses.)
> - **Carry-forward arrears ledger:** dues are a **running per-member balance**, not year-isolated. A member may pay in parts within a year, pay partially across years, or clear two years' fees in a later year after defaulting. The system maintains **outstanding/dues per member carried forward** and always shows the running balance. Professional partial-payment handling + payment history + clear outstanding tracking.

### 2.6 Death, Bereavement & Graves

- On a member's death, the Khundi may pay bereavement support (e.g. Jafrani-B pays Rs 15,000 for the public dinner). Amount is **per-Khundi configurable.**
- OMJ provides **free bus to graveyard** and a **free grave**.
- **Grave eligibility extends to anyone "Okhai by birth"** — not strictly a registered member. So a death/grave record may attach to a person who is **not** an active member.
- **Graveyards and grave numbers are issued by OMJ.** The Khundi only **records** the issued graveyard name + grave number (no allocation logic, no master registry, no collision-prevention in this software — pure record-keeping). OMJ super-users can view these records across Khundis.

> **v3 BEREAVEMENT CORRECTION (build-step 47).** Bereavement becomes its **own top-level module** (extracted from Welfare). Two corrections to the above: (1) **Bus Service is removed** — the free bus is a **Jamat-provided** service, not a Khundi record, so it is not tracked in this software (the `bus_service_recorded` field is dropped). (2) Graveyards are **no longer assumed OMJ-only** — a grave may be recorded in an OMJ graveyard **or any external/other graveyard** (graveyard name remains free-text/lookup; still pure record-keeping, no allocation logic). A bereavement payment posts a balanced expense to accounting (explicit human act, no-auto).

### 2.7 Council & Governance (CONTEXT ONLY — NOT BUILT)

OMJ has a 36-member Managing Board, a Council of Councilors, Office Bearers, etc. **This governance layer is explained for understanding only and is NOT a software feature.** Do not build board/council election management. The only governance artifacts that surface in software are **roles** (e.g. a Khundi may have President, General Secretary, Councilor, etc. as *roles*) — see the dynamic role system.

---

## 3. Top-Level Design Principles (Non-Negotiable)

1. **Simplicity first.** Users are mostly non-technical volunteer councilors and community members of varied literacy. The simplest path is always the default; advanced features stay hidden until requested. Plain language, Urdu support, icon-driven, minimal typing. **Where feature-richness conflicts with simplicity, simplicity wins.**
2. **Humans in the loop.** The system records reality; it does not act on its own. No auto-dissolution, no auto-suspension, no auto-money-movement, no silent data overwrites. Automation **flags for a human**, never executes irreversible actions.
3. **Single Page Application (SPA) — no page reloads.** One-time load, then smooth client-side transitions and animated state changes. (See §6.)
4. **Two independent gates on every action:** (a) **Module-Enablement gate** (is this module enabled/paid for this tenant?) AND (b) **Permission gate** (does this user's role allow this action at this scope?). Both must pass.
5. **Multi-tenant isolation.** A tenant must never see another tenant's data except through the explicitly-built OMJ/Vendor cross-tenant layer.
6. **Provenance & audit.** Every sensitive action (impersonation, welfare disbursement, census approval, financial entry, AI generation, suspension) is logged immutably with who/what/when.
7. **Secrets are sacred.** CNICs, API keys (messaging + AI), credentials — encrypted at rest, masked in UI, never in logs or URLs.
8. **Privacy on AI.** Census/personal data is aggregated/anonymized before being sent to any LLM. Never send CNICs or identifying detail unless strictly necessary.
9. **Modern, professional UI.** Great dashboards, animated charts, polished transitions. (See §6 and §13.)
10. **Built by agentic AI.** This document is structured so an autonomous agent can build module-by-module. Modules are decoupled; each has clear data, permissions, and UI contracts.

---

## 4. Recommended Tech Stack

Chosen for: SPA smoothness, multi-tenancy, scalability to 100+ tenants, modern dashboards, and being highly familiar to AI coding agents (so the agent builds reliably).

### Backend
- **NestJS (Node.js, TypeScript)** — modular, opinionated, excellent for multi-tenant SaaS with guards/interceptors for the dual-gate model. Module-per-feature maps 1:1 to this document.
- **PostgreSQL** — primary relational DB. Strong for relational/family-graph data, row-level security, JSON columns where needed.
- **Prisma** (or TypeORM) as ORM. **[DECIDE AT BUILD]** — Prisma recommended for type-safety and agent-friendliness.
- **Redis** — caching, sessions, rate-limiting, background-job queue backing (BullMQ).
- **BullMQ** — background jobs (daily OMJ fetch cron, notification sending, AI generation, report exports).

### Frontend
- **Next.js (React, TypeScript)** in SPA/App-Router mode — one-time load, client-side routing, no full page reloads. Server components for initial load speed, client navigation thereafter.
- **TanStack Query (React Query)** — data fetching/caching so navigation feels instant (cached data shows immediately, revalidates in background).
- **Tailwind CSS + shadcn/ui** — modern, consistent, professional component system.
- **Framer Motion** — smooth page/element transitions and micro-animations.
- **Recharts** (or Chart.js) — animated charts for dashboards.
- **next-pwa / Workbox** — PWA capabilities (installable, offline, push, camera).

### Client App
- **PWA (Progressive Web App)** — single codebase serves desktop dashboards and the mobile member app. Installable to home screen, offline census entry, push notifications, camera for QR scanning. No app-store friction. Native wrapper deferred to Phase 2 only if needed.

### Infrastructure
- **Cloud-ready, containerized (Docker).** Deployable to a VPS (Oracle Cloud ARM / aaPanel pattern) initially; scalable to managed cloud later.
- **Nginx** reverse proxy; **PM2** or container orchestration for process management.
- **Cloudflare** for CDN, DNS, custom-domain mapping (white-label), and TLS.
- **Object storage** (S3-compatible, e.g. Cloudflare R2) for documents, uploads, card images, census attachments.
  - **v3 (build-step 46):** add a **Google Drive** backend (ENV-selectable via `STORAGE_BACKEND`, behind the same `StorageService` contract; R2 + DEV fallback preserved), enforce a **per-Khundi storage quota** (configurable MB cap, checked on upload), and **auto-compress + dimension-reduce** every processable file (images/PDFs) before storing. Users only ever see **upload controls** — never a raw key/URL.

### Cross-cutting
- **Auth:** JWT (access + refresh). OMJ Card Number as login identifier + OTP-to-phone or password. **[DECIDE AT BUILD]** — OTP-to-phone recommended for low-literacy audience.
- **i18n:** Urdu + English from day one (Urdu-first option). RTL support for Urdu.
- **Adapter pattern** for pluggable providers (Messaging providers, AI/LLM providers).
- **Node v22** targeted from the start.

---

## 5. Multi-Tenant SaaS Architecture

### 5.1 Tenant Isolation Strategy

**Chosen model: Shared database, shared schema, row-level tenant isolation via `tenant_id` + PostgreSQL Row-Level Security (RLS).**

Rationale: simplest to operate and scale to 100+ tenants, cheapest, and — critically — makes the **OMJ/Vendor cross-tenant queries** (aggregate demographics, cross-Khundi insights) straightforward, since all tenant data lives in one queryable place. Schema-per-tenant or DB-per-tenant would make cross-tenant aggregation painful, which is a core requirement here.

**Isolation enforcement (defense in depth):**
- Every tenant-scoped table has a non-null `tenant_id`.
- **PostgreSQL RLS policies** enforce that a normal tenant session can only read/write rows matching its `tenant_id`. This is the hard backstop — even a buggy query cannot leak across tenants.
- Application layer also injects `tenant_id` automatically via a NestJS request-scoped context (resolved from JWT / subdomain).
- **Cross-tenant access** (OMJ/Vendor) uses a distinct, explicitly-permissioned execution path that bypasses single-tenant RLS in a controlled, audited way — never the default.

> Any change touching **RLS policies or `tenant_id`** is a checkpoint event (see AGENT.md). Tenant isolation is the single most important property of this system.

### 5.2 Tenant Resolution

- Each tenant has a default **KhundiConnect subdomain** (`{tenant}.khundiconnect.com`, auto-provisioned at tenant creation). **[CONFIRMED]** subdomain base = `{tenant}.khundiconnect.com`. A custom **subdomain or domain** is available **only with the paid White-Label module** (§15).
- Tenant is resolved from the host header (subdomain/custom domain) → mapped to `tenant_id`.
- Vendor and OMJ super-users authenticate at a **platform-level entry** and then *select* or *impersonate* a tenant; their session carries cross-tenant scope.

### 5.3 The Three Altitudes & Scope Model

| Altitude | Who | Scope | How represented |
|---|---|---|---|
| **Platform / Vendor** | You + your team | All tenants, all platform controls | Platform roles; full cross-tenant; billing/module control |
| **OMJ Super-User** | OMJ board persons (permission-based) | Cross-Khundi, **scoped by permission** | Dynamic OMJ roles; e.g. "impersonate-as-admin" role vs "read-only-analytics" role |
| **Khundi Tenant** | Khundi Super Admin + tenant users | Single tenant only | Tenant-managed dynamic roles (President, Gen.Sec, Councilor, Accountant, Operator, View-Only, custom) |

**Key distinction the builder must preserve:** *Vendor* (the SaaS owner) and *OMJ* (the apex customer who can see across tenants) are **different things** and must be modeled separately, even though both are "above" tenants.

### 5.4 Module-Enablement (Commercial Gate)

- Every **module** is independently toggleable **per tenant**, controlled by the Vendor. The enabled set drives billing.
- A disabled module is **fully unreachable**: no route, no API access, no menu item, no leftover data exposure — not merely hidden.
- Implemented as a per-tenant `module_flags` record checked by a NestJS guard on every module route, *before* the permission guard.

### 5.5 Dynamic Roles & Permissions Engine (Architectural Backbone)

Everything hangs off this. **Not hardcoded roles** — fully dynamic at runtime, at all three altitudes.

- **Permission** = a granular capability string, namespaced by domain + action, e.g. `members.view`, `members.edit`, `census.approve`, `accounting.view`, `accounting.entry.create`, `welfare.disburse`, `tenant.impersonate`, `insights.generate`, `insights.view`, `ai_key.manage`, `crossKhundi.view`.
- **Scope** = the second dimension: `OWN_TENANT` | `TENANT_SET` (a chosen subset) | `ALL_TENANTS` | `PLATFORM`. The same permission means different reach at different altitudes (a Khundi's `members.view` = one tenant; an OMJ role's `members.view` + `ALL_TENANTS` = all).
- **Role** = a named bundle of (permission × scope) pairs. Roles can be created, cloned, edited, time-limited, and optionally IP-restricted.
- **Role assignment** = (user × role × optional tenant context). A single human can hold **multiple stacked roles** (e.g. member of Jafrani-B + an OMJ analytics role).
- **Protected roles:** the Khundi Super Admin role (per tenant) and the Vendor Owner role are protected/unremovable so a tenant cannot lock itself out.
- **Authorization check (every request):** `moduleEnabled(tenant, module)` AND `hasPermission(user, permission, requiredScope, targetTenant)`.

> Any change to **auth or the permission engine** is a checkpoint event (see AGENT.md).

---

## 6. SPA / "No Page Reload" Architecture

This is an explicit requirement: **one-time load, then smooth in-app transitions with animated charts.**

- **Single Page Application** via Next.js App Router client navigation. The shell (sidebar, top bar, tenant context) loads once and persists; only the content region swaps.
- **Client-side routing** — navigating between modules never triggers a full document reload.
- **TanStack Query** caches all fetched data. Returning to a previously-visited screen shows cached data **instantly**, then silently revalidates — so the app feels reload-free even on data changes.
- **Optimistic UI** for create/edit actions where safe, so the UI responds immediately.
- **Framer Motion** for route transitions (fade/slide), list item enter/exit, modal/drawer animations, and skeleton-to-content transitions.
- **Animated charts** (Recharts) — charts animate on mount and on data change (count-up numbers, growing bars, drawing lines).
- **Skeleton loaders** instead of spinners/blank screens during fetches.
- **Persistent layout state** — sidebar collapse, filters, and tenant selection survive navigation.
- **Code-splitting / lazy loading** per module so initial load stays fast despite the large feature set.
- **Offline-capable (PWA)** for member-facing flows (esp. census entry) — queue writes locally, sync when back online.

---

## 7. Database Design Structure

High-level relational design. Names indicative; the building agent should implement with proper migrations, indexes, FKs, and RLS. All tenant-scoped tables carry `tenant_id` (FK → `tenants.id`) and an RLS policy.

### 7.1 Platform / Tenancy
- **tenants** — `id`, `khundi_family`, `sub_group`, `display_name` (derived), `default_subdomain` (KhundiConnect-provided, e.g. `{tenant}.khundiconnect.com`), `custom_domain` (nullable; only active with White-Label), `status` (active/suspended/inactive), `branding` (logo, colors, brand name — JSON; applied only when White-Label flag on, else KhundiConnect defaults), `subscription_plan_id`, `created_at`.
- **subscription_plans** — `id`, `name`, `billing_cycle`, `price`, `included_modules` (JSON), `limits` (JSON).
- **subscriptions** — `id`, `tenant_id`, `plan_id`, `status`, `start`, `renews_at`, `payment_history` (or separate table).
- **module_flags** — `tenant_id`, `module_key`, `enabled` (the commercial gate source).
- **support_tickets** — `id`, `tenant_id`, `subject`, `status`, `messages`...

### 7.2 Identity, Roles & Permissions
- **users** — `id`, `omj_card_number` (unique, login identifier; login users must have it), `phone`, `auth_credential` (hashed), `locale`, `status`.
- **permissions** — `id`, `key`, `domain`, `action`, `description` (seed catalog).
- **roles** — `id`, `name`, `altitude` (platform/omj/tenant), `tenant_id` (null for platform/omj), `is_protected`, `time_limited_until`, `ip_restrictions` (JSON).
- **role_permissions** — `role_id`, `permission_id`, `scope` (OWN_TENANT/TENANT_SET/ALL_TENANTS/PLATFORM), `tenant_set` (JSON, when TENANT_SET).
- **user_roles** — `user_id`, `role_id`, `tenant_context` (nullable), `assigned_at`, `assigned_by`.

### 7.3 Members & Family Graph

> **v2 CORRECTED SCHEMA (build-step 19).** Replaces the original `members` definition. Removes `member_type` and `is_honorary` (now derived) and removes Khundi/Group-code (read from the tenant). Adds `is_okhai_by_birth`, `marital_residence`, and the husband-side fact block. Member profile depth (the full Add-Member field set, minus the dropped fields) lands in build-step 23; the membership-vs-census field split (which facts live here vs. on a census entry) is finalised in build-steps 19/24.

- **members** — `id`, `tenant_id`, `omj_card_number` (nullable; canonical identity when present), `cnic` (encrypted, nullable), `full_name`, `father_name`, `grandfather_name`, `gender`, `dob`, **`is_okhai_by_birth`** (bool — the single okhai/honorary input; replaces `member_type`+`is_honorary`), `status` (active/suspended/expired/inactive), **`marital_residence`** (`lives_in_khundi` | `married_out`, nullable), `blood_group` (nullable), `marital_status` (nullable), `education` (nullable), `occupation` (nullable), `income_band` (nullable), `disability` (nullable), `contact` (JSON: cell/landline/whatsapp/email/emergency), `address` (JSON), `household_id`, `provenance` (omj_fetch/manual), `photo_ref` (nullable), `remarks` (nullable), `created_at`.
  - **Husband-side facts** (only for a female married to a non-Okhai husband — §2.3; attached facts, NOT a Khundi/Group code): `husband_name`, `husband_father_name`, `husband_grandfather_name`, `husband_cnic` (encrypted, nullable), `husband_community` (biradari), `husband_surname`, `husband_community_membership_no` (nullable). Stored as a nullable JSON block on the member (or a 1:1 side table).
  - **NO `member_type`. NO `is_honorary`. NO `khundi`/`group_code` columns** — Khundi family, sub-group, and group code are read from the **tenant**, auto-assigned, never on the member.
  - **Derived (computed on read, never stored, never hand-set):** `is_voting_eligible` = active ∧ male ∧ age≥18 ∧ is_okhai_by_birth; `is_child` = age<18; `is_non_member_okhai` = is_okhai_by_birth ∧ no active card/membership; `is_pending_voting_activation` = male ∧ okhai ∧ just-turned-18 ∧ active (flag, never auto-activate). These power the threshold count (counts only eligibles). **(v3.1: the old derived `is_honorary` = female is REMOVED — split by `gender` where needed. A new derived `is_married_in` = (linked SPOUSE_OF an Okhai male) ∧ `is_okhai_by_birth=false` marks a non-Okhai woman married into the Khundi; on divorce the link closes and her `status` becomes `inactive`.)**

> **v3.1 MEMBER NOTE (build-steps 54+).** (1) **"Honorary" is removed** as a concept/label everywhere — gender split replaces it. (2) A **Married-In** marker (derived by relationship, above) is added and is filterable in lists/demographics. (3) A **per-member timeline** is added — a chronological history of life events (marriage/divorce, prize participation/win, academic results, status changes, census participation), partly **auto-written** by modules and partly **manually added**; immutable entries, newest-first, filterable by type. (4) A free-form **controller notes** field is added — private operator notes, **role-gated** (`members.notes`, controllers/admins only, never shown to the member or in member-visible exports), **audited** on write. (5) **Marriage/divorce history is never deleted** — links close with a date; current markers derive from the open link.

> **v3 MEMBER/IDENTITY NOTE (build-steps 45, 48).** **CNIC** is promoted to the **stable cross-context member anchor** (the one identifier a member cannot deny) — captured/validated, **encrypted at rest, masked on display** (e.g. `*****-*******-*`), permission-gated + audited on any reveal, never returned raw; **OMJ Card Number is kept** as the canonical OMJ identity. A **household / living-arrangement grouping** is added (build-step 45): who **lives together** vs **separately**, independent of blood relationship (two related brothers may live in separate households; an in-law lives in by marriage). This grouping powers the family-tree household overlay and the welfare household-dedup (build-step 48), and is forward-compatible with the v4 cross-org ledger (§27.1). Tenant isolation stays ABSOLUTE in v3.
- **households** — `id`, `tenant_id`, `head_member_id`, `address` (JSON), `notes`.
- **relationships** — `id`, `tenant_id`, `from_member_id`, `to_member_id`, `type` (PARENT_OF / CHILD_OF / SPOUSE_OF / SIBLING_OF). In-law links derived through SPOUSE_OF. This table powers the navigable family tree (professional visualization rebuilt in build-step 23).
- **member_status_history** — `member_id`, `tenant_id`, `old_status`, `new_status`, `reason`, `changed_by`, `at`.

### 7.4 OMJ Data Sync
- **omj_api_config** — `tenant_id`, `api_key` (encrypted), `last_fetch_at`, `last_fetch_status`, `enabled`. (Base endpoint URL lives in env, not here.)
  - **v3 (build-step 49):** the sync **key/connection moves to the OMJ/platform side** (one central connection) — Khundis no longer enter keys. A Khundi still **triggers** a sync from its side (key-less); the **review/conflict queue and no-silent-overwrite rule are preserved**.
- **fetch_conflicts** — `id`, `tenant_id`, `match_key` (cnic/card), `incoming_data` (JSON), `local_member_id` (nullable), `conflict_type` (conflict/new/unchanged), `status` (pending/accepted/rejected/merged), `created_at`, `resolved_by`. (Cron updates existing pending conflict rather than stacking duplicates.)

### 7.5 Census
- **census_snapshots** — `id`, `tenant_id`, `label` (e.g. "Census 2026"), `status` (open/finalized), `opened_at`, `finalized_at`, **`form_definition_id`** (the form-builder definition this round uses — build-step 24).
- **census_entries** — `id`, `tenant_id`, `snapshot_id`, `member_id` (nullable for new persons), `submitted_by_user_id`, `data` (JSON: answers keyed by form-field, incl. household/relation/income-band/education/profession/welfare-need), `status` (pending/approved/rejected), `approved_by`, `approved_at`. (Approval by Councilor/President/Gen.Sec roles.)

> **v3 CENSUS CORRECTION (build-step 53).** Census becomes **facts-only**: the **welfare-need flag is removed** from census entries (self-declared need is undignified + unreliable). New **fact** fields are added (family/household-centric): **home ownership** (own/rent/goodwill), **transport ownership**, **staged education** (school/college/university with in-vs-out per stage → retention-rate analysis), and a structured **needs/asks** capture (housing / business / education-finance / other — the family's expressed wants, distinct from a welfare case). A **vulnerability/priority indicator is DERIVED** from these facts (computed on read, like voting-eligibility — never a stored self-declaration) and surfaced as a **welfare outreach-suggestion view** (a prompt for humans; **never** auto-creates a welfare case — no-auto). Census also gains a **report/presentation generator** producing a shareable, chart-driven community report (the "Our Community, Our Plan" deck) from an approved round.
- Family-tree data is read from **relationships** + **members**; census adds economic/welfare fields and snapshotting.
- **census_progress** (view/derived) — per-tenant completion %, counts.

> **v2 CENSUS DEEPENING (build-step 24).** Census becomes **form-builder-driven** (build-step 20): each round ships a **default form** the Khundi can customize — they cannot remove **locked-core** fields but can author their own **option sets** per field (Profession/Education/income/etc.) in either a **light** style (Below Average / Average / Above Average) or a **detailed** style (actual ranges/lists), and add unlimited **custom fields**. Public-facing **multi-step**, low-literacy UX. Locked core (for population + cross-Khundi comparability): household, relation-to-head, name, gender, DOB, marital status, okhai-by-birth, OMJ card (if any), education, profession, disability. **Income band is OPTIONAL** (public may skip; welfare/insights treat it as best-effort/sparse). **OMJ receives all Khundi data including custom fields, but cross-Khundi aggregation / Jamat-AI uses ONLY the locked core** (custom fields aren't comparable across Khundis; they still appear in that Khundi's own reports and to OMJ per-Khundi). The membership-vs-census split: stable identity facts live on the **member**; round-specific economic/welfare answers live on the **census entry**.

### 7.6 Lawazim & Accounting
- **fee_config** — `tenant_id`, `year`, `khundi_fee_per_member`, `omj_fee_per_member` (default 100).
- **lawazim_payments** — `id`, `tenant_id`, `member_id`, `year`, `amount`, `paid_at`, `recorded_by`. (Member→Khundi income; no auto-split.)
- **omj_remittances** — `id`, `tenant_id`, `year`, `receipt_ref`, `amount`, `paid_at`, `recorded_by`. (Khundi→OMJ expense; separate.)
- **accounts** (chart of accounts) — `id`, `tenant_id`, `name`, `type` (income/expense/asset/liability), `is_default`, `parent_id`. Seeded defaults: Lawazim, Donations, Sponsorships, Event Collections (income); Bereavement, Welfare, Prize Distribution, Get-together, Printing/Cards, General (expense).
- **bank_accounts** — `id`, `tenant_id`, `name`, `opening_balance`...
- **journal_entries** — `id`, `tenant_id`, `date`, `description`, `created_by`, `source_module` (lawazim/welfare/event/manual). **Always double-entry underneath, regardless of accounting mode.**
- **journal_lines** — `entry_id`, `account_id`, `debit`, `credit`, `bank_account_id` (nullable). (Simple mode auto-assigns default accounts; Detailed mode exposes/edits them.)
- **accounting_settings** — `tenant_id`, `mode` (simple/detailed). Mode is a *view* over the same double-entry data; switching is non-destructive and retroactive.

### 7.7 Welfare, Bereavement, Graves
- **welfare_cases** — `id`, `tenant_id`, `category` (medical/student/widow/emergency), `subject_member_id`, `confidential` (true), `notes` (restricted), `status`, `approval_workflow_state`.
- **welfare_disbursements** — `case_id`, `amount`, `disbursed_at`, `disbursed_by`, `journal_entry_id`.
- **death_records** — `id`, `tenant_id`, `deceased_person` (member_id nullable — supports Okhai-by-birth non-members), `name`, `date_of_death`, `bereavement_amount`, `bus_service_recorded` (bool), `graveyard_name`, `grave_number`, `grave_issued_date`, `recorded_by`. (Graveyard/grave = OMJ-issued, Khundi only records.)
  - **v3 (build-step 47):** `bus_service_recorded` is **dropped** (Jamat-provided, not a Khundi record). Add a graveyard **source** indicator (OMJ vs external) so non-OMJ graveyards are supported; `graveyard_name`/`grave_number` stay free-text (no allocation logic). Add `journal_entry_id` linking the bereavement payment to accounting (build-step 26). Bereavement moves to its own module surface.

> **v2 WELFARE DEEPENING (build-step 27).** `welfare_cases` gains: **fund source** (`zakat` | `general` | `endowment` — Zakat carries eligibility rules, so the source is explicit and reportable separately), richer **case types** (Khundi-configurable categories + sub-types via the form-builder, beyond the four seed categories), structured **case fields**, **document upload** on a case (attachments → R2), and an **inquiry/investigation report** record (a case can require an inquiry step + findings before approval/disbursement). All no-auto-action rules hold (disburse only after approval).

> **v3 WELFARE DEEPENING (build-step 48).** Welfare gains: **per-fund available balances** (Zakat/Endowment/General — Zakat never mixed with general/endowment in reporting), tied to accounting; a **recipient + family/household welfare-history** view shown in the case flow (confidential, dignity-preserving, welfare-access gated); **household-aware dedup** — using the household/living-arrangement grouping (build-step 45), the system **flags** when a same-household member has an open/recent request of the same type (e.g. ration) so the team approves only one (a human prompt, **never** an auto-reject); enforcement that **every payment across all modules posts to accounting** (corrections via reversal, not silent edit); an optional, per-Khundi-authorable **Sharia Compliance form** (form-builder; a truthfulness declaration / deterrent layer; immutable acceptance snapshot); and **CNIC as a forward-compatible member anchor** (captured/validated, masked on display, permission-gated, audited; OMJ Card kept). **Tenant isolation stays ABSOLUTE in v3** — the CNIC anchor only makes the model ready for the v4 cross-org ledger (§27.1); no cross-tenant welfare data is shared in v3.

### 7.8 Events & Meetings
- **events** — `id`, `tenant_id`, `type` (meeting/gathering/prize/welfare_drive/sports), `title`, `datetime`, `venue`, `budget`, `status`, `archived`.
- **event_registrations** — `event_id`, `member_id`, `rsvp_status`, `attended` (bool, QR-set).
- **meeting_minutes** — `event_id` (or standalone), `tenant_id`, `minutes`, `resolutions` (JSON list), `attendance_ref`.

> **v2 EVENTS DEEPENING (build-step 28).** Full **datetime + budget**, proper **RSVP management**, real **attendance/registration** (human mark or QR — never auto), **structured resolutions**, and **archive**. Prize ceremonies link to an event (typically the Annual Meeting). All Simple/Pro-aware.

### 7.9 Cards, Documents, Notifications
- **cards** — `id`, `tenant_id`, `member_id`, `type` (membership/qr/attendance), `qr_payload`, `template`, `printed`. (Khundi cards = non-constitutional. QR encodes the OMJ card number + ids only — never a CNIC/secret, §2.4/§17.)
- **documents** — `id`, `tenant_id`, `category` (minutes/notice/constitution/financial/historical/prize_doc), `file_ref`, `access_roles` (JSON), `uploaded_by`, `uploaded_at`, `size`, `mime`.
- **messaging_config** — `tenant_id`, `provider` (twilio/infobip/...), `credentials` (encrypted), `enabled`.
- **notifications** — `id`, `tenant_id`, `type`, `channel` (sms/whatsapp), `recipients`, `body`, `sent_at`, `status`. (Communication log.)

> **v2 DOCS & CARDS DEEPENING (build-step 30).** Documents get **real file upload to R2** (not a raw "file ref" string), metadata (uploader/date/size/mime), **search**, and a proper **role-ACL editor UI**. Cards get a **visual rendered membership/QR card** (never raw payload strings on screen), card **templates**, **print** and **digital-wallet** views. **Prize Distribution** gains its own data model in build-step 29 (form definitions, open/close windows, submissions, configurable categories/criteria incl. Hifz milestones, position naming, gift/prize assignment, **certificate canvas** templates with relative-coordinate placeholders, bulk certificate generation, Excel + announcement-list exports).

### 7.10 AI Insights
- **ai_config** — `owner_altitude` (vendor/omj/tenant), `tenant_id` (nullable), `provider` (openai/claude/...), `api_key` (encrypted), `enabled`, `managed_by`.
  - **v3 (build-step 49):** AI keys/config **move to the OMJ/platform altitude** — Khundis no longer manage API keys. OMJ generates an **overall Jamat-wide insight + per-Khundi insights** in one run; each Khundi **receives and views its own** (read-only, tenant-scoped; isolation enforced). Generation is an explicit OMJ action (no-auto); insights are advisory prompts.
- **ai_insights** — `id`, `scope` (JAMAT/KHUNDI), `tenant_id` (nullable; set for KHUNDI scope), `generated_by_altitude` (vendor/omj/tenant), `generated_by_user`, `provider`, `model`, `data_snapshot_ref` (which census snapshot), `content` (JSON/markdown), `generated_at`, `is_stale` (bool). Cached & shared per the visibility rules (§11).

### 7.11 Audit
- **audit_log** — `id`, `actor_user_id`, `actor_altitude`, `action`, `target_type`, `target_id`, `tenant_id`, `metadata` (JSON), `at`. Immutable. Captures impersonation, disbursements, census approvals, financial entries, AI generation, suspensions, module/permission changes.

---

## 8. Module Breakdown (FEATURE AREAS — not the build order)

> The numbers below are **descriptive labels for feature areas only.** They are **NOT** the build sequence. The canonical build sequence is **§27 (build-steps 1–18).** Each feature area is a self-contained NestJS module + frontend feature folder, gated by module-enablement + permissions.

### Platform / Vendor Feature Areas
- **Tenant Management** — create/activate/suspend tenants; set family/sub-group; provision the one Khundi Super Admin account; subdomain/custom-domain mapping.
- **Billing & Subscriptions** — plans, cycles, invoices, payment history, renewals.
- **Module Toggles** — per-tenant enable/disable; drives billing & unreachability.
- **White-Label (PAID add-on)** — see §15. Default: KhundiConnect branding on `{tenant}.khundiconnect.com`. Purchased: own logo, color palette, brand name, own subdomain/custom domain.
- **Platform Analytics & Revenue Dashboard** — tenant usage, revenue, growth.
- **Support Tickets**.
- **Vendor Impersonation** — log into any Khundi as admin (audited).

### OMJ Super-User Feature Areas
- **OMJ Role & Permission Management** — dynamic OMJ roles (scoped).
- **Cross-Khundi Oversight** — aggregate + per-Khundi (filtered) demographics; census progress monitoring; graveyard records view; OMJ impersonation (audited).
- **OMJ AI Insights** — generate Jamat-wide insights (own API key), view all.

### Khundi Tenant Feature Areas
- **Khundi Admin & Roles** — Khundi Super Admin; tenant-managed dynamic roles/users.
- **Member Management** — members (by gender), children, non-member-Okhai records, Married-In members; status & lifecycle; per-member timeline + controller notes (v3.1); derived voting eligibility.
- **Family Graph & Households** — relationships powering the navigable family tree.
- **OMJ Data Sync** — API key config; manual + cron fetch; conflict review queue.
- **Census** — snapshots, crowdsourced entry, approval workflow, live demographics, family tree, economic/welfare fields, progress.
- **Lawazim / Fee Engine** — per-member-per-year tracking, defaulters, OMJ remittance record, reminders.
- **Accounting (Dual-Mode)** — simple/detailed switchable over one double-entry engine; chart of accounts; banks; receipts/vouchers; reports.
- **Welfare Case Management** — confidential cases, categories, approval, disbursement.
- **Bereavement & Death/Grave Register** — death records, bereavement payment, bus record, OMJ-issued grave recording, non-member support.
- **Prize Distribution (optional)** — application forms, doc upload, approval, eligible list, assignment, history.
- **Event Management** — events, RSVP, registration, QR attendance, budgets, archive.
- **Meeting & Resolution Records (MoM)**.
- **Card System** — membership/QR/attendance cards, digital card wallet, printable.
- **QR Attendance Scanner** — PWA camera scanning.
- **Document Vault** — role-based secure document store.
- **Notifications** — bring-your-own messaging provider (adapter), SMS/WhatsApp, reminders, bulk, communication log.
- **Reports & BI Dashboards** — animated dashboards; PDF/Excel/CSV export.
- **Khundi AI Insights** — generate own-Khundi insights (own API key), view shared.

### Member-Facing (PWA) Feature Area
- **Member Self-Service** — card-gated login; profile view + update requests (approval-gated); family census entry; fee view/pay; prize application; event registration; card wallet; notifications; suggestions.

### Cross-Cutting Services
- **Auth & Identity** — OMJ-card login, OTP/password, JWT, sessions.
- **Dynamic Permission Engine** — the dual-gate authorization.
- **Audit Service**.
- **Provider Adapters** — messaging + AI provider plug-ins.
- **i18n / Urdu-first UX layer**.
- **Lifecycle Automation (flag-only)** — turning-18, lawazim-due, status review.
- **Backup / Export / Data-Retention**.

---

## 9. User Roles & Permissions Matrix
> **v3.1 ADDITIONS (build-steps 54+).** (a) **Committee chairmen / volunteers** are modelled as **scoped custom roles** for v3.1 (e.g. "Census Chairman", "Prize Committee") using the existing role engine — a first-class Committees entity (with rosters, terms, and chairman elections via the Polling module) is a later/v4 add. (b) **Admin/config capabilities are NEVER mode-gated** (Simple/Pro is presentation only) — they are **permission-gated** only; the Simple/Pro toggle must never hide Roles & Permissions, provider config, user management, or accounting depth. (c) New permission `members.notes` (controller-only private member notes). (d) The **Polling & Elections module** adds per-poll, Khundi-configurable eligibility (who may vote / who may stand), independent of OMJ voting-eligibility.


Roles are **dynamic**; below are **seed/default** roles the system ships with. Tenants/OMJ can create more.

### Platform Altitude
| Role | Key permissions (scope: PLATFORM/ALL_TENANTS) |
|---|---|
| **Vendor Owner** (protected) | Everything: tenant CRUD, billing, module toggles, white-label, impersonate, platform analytics, AI generate (Jamat), audit view |
| **Vendor Staff** | Configurable subset (e.g. support tickets, no billing) |

### OMJ Altitude (all permission-scoped; assigned to OMJ board persons)
| Role | Key permissions |
|---|---|
| **OMJ Admin** | `crossKhundi.view` (ALL), `tenant.impersonate` (ALL), `insights.generate` (JAMAT), `insights.view`, census progress, graveyard view |
| **OMJ Analyst (read-only)** | `crossKhundi.view` (ALL), `insights.view` only — **no impersonation, no edits, no generation** |
| **OMJ Insights Operator** | adds `insights.generate` + `ai_key.manage` to read-only |

### Khundi Altitude (seed roles; tenant may clone/edit/add)
| Role | Key permissions (scope: OWN_TENANT) |
|---|---|
| **Khundi Super Admin** (protected) | All tenant permissions incl. role mgmt, settings, AI key, accounting mode |
| **President** | Broad view + approvals (census, welfare), reports |
| **General Secretary** | Members, census approval, documents, notifications, meetings |
| **Councilor** | Members, census approval, lawazim collection + OMJ remittance record |
| **Accountant** | Accounting full, lawazim, reports — no member admin |
| **Operator** | Data entry (members, events) — no approvals, no financials |
| **View-Only** | Read dashboards/reports only |

**Permission keys catalog (seed, extensible):** `members.{view,create,edit,delete}`, `family.{view,edit}`, `census.{view,enter,approve,finalize}`, `lawazim.{view,record,remit}`, `accounting.{view,entry,mode,banks,reports}`, `welfare.{view,create,approve,disburse}`, `death.{view,record}`, `prize.{view,apply,approve,assign}`, `events.{view,create,attendance}`, `mom.{view,edit}`, `cards.{view,issue}`, `documents.{view,upload,manage_access}`, `notifications.{view,send,config}`, `reports.view`, `insights.{view,generate}`, `ai_key.manage`, `messaging_key.manage`, `roles.manage`, `settings.manage`, `tenant.impersonate`, `crossKhundi.view`, `members.suspend` (OMJ-altitude).

---

## 10. Census System — Detailed Design (Flagship)

### 10.1 Why Census Lives at Khundi Level
A central census of 150,000+ is impractical; OMJ pushed it down to Khundis because each Khundi is a small extended family where members know each other. The 2005 census is the last; data is stale. This platform enables a fresh, crowdsourced, digital re-census.

### 10.2 Family Tree (Person-Centric, Bidirectional)
Rendered from any chosen person; pivots to **father's side** or **mother's side**. From a focal person, the tree resolves via the **relationships** table:
- **Up:** father & mother; grandfather & grandmother.
- **Across:** spouse → spouse's parents (in-laws) + spouse's siblings.
- **Down:** children → children's spouses *(optional)* → grandchildren *(optional)*.
- **Sideways:** own brothers & sisters *(optional)*.

**Block rendering rules:**
- **Required blocks** (father, mother, grandparents): if data unknown, render an **empty placeholder** ("data missing — add").
- **Optional blocks** (children's spouses, grandchildren, in-laws, siblings): if empty, **remove/hide** entirely.

### 10.3 Crowdsourced Entry + Approval
- **Any card-holding member (female included)** can enter/update their family's census via the PWA.
- Each submission/update is **pending** until approved by a **Councilor / President / General Secretary** (role with `census.approve`).
- On approval → official, counts in demographics, and becomes **visible to OMJ** (cross-tenant view).

### 10.4 Live Demographics + Dated Snapshots (Option C)
- **Live roll-up:** demographics (population, age bands, voting members, gender split, children, seniors, education, occupation, income/below-threshold) **derived continuously** from member + family data.
- **Dated snapshot:** run a formal "Census {year}", capturing extra economic/welfare fields, then **finalize** to freeze it for year-over-year comparison and AI analysis.

### 10.5 Census Data Captured (beyond routine member fields)
Total population; age distribution; **income band / below-living-wage flag**; employment status; education level; marital status; senior citizens; welfare needs; household composition. These feed welfare prioritization and growth decisions.

### 10.6 Visibility
- **Khundi:** its own demographics + its own census progress.
- **OMJ:** all-Khundi aggregate demographics; per-Khundi demographics via filters; each Khundi's census progress. (Permission-scoped.)

---

## 11. AI Insights — Detailed Design

### 11.1 Bring-Your-Own-Key, at Every Altitude
AI is **not** vendor-funded by default. Whoever wants insights **plugs in their own LLM API key** (OpenAI, Claude, etc.) and bears their own cost. Three independent generators:
- **Vendor** integrates key → can generate **Jamat-wide** insights.
- **OMJ** (permission-based) integrates key → can generate **Jamat-wide** insights.
- **Khundi** integrates key → can generate **its own Khundi** insights only.

### 11.2 Scope = Generator's Altitude
| Generator | Insight scope |
|---|---|
| Vendor | Whole Jamat |
| OMJ | Whole Jamat |
| Khundi | That Khundi only |

### 11.3 Shared, Cached, Cross-Altitude Visibility
- Generated insights are **cached** with **provenance** (who generated, which model, which data snapshot, when).
- **Khundi-generated** insight: visible to that Khundi + **upward** to OMJ & Vendor. **Never sideways** to other Khundis.
- **Jamat-wide** insight (Vendor/OMJ-generated): a Khundi viewing it sees **aggregate + its own slice**; **other Khundis' identifiable private detail is permission-protected** (a Khundi should not see another Khundi's private breakdown inside a Jamat-wide report).
- A Khundi with **no key can still view** insights generated by OMJ/Vendor — preserving a "free to tenant" experience without the vendor paying.
- **Cache structure (two tiers):** a **Jamat-level slot** (latest Vendor/OMJ Jamat-wide insights, both kept and tagged by generator) + **per-Khundi slots**.

### 11.4 Cost & Freshness Controls
- **On-demand + cached, not per-action.** Generate on census finalization or explicit "Generate Insights" click; then serve the cached result to all viewers.
- **Avoid redundant calls:** one cached result serves many viewers; don't auto-fire multiple generators on the same snapshot.
- **Staleness:** if underlying data changes after generation, mark insight `is_stale = true` ("based on older data — regenerate?") rather than silently showing outdated analysis. Never auto-regenerate (cost control + human-in-loop).

### 11.5 Provider Adapter & Permissions
- **Adapter pattern:** one internal `generateInsight()` interface; OpenAI / Claude / others as pluggable backends. New provider = new adapter + dropdown entry; zero change to consumers.
- **Three separate permissions:** `ai_key.manage` (integrate/manage key), `insights.generate` (spend/generate), `insights.view` (read). Set independently at each altitude.

### 11.6 Privacy (Mandatory)
- **Aggregate/anonymize** census data before sending to any LLM. **Never** send CNICs or directly-identifying personal detail unless strictly necessary. Whose key it is does not reduce this duty.

### 11.7 Use Cases (MVP)
- **Khundi-level:** census insights, **welfare prioritization** (who needs help most), demographic summaries, plain-language (Urdu) reports a non-technical councilor can act on.
- **OMJ-level:** cross-Khundi aggregate insights, Khundi comparisons, community-wide trends, anomaly spotting.

---

## 12. Notifications — Bring-Your-Own Messaging Provider
> **v3.1 OVERHAUL (build-steps 54+).** One notification/announcement engine: media attachments; show only ACTIVE channels; multi-member + multi-role + **custom saved-filter audiences** (one filter engine shared with table filters); **per-notification-type channel routing** + **per-channel templates** (OMJ ships defaults, Khundi overrides); a fixed scheduler; verified push tenant-scoping. **Announcement landing pages**: login-required (logged-in + tenant-eligible only) vs public-shareable (hard-restricted from welfare/CNIC/financial/confidential content), with URL shortening; Khundi pages on the Khundi subdomain, OMJ pages on a central OMJ URL. **OMJ-level cross-Khundi announcements** (all-members / specific-Khundi / councilors / roles / all-males / all-females / custom-filter audiences). **Login credentials:** password-primary; OTP optional and only offered where a Khundi has an SMS/WhatsApp provider configured (Khundi chooses the OTP channel); auto-generated passwords shown once and embedded in the WhatsApp/SMS credential template; a `wa.me` click-to-send handoff works without any provider API.


- Each Khundi enters **their own** messaging API credentials in Settings → messaging cost sits with the **tenant**, not the vendor.
- **Provider dropdown:** seed with **Twilio** and **Infobip** (both SMS + WhatsApp); extensible.
- If a tenant needs an unlisted provider, they request it → vendor **builds one adapter** → it **joins the dropdown for all tenants** (reusable).
- **Adapter pattern:** one internal `sendMessage()` interface; each provider is a plug-in using that tenant's credentials. Notification features never change when providers are added.
- **Credentials** = tenant secrets: encrypted at rest, masked in UI, never logged/in-URL.
- **Use cases:** fee/lawazim reminders, meeting notices, event invites, welfare alerts, form deadlines, bulk announcements, auto-reminders. **Communication log** records what was sent, when, to whom.

---

## 13. UI/UX & Dashboard Structure
> **v3.1 FOUNDATION (build-steps 54+).** The shared list surface (`DataList`) is redesigned: the **Actions column is FIRST** in table view and actions sit in the **card footer** in card view, with responsive **colored-icon ↔ label** controls and a **fail-loud** error on an unregistered list key. **Column preferences become per-USER**, set from the table header (over a tenant-wide admin default). **Filters** become dynamic/fuzzy/collapsible with automatic client-vs-server sourcing by row-count and debounced partial search (incl. debounced partial CNIC lookup where permitted). A real **toast system** + **inline field validation** replace the static banner. **Modals/keyboard**: ESC closes every popup, Cmd/Ctrl+Enter saves, plus rush-time shortcuts. **One project-wide pass** ensures human labels everywhere (no raw enums/keys/slugs) and complete Urdu/RTL coverage (hardcoded English is a defect). Member/other pages gain dashboard-style info-boxes. A dedicated **Reporting module** (hybrid server-side PDF + Excel) and a polished **census PDF report** are added.


### 13.1 Design Language
- **Modern, professional, polished.** Clean shadcn/ui components, generous spacing, consistent design tokens, light/dark mode.
- **White-label aware:** each tenant's logo/colors/brand theme the portal.
- **Animated dashboards:** Recharts charts animate on mount and data change; count-up KPI numbers; growing bars; drawing lines; smooth Framer Motion transitions between views.
- **Skeleton loaders**, no jarring spinners; instant cached navigation.

### 13.2 Accessibility for Low-Literacy / Non-Technical Users
- **Urdu-first option** (full RTL); English available. Plain language throughout.
- **Icon-driven navigation**, large touch targets, minimal typing.
- **Voice / photo input** for census where possible (speak or photograph a document instead of typing).
- Progressive disclosure — advanced options hidden until needed.

### 13.3 Dashboard Layouts by Altitude
- **Vendor dashboard:** tenants overview, revenue, growth, usage, tickets, system health.
- **OMJ dashboard:** Jamat-wide KPIs, all-Khundi demographics, per-Khundi filters, census progress heatmap, Jamat-wide AI insights, graveyard records.
- **Khundi dashboard:** members growth, paid/unpaid lawazim, income vs expenses, welfare spending, event attendance, youth/student counts, donation trends, aging demographics, this-Khundi AI insights + Jamat-wide insight (own slice highlighted).
- **Member (PWA) home:** my card, my fees/lawazim status, my family census, notifications, events, prize applications, suggestions.

### 13.4 Reports
- All dashboards exportable to **PDF / Excel / CSV**. Scheduled email exports (via tenant messaging/email) optional.

---

## 14. Subscription Plans Strategy

- **Mechanism (technical):** per-tenant **module flags** drive entitlement & billing regardless of packaging.
- **Recommended packaging:** **tiers + à-la-carte exceptions.**
  - **Basic:** Members, Family Graph, Lawazim, Simple Accounting, basic Reports.
  - **Standard:** + Census, Events, Notifications, Documents, Cards, QR Attendance.
  - **Premium:** + Welfare, Bereavement/Graves, Prize Distribution, Detailed Accounting, AI Insights, full BI.
  - **À-la-carte toggles** to add/remove individual modules (e.g. disable Prize Distribution → not charged).
- **White-Label = paid add-on** (any tier). Without it: KhundiConnect branding + KhundiConnect subdomain. With it: own logo, color palette, brand name, and own subdomain/custom domain (§15).
- **Billing cycles:** monthly / yearly (yearly discounted).
- **AI Insights:** included as a *capability* (bring-your-own-key), so no per-use vendor cost; can be a tier feature or free.
- **[DECIDE AT BUILD]** exact prices (PKR-denominated; community/non-profit-friendly pricing recommended).

---

## 15. White-Label Architecture (Paid Add-On)

White-Label is a **paid, per-tenant module** (a module flag like any other — see §5.4). It has two distinct states:

### 15.1 Default (White-Label NOT purchased)
- Tenant runs under **KhundiConnect branding** — the KhundiConnect logo and default theme are shown throughout the portal, login page, PWA, and outbound notification sender name.
- Tenant is hosted on a **KhundiConnect-provided subdomain**, e.g. `{tenant}.khundiconnect.com` (auto-provisioned by the vendor at tenant creation).
- Tenant **cannot** change logo, brand name, color palette, or domain.

### 15.2 White-Label Enabled (paid)
When the vendor enables the White-Label module for a tenant (billed), the tenant gains, from **Settings**:
- **Own logo upload** — replaces the KhundiConnect logo everywhere in their portal (and optionally PWA icon).
- **Own brand name** — replaces "KhundiConnect" in their portal headers/titles.
- **Color palette customization** — primary/secondary/accent theme tokens editable from settings; applied at runtime across dashboards, charts, and login page.
- **Own subdomain or custom domain** — either a chosen subdomain or a full custom domain, mapped via Cloudflare with auto-TLS.
- **Login-page customization** reflecting their identity.
- Optional white-label on **PWA** (name/icon) and **outbound notification** sender name.

### 15.3 Mechanics
- Per-tenant **branding record** (logo ref, brand name, color tokens JSON, login customization). Populated/editable **only** when the White-Label flag is on; otherwise the system falls back to KhundiConnect defaults.
- **Theme tokens injected at runtime** from the branding record (or KhundiConnect defaults if not white-labeled).
- **Domain/subdomain mapping** handled at the tenant-resolution layer (§5.2): default KhundiConnect subdomain always works; custom subdomain/domain activates only with White-Label, via Cloudflare mapping.
- Disabling White-Label (non-payment/downgrade) cleanly **reverts to KhundiConnect branding + KhundiConnect subdomain** without data loss (branding record retained but not applied).

---

## 16. Mobile App Strategy
> **v3.1 PWA (build-steps 54+).** Full installable PWA: install banner (authenticated pages only), **white-label per Khundi** via a dynamic per-subdomain manifest + per-Khundi icon/splash/theme, smart update prompts, offline app-shell (never cache `/api`), and a 90-day refresh session with silent refresh. The architecture is kept **realtime-ready** (announcements/attendance/poll surfaces designed so a Socket.io layer can drop in without rework), but the socket layer itself is **deferred** (built when live results/dashboards become a priority).


- **Phase 1: PWA** — single codebase, installable, offline census, push notifications, camera QR scanning, card wallet. No app stores.
- **Phase 2 (only if needed):** native wrapper (Capacitor) reusing the PWA, for store presence / deeper device features.
- Member features identical across web and PWA; responsive-first.

---

## 17. Security Best Practices

- **Tenant isolation:** PostgreSQL **RLS** + app-layer `tenant_id` injection; cross-tenant access only via explicit, audited, permissioned path.
- **Auth:** JWT access+refresh; OMJ-card login + OTP/password; session management; optional 2FA for admin roles; optional IP restriction & time-limited roles.
- **Secrets:** CNICs and all API keys (messaging, AI) **encrypted at rest**, masked in UI, never logged or placed in URLs/query strings.
- **Authorization:** dual-gate (module-enablement + scoped permission) on every request.
- **Audit:** immutable log of sensitive actions (impersonation, disbursements, approvals, financial entries, AI generation, suspensions, permission/module changes).
- **Privacy:** anonymize before LLM calls; never compile/leak personal data cross-tenant; confidential welfare data restricted to authorized roles.
- **Input validation** (class-validator), rate limiting (Redis), CSRF/XSS protections, secure file upload scanning.
- **Backups:** automated, with documented retention & restore policy.
- **Impersonation safeguards:** explicit permission, always audited, visible banner during impersonation.

---

## 18. Accounting — Dual-Mode Detailed Design
> **v3.1 ADDITIONS (build-steps 54+).** Surface the existing **hierarchical chart of accounts** (Pro) and clean up reconciliation. Add **Funds + Donations**: OMJ-standard core funds — **Business Aid, Ration Aid, Zakat, General, Donations, Dowry, Educational** — plus Khundi-authored custom funds; a **lightweight donor record** for non-member donors; the fund model is kept **v4-forward-compatible** (the cross-org ledger is the same domain at a higher altitude). Every money path posts a balanced entry; corrections are reversal-not-edit. Bereavement adds per-record **expense lines + invoices** that post here.


**Principle: one double-entry engine underneath; mode only changes the interface and how much the user enters.**

- **Always record full double-entry** (journal_entries + journal_lines) regardless of mode. There is **no "simple" data model** — only a simple *interface*.
- **Simple mode:** user enters "Received Rs 1500 — Lawazim from Saad" / "Paid Rs 15000 — Bereavement". System **auto-assigns** a default account/head behind the scenes and writes proper debit/credit. User sees only a **cashbook**: in / out / balance.
- **Detailed mode:** same records **exposed** with heads of accounts, ledgers, debit/credit, bank accounts, statements; user can assign heads, reconcile banks.
- **Switching Simple → Detailed (even after a year):** works perfectly because detail was always recorded. Past entries appear correctly against default heads; user may optionally re-categorize generic defaults into specific heads. **Nothing is lost or reconstructed.**
- **Switching Detailed → Simple:** just hides complexity; cashbook (in/out/balance) is always derivable.
- **Seeded default chart of accounts** (so Simple-mode auto-categorization and Detailed-mode start both work out of the box): Income — Lawazim, Donations, Sponsorships, Event Collections; Expense — Bereavement, Welfare, Prize Distribution, Get-together, Printing/Cards, General.
- **Lawazim integration:** member payment → income entry (full, no split); OMJ remittance → separate expense entry against OMJ receipt. **No auto-deduction/transfer between Khundi and OMJ — ever.**
- **Outputs:** cashbook, ledger, receipts, vouchers, bank statements, defaulters, financial reports; export PDF/Excel/CSV; accounting audit log.

---

## 19. OMJ Data Sync — Detailed Design

- **Endpoint base URL** lives in **env** (platform-level, same OMJ API for all). **API key** entered **per-tenant** in Settings.
- **Two fetch triggers:** (1) **Manual "Fetch Now"** button per tenant; (2) **Daily cron** iterating all tenants with a valid key.
- **Fetch, not sync:** one-directional pull (OMJ → Khundi). No write-back, no continuous reconciliation. OMJ is upstream source of truth; Khundi takes a copy.
- **Matching:** **CNIC primary**, **OMJ Card Number fallback**. No match → treat as **new**.
- **Conflict handling (never silent overwrite):** results go to a **permission-gated review queue** (Khundi Management with rights). For each record: `unchanged` (pass), `conflict` (same person, differing fields → old-vs-new review: accept OMJ / keep local / merge field-by-field), `new` (add), `local-only` manual member with no CNIC/card match → **never touched**.
- **Cron resilience:** per-tenant try/catch — one failed/expired key doesn't break the run; log per-tenant `last_fetch_status`. **Stagger calls** to be gentle on OMJ's API (avoid spikes/throttling/revocation).
- **Queue hygiene:** next cron **updates an existing pending conflict** rather than stacking duplicates.
- **Visibility:** Settings shows **last successful fetch time** + **status** (success/failed/key invalid) so a volunteer admin can self-diagnose.

---

## 20. Revenue Model

- **Primary:** per-tenant SaaS subscriptions (tiered + à-la-carte modules), monthly/yearly, PKR-denominated, community-friendly.
- **Add-ons:** custom messaging-provider integrations; custom domain; premium support; data-migration assistance (from registers/Excel).
- **AI:** bring-your-own-key keeps vendor AI cost ~zero; optionally monetize a vendor-provided AI tier later.
- **Expansion revenue:** onboard more Khundis; later other Memon Jamats / communities (same platform).
- **Cost control:** tenant bears messaging + AI costs; vendor's main costs are infra (shared, scales well due to single-DB multi-tenancy).

---

## 21. Roadmap — MVP / Phase 2 / Phase 3

### MVP (launch-critical)
- Multi-tenant core, tenant provisioning, subdomains, RLS isolation.
- Dynamic roles & permissions (dual-gate); Khundi Super Admin.
- Member management + family graph + households; derived voting eligibility.
- OMJ data sync (manual + cron, conflict queue).
- **Census (flagship): family tree, crowdsourced entry, approval, live demographics + snapshots, economic/welfare fields, progress.**
- Lawazim/fee engine (per-member-per-year, defaulters, OMJ remittance record).
- Dual-mode accounting.
- **AI Insights (Khundi + OMJ, bring-your-own-key, cached, scoped).**
- Notifications (Twilio/Infobip adapters, communication log).
- Reports & animated dashboards (3 altitudes).
- PWA member self-service (card login, census entry, profile/fee view, notifications).
- White-label (logo/colors/brand/subdomain).
- Vendor + OMJ super-user layers (impersonation audited, cross-Khundi oversight).
- Module toggles + basic billing.
- Urdu-first UX, audit log, backups.

### Phase 2
- Welfare case management (confidential) — *can be MVP if priority allows.*
- Bereavement & death/grave register.
- Prize distribution (optional module).
- Event management + QR attendance + MoM.
- Document vault; card system (digital wallet + print).
- Online fee payment gateway (member self-pay).
- Custom domains at scale; more messaging/AI providers.
- Voice/photo census input.
- Native app wrapper (if needed).

### Phase 3
- Expansion to other Memon Jamats / communities (templated onboarding).
- Advanced AI (predictive welfare, demographic forecasting, anomaly detection, churn/engagement).
- Khundi → OMJ census data push (if OMJ adopts this as upstream).
- Advanced BI (custom report builder, scheduled exports, external BI API).
- Self-hosted / Docker delivery option for large communities.

---

## 22. Risks & Solutions

| Risk | Solution |
|---|---|
| Low-literacy users struggle | Urdu-first, icon-driven, minimal typing, voice/photo input, progressive disclosure, simplicity-first principle |
| Cross-tenant data leak | PostgreSQL RLS + app-layer tenant injection + explicit audited cross-tenant path |
| OMJ API instability / key revocation | Manual + resilient staggered cron, per-tenant failure isolation, manual entry fallback always available |
| AI cost blowout | Bring-your-own-key + on-demand cached generation + staleness flag (no auto-regen) |
| Accounting upgrade pain | Single double-entry engine underneath; modes are views — retroactive switch is non-destructive |
| Volunteers churn / data entry burden | Crowdsourced census (members self-enter), member self-service profile updates, approval gates |
| Sensitive data exposure (CNIC, welfare) | Encryption at rest, masking, role-restricted confidential modules, anonymized AI |
| Auto-actions causing real-world harm | Human-in-loop everywhere: no auto-dissolution/suspension/transfer/overwrite |
| Scaling to 100+ tenants | Single-DB multi-tenancy, Redis caching, code-split SPA, background-job queue, horizontal scaling |
| Onboarding friction | Vendor provisions one Super Admin; data migration assistance; OMJ fetch to pre-populate members |

---

## 23. Future AI Features (Beyond MVP)
- Predictive **welfare prioritization** (who will need help, not just who does now).
- **Demographic forecasting** for community growth planning.
- **Anomaly detection** in finances and membership.
- AI-assisted **census data cleaning** & duplicate detection.
- Natural-language **"ask your data"** querying (gated/rate-limited for cost).
- AI-generated **plain-Urdu** reports & recommendations for councilors.

---

## 24. Product Name (Confirmed)

**KhundiConnect** — confirmed product/brand name. Used throughout the platform by default; replaced by a tenant's own brand only when that tenant holds the paid White-Label module (§15).

The name is intentionally community-neutral (not OMJ-specific), supporting Phase-3 expansion to other Memon Jamats and communities on the same platform.

---

## 25. How to Scale to 100+ Khundis
- **Single-DB multi-tenant** keeps per-tenant overhead near zero and makes cross-Khundi analytics native.
- **Self-service-ish onboarding:** vendor creates tenant + one Super Admin; OMJ fetch pre-populates members; tenant configures the rest.
- **Redis caching + TanStack Query** for fast UX at scale; **BullMQ** offloads cron/notification/AI/export work.
- **Horizontal scaling** of stateless NestJS + Next.js behind a load balancer; Postgres read-replicas if needed.
- **Per-tenant module flags** mean one codebase serves all configurations.
- **Templated white-label** so each new Khundi is branded in minutes.

## 26. How to Launch Commercially
1. **Pilot with your own Khundi (Jafrani-B)** — dogfood, refine UX with real volunteers.
2. **Onboard the Khundis already asking** (Toberiya-A, Kath-A, etc.) as early adopters; gather feedback.
3. **Win OMJ endorsement** — the OMJ oversight layer + Jamat-wide AI insights make the platform valuable to the supreme body; OMJ backing drives adoption across all Khundis.
4. **Data migration service** from registers/Excel to reduce switching friction.
5. **Tiered, community-friendly pricing**; free/discounted census tooling to seed the network effect (more Khundis on census = more valuable to OMJ).
6. **Train a few "power" volunteers** per Khundi; lean on simplicity-first UX for the rest.
7. **Expand** to other Memon Jamats / communities once proven (Phase 3), using the same platform with neutral branding.

---

## 27. Build Order for the Agentic AI (CANONICAL SEQUENCE)

> **This is the canonical numbering for the project.** The agent builds **one build-step at a time, in this order.** Branch names use the build-step number: `feat/NN-shortname` (zero-padded). 🛑 = mandatory human checkpoint (agent prints `[CHECKPOINT]` and pauses for APPROVE/REJECT). See AGENT.md for the full checkpoint rule (the fixed list below PLUS a general rule).

| # | Build-step | Branch | Checkpoint |
|---|-----------|--------|-----------|
| 1 | Foundation: monorepo, NestJS + Next.js + Postgres + Prisma + Redis; Docker; env config (incl. OMJ endpoint placeholder) | `feat/01-foundation` | — |
| 2 | Multi-tenancy core: tenants, subdomains, **RLS**, tenant context middleware | `feat/02-multitenancy` | 🛑 |
| 3 | Auth & Identity: OMJ-card login, OTP/password, JWT, sessions | `feat/03-auth` | 🛑 |
| 4 | **Dynamic Permission Engine + dual-gate guards** (build early — everything depends on it) | `feat/04-permissions` | 🛑 |
| 5 | Module-enablement (module_flags) + Vendor tenant provisioning + one Super Admin per tenant | `feat/05-module-flags` | — |
| 6 | Member management + family graph + households + derived eligibility | `feat/06-members` | — |
| 7 | OMJ data sync (manual + cron + conflict queue) | `feat/07-omj-sync` | 🛑 |
| 8 | Census (snapshots, entry, approval, family tree, demographics, progress) | `feat/08-census` | — |
| 9 | Lawazim / fee engine | `feat/09-lawazim` | — |
| 10 | Dual-mode accounting | `feat/10-accounting` | — |
| 11 | Notifications (adapter + Twilio/Infobip) + communication log | `feat/11-notifications` | — |
| 12 | AI Insights (adapter + cache + scope/visibility + permissions) | `feat/12-ai-insights` | — |
| 13 | Reports & animated dashboards (3 altitudes) + exports | `feat/13-reports` | — |
| 14 | PWA member self-service | `feat/14-pwa` | — |
| 15 | White-label + billing + module toggles UI | `feat/15-whitelabel` | — |
| 16 | OMJ super-user oversight UI + Vendor analytics | `feat/16-oversight` | — |
| 17 | Audit, backups, i18n/Urdu polish, accessibility | `feat/17-audit-i18n` | — |
| 18 | Phase-2 modules (welfare, bereavement/graves, prize, events, MoM, docs, cards, QR) | `feat/18-phase2` | — |

> **Build-steps 1–18 are FEATURE-COMPLETE (the MVP).** They shipped the full module surface but at MVP depth — several modules are intentionally thin. **Build-steps 19–30 are the v2 DEPTH PASS:** they correct the data model and re-build the thin modules to professional, dynamic depth, plus add three new cross-cutting subsystems (form-builder, Simple/Pro mode, certificate canvas). The agent classifies each as **FEATURE** (new subsystem) or **FIX** (deepening shipped behaviour) per its WORK-TYPE rule. This is the priority work before any production promotion.

### v2 Depth Pass (build-steps 19–30)

| # | Build-step | Branch | Checkpoint |
|---|-----------|--------|-----------|
| 19 | **Data-model correction** — `is_okhai_by_birth` replaces `member_type`/`is_honorary`; drop Khundi/Group-code from members (read from tenant); add `marital_residence` + husband-side facts (non-Okhai husband, §2.3); all categories derived; membership-vs-census split. Migration + RLS-preserving. | `feat/19-datamodel` | 🛑 |
| 20 | **Shared Form-Builder engine** (new subsystem §32) — locked-core + custom fields, per-field option sets with light/detailed styles, multi-step, validation; consumed by Census + Prize (+ future). | `feat/20-form-builder` | 🛑 |
| 21 | **Simple/Pro mode framework** (new subsystem §33) — per-user saved preference per module, lossless, role-aware ceiling; the contract every later UI module implements. | `feat/21-mode-framework` | 🛑 |
| 22 | **Global UI/UX layer** (§34) — human-readable label formatting everywhere (no raw enums), Table/Card toggle on every listing, per-Khundi column chooser in settings, professional list components, design-bar pass. | `feat/22-ui-layer` | — |
| 23 | **Members deepen** — full Add-Member field set (minus dropped fields), child handling, female/non-Okhai-husband form path, professional list (table+card), professional family-tree visualization. Simple/Pro faces. | `feat/23-members-v2` | 🛑 |
| 24 | **Census deepen** — form-builder-driven, multi-step public UX, income/education/profession option sets, finalized membership-vs-census split, demographics/insights wiring. Simple/Pro faces. | `feat/24-census-v2` | 🛑 |
| 25 | **Lawazim deepen** — liable-only register, carry-forward arrears ledger + running balance, remove OMJ-100 from collection screen (remittance → expense side), professional dues/partial-payment UX. | `feat/25-lawazim-v2` | 🛑 |
| 26 | **Accounting deepen** — full double-entry depth; professional Simple cashbook (clean in/out + expenses) and Pro ledger/vouchers/receipts/bank-rec/statements/reports. Core module — must be perfect. | `feat/26-accounting-v2` | 🛑 |
| 27 | **Welfare deepen** — fund source (Zakat/General/Endowment), configurable case types/sub-types, structured fields, document upload, inquiry/investigation report step. Simple/Pro faces. | `feat/27-welfare-v2` | 🛑 |
| 28 | **Events deepen** — datetime/budget/RSVP/attendance(human+QR)/structured resolutions/archive; ceremony↔event linkage. Simple/Pro faces. | `feat/28-events-v2` | — |
| 29 | **Prize deepen** — form-builder forms with announce/open/close windows, configurable categories+criteria (academic + Hifz milestones), submission by members & controllers, approve→assign, position naming, **certificate canvas + bulk generate**, Excel eligibles export + printable announcement list. | `feat/29-prize-v2` | 🛑 |
| 30 | **Documents & Cards deepen** — real R2 file upload, metadata/search, role-ACL editor UI; visual rendered membership/QR card, templates, print + wallet (no raw payloads). Simple/Pro faces. | `feat/30-docs-cards-v2` | 🛑 |

### v2 Final-Surfaces Polish Pass (build-steps 31–37) — COMPLETE

Applied the same depth/professional/Simple-Pro/label/list standards to every remaining surface.

| # | Build-step | Branch | Checkpoint |
|---|-----------|--------|-----------|
| 31 | **Dashboards deepen** (3 altitudes) — animated KPIs/charts, Khundi + OMJ + Vendor; cross-Khundi oversight read views. | `feat/31-dashboards-v2` | — |
| 32 | **Member PWA deepen** — card-gated self-service: profile/update-requests, family census entry, fee view, card wallet, notifications. | `feat/32-member-pwa-v2` | 🛑 |
| 33 | **AI Insights deepen** — adapter + cache + scope/visibility + permissions; Khundi + Jamat insights (bring-your-own-key at the time). | `feat/33-ai-insights-v2` | 🛑 |
| 34 | **OMJ Sync deepen** — manual + cron fetch, CNIC/card matching, conflict review queue (no silent overwrite). | `feat/34-omj-sync-v2` | 🛑 |
| 35 | **Notifications deepen** — adapter + Twilio/Infobip, templates, communication log. | `feat/35-notifications-v2` | 🛑 |
| 36 | **Billing & White-Label deepen** — plans/subscriptions/module toggles; branding/logo/colors/subdomain. | `feat/36-billing-whitelabel-v2` | 🛑 |
| 37 | **Bereavement/Graves + Audit deepen** — death/grave register surface; polished immutable audit viewer. | `feat/37-bereavement-audit-v2` | 🛑 |

> **v2 PROGRAM COMPLETE (19–37):** data-model correction (19), three subsystems (20–22), eight module deepens (23–30), seven final-surface polishes (31–37). Every shipped surface at professional depth with Simple/Pro faces, human labels, global UI layer.

### v3 — Experience + Capability Pass (build-steps 38–53)

v3 has two phases. **v3-A (38–46) ships first** (the experience layer — it unblocks showing the team); **v3-B (47–53) follows** (new capability). The cross-org welfare ledger is deferred to **v4** (see §27.1) but stays in KhundiConnect; v3's data model is built forward-compatible with it. Tenant isolation remains ABSOLUTE throughout v3 — no cross-org seam is opened.

**v3-A — Experience layer**

| # | Build-step | Branch | Checkpoint |
|---|-----------|--------|-----------|
| 38 | **Global IA + design system** — heavy modules → tabs/sub-nav (no long-scroll); shared date/currency/number components (kill US `mm/dd/yyyy`); empty states; density; Simple/Pro toggle → top bar (only on applicable pages). | `fix/38-ia-design-system` | 🛑 |
| 39 | **Modals/selectors/uploads** — all create/add/edit → modals/drawers (no inline-expand); member autocomplete (scales to 1000s) + role/value searchable multi-select; file **upload** controls everywhere (no R2/key/URL shown). | `fix/39-modals-selectors-uploads` | 🛑 |
| 40 | **Enum-label sweep + de-jargon copy** — no machine value (`incomeBand`, `below_15K`, `relation_to_head`…) ever shown; strip all §-citations / "never auto" / internal notes from user copy → one short human line per module/section. | `fix/40-label-sweep-copy` | 🛑 |
| 41 | **RTL layout fix + full Urdu translation coverage** — clean RTL across all surfaces + primitives; zero English leakage in Urdu mode (en/ur key parity). | `fix/41-rtl-urdu` | 🛑 |
| 42 | **Edit/Delete everywhere applicable** — add edit/delete to create-only records; hard vs soft-delete rule; audit-log + posted-financial stay immutable. | `fix/42-edit-delete` | 🛑 FIXED |
| 43 | **Settings consolidation + Role editor UI** — Branding, Display Mode, List Columns, providers, storage under one Settings area; the missing create/edit-roles screen. | `feat/43-settings-roles` | 🛑 FIXED |
| 44 | **Correctness fixes** — tenant-name bleed; audit noise (stop logging token refreshes); member hygiene (no expired/inactive/married-out in active pickers/RSVP); fix all-zeros Jamat insight aggregation; minutes-only-when-completed. | `fix/44-correctness` | 🛑 |
| 45 | **Family tree → modal/focused view** + relationship lenses (By Birth / By Marriage / Both) + household/living-arrangement grouping (substrate for 48 dedup). | `feat/45-family-tree` | 🛑 |
| 46 | **Storage rework** — Google Drive backend (ENV-selectable, behind `StorageService`); per-Khundi MB quota (enforced); auto-compression + dimension reduction before storage. | `feat/46-storage-drive-quota` | 🛑 FIXED |

**v3-B — New capability**

| # | Build-step | Branch | Checkpoint |
|---|-----------|--------|-----------|
| 47 | **Bereavement as its own module** — extract from Welfare; deceased register; OMJ **and external** graveyards; **remove Bus Service** (Jamat-provided); bereavement payment → accounting. | `feat/47-bereavement-module` | 🛑 |
| 48 | **Welfare deepening** — per-fund available balances (Zakat/Endowment/General, Zakat never mixed); recipient + family welfare history; household-aware dedup (the ration case, human prompt not auto-reject); enforce every-payment→accounting; optional per-Khundi **Sharia Compliance form**; **CNIC anchor** (v4-forward-compatible, isolation kept absolute). | `feat/48-welfare-deep` | 🛑 FIXED |
| 49 | **Centralize AI Insights + OMJ Sync at OMJ** — keys/config move to OMJ altitude; OMJ generates Jamat-wide + per-Khundi insights (each Khundi receives its own); Khundi triggers sync key-less; review queue + no-silent-overwrite preserved. | `feat/49-omj-centralize` | 🛑 FIXED |
| 50 | **Prize recognition awards** — categories beyond academic/Hifz (most senior, most active…); Khundi-**assigned** recognition flow (no application cycle) alongside application-based prizes. | `feat/50-prize-recognition` | 🛑 |
| 51 | **Form Builder per-instance binding** — bind a specific form to a specific Census round / Prize ceremony (2026 ≠ 2025); mark a form default. | `feat/51-form-binding` | 🛑 |
| 52 | **Notifications channels & providers** — PWA push + SMS + WhatsApp; pluggable providers (generic REST, Twilio, Infobip, WhatsApp Business API; separate SMS/WhatsApp APIs); provider config in Settings. | `feat/52-notifications-channels` | 🛑 FIXED |
| 53 | **Census report generator + new fields** — shareable chart-driven community report (the Gatta-G deck); add home ownership, transport, staged education (in/out + retention), structured needs/asks; **remove the welfare-need flag** → derive a vulnerability indicator → welfare outreach-suggestion view (human prompt, never auto-case). | `feat/53-census-report` | 🛑 FIXED |

> **v3 checkpoint guidance:** 42, 43, 46, 48, 49, 52, 53 are 🛑 FIXED (money / identity / access-control / storage-backend / cross-tenant generation). 38–41, 44, 45, 47, 50, 51 are 🛑 by the general rule. The general checkpoint rule still applies on ANY step touching schema/RLS/auth/permissions/production.

> **v3 ordering (within phases):** 38 is the foundation (shell/components/top-bar mode) — 39–46 assume its primitives. 40 before 41 (translate final human copy, not soon-to-be-rewritten strings). 45's household grouping is a deliberate dependency for 48's welfare dedup. 44's insight-aggregation fix precedes 49's move of generation to OMJ. 51's per-instance forms feed 53 (round-bound census fields) and 50 (application-based categories). 46's storage abstraction backs 39's uploads and 53's report export.

> **General checkpoint rule (applies on ANY build-step):** the agent also prints `[CHECKPOINT]` whenever it changes the data model, touches RLS/`tenant_id`, changes auth or permissions, or does anything production-affecting — not only on the four fixed steps above. Err toward more checkpoints. (Full rule in AGENT.md.)

> **Each module must ship with:** its data model + migrations + RLS, its permission keys, its module-flag gate, its API, its SPA UI (animated, Urdu-ready), and its audit hooks. Build, test, and verify each module before the next.

### v3.1 — Experience-overhaul + new-capability program (build-steps 54+)

> **v3.1 is one large foundation-first program** building on top of v3, derived from a 40-concern + 5-new-capability review of the deployed app (see `specs/README-v3.1.md` for the full decision register). It is sequenced in **six waves**: (1) shared UI primitives, (2) member model + identity, (3) form engine + census, (4) finance + community modules, (5) communication + access + polling, (6) platform experience. Foundations land before the features that sit on them. The cross-org welfare ledger stays deferred to **v4** (§27.1); v3.1 keeps tenant isolation ABSOLUTE.
>
> **Checkpoint mode for v3.1 — CONTINUOUS DEVELOPMENT.** Unlike v2/v3 (which used fixed manual stops), v3.1 runs continuously: the agent ends **every** build-step with `[CHECKPOINT]` (no `[FIXED_CHECKPOINT]`), the controller auto-approves, merges the branch to **staging**, deploys staging, and proceeds to the next step. The build stops ONLY when a spec is missing (`[HUMAN_REQUIRED]`), an infra problem is not code-fixable, or after repeated failure. **Production promotion stays manual (operator-only); staging is the testing gate.** Data-model / RLS / auth / permission changes are still flagged in the step summary, but they do not halt the loop.

The exact build-step numbers (54+) and per-step branch slugs are assigned in the v3.1 specs. The wave ordering:

| Wave | Theme | Work-items |
|------|-------|-----------|
| 1 | **Core UI primitives & cross-cutting** | Shared `DataList` redesign (actions-first column / card-footer actions, responsive colored-icon↔label, fail-loud on bad list key); per-USER column preferences from the table header; dynamic/fuzzy/collapsible filters (auto client/server by row-count, debounced); toast system + inline field-validation; validation/masking layer + phone library (CNIC/OMJ masks, two-level validation); modal/keyboard fix (ESC everywhere, Cmd/Ctrl+Enter, rush-time shortcuts); **mode-gating fix (admin/config NEVER mode-gated, only permission-gated)** + human-labels/Urdu pass + image-ref→URL fix + repo hygiene. |
| 2 | **Member model & identity** | **Marriage-linkage model** (Married-In derived by relationship; divorce → Inactive after confirmation, history preserved; **remove "honorary" entirely** → gender split); **member timeline** (auto + manual events) + role-gated audited **controller notes**; members info-boxes; card system (OMJ official / Khundi optional / **OMYS number**) + certificate/card canvas (bg + font/size/color); family-tree fix (children→marriage lens; two-lineage no-doubling). |
| 3 | **Form engine & census** | Advanced **form builder** (inline options, auto-slug/no keys shown, no enums to users, radio/checkbox/member/searchable types, **multi-level conditional/cascading logic**); **taxonomy governance** (OMJ standard + Khundi-local-immediate + OMJ review queue with promote / merge-alias / re-map); **census redesign** (auto-fill, marriage/household fill, full Gatta-G field set, unify "Other"); polished **census PDF report**; **instance/season pattern** (Census rounds + Prize ceremonies: active-latest + archived-viewable + cross-instance report). |
| 4 | **Finance & community modules** | Accounting (surface hierarchical chart of accounts in Pro + reconciliation cleanup); **Funds + Donations** (OMJ core funds: Business Aid / Ration Aid / Zakat / General / Donations / Dowry / Educational + Khundi custom; lightweight donor record; v4-forward-compatible); **Bereavement** (expense-lines per record + invoices → accounting; mark member expired + notify OMJ; janaza announcement; eligibility flag-not-block; drop grave-issued-date & relation-non-member field); **Events/attendance** (multi-type + agenda; focused fast roll-call surface — bulk, multi-field lookup incl. debounced partial CNIC, optional QR; inline lawazim collection; announce-prompt-on-create + soft-reminder); Prize fixes (ceremony-scoped applications, reopen, category selectability); **graveyard registry** (OMJ-managed, merge-on-remove). |
| 5 | **Communication, access & polling** | **Notification/announcement engine** (media, active-channels-only, multi-member/multi-role + **custom audience builder**, per-type channel routing, per-channel templates, scheduler fix, push tenant-scope verify); **announcement landing pages** (login-gated + tenant-eligible + public-shareable + link shortening + public-content restrictions) + **OMJ-level cross-Khundi announcements**; Access (My-Access readability; user onboarding; **password-primary credentials** + auto-generate + WhatsApp `wa.me` handoff; OTP optional only where a provider is configured; scoped committee roles); **CSV bulk member import** (+ sample, server validation, error report); **Polling & Elections module** (full: nominations, secret ballot, configurable winning rules, certification; Khundi-configurable per-poll eligibility; also decision-polls). |
| 6 | **Platform experience** | Full **PWA** (install banner, white-label per-subdomain manifest/icon/splash, smart updates, offline, 90-day session, **realtime-ready but Socket.io deferred**); **Reporting module** (dedicated report library — member directory w/ filters, financial statements, lawazim defaulters, welfare, census-filtered e.g. doctors, printable per-member family tree, etc.; hybrid server-side PDF + Excel; RTL/Urdu). |

> **v3.1 deferrals (to v4 / later):** the cross-org CNIC-anchored welfare ledger (§27.1); a realtime/Socket.io layer (architecture kept realtime-ready in v3.1); a first-class **Committees** entity (v3.1 uses scoped roles; committee-chairman elections via the polling module is a natural later add); true row-level "own record only" volunteer scopes (v3.1 uses task-limited roles — row-level scoping touches the safety-critical permission engine and warrants its own design round).

### 27.1 v4 — Cross-Organization Welfare Ledger (DOCUMENTED, NOT BUILT IN v3)

Deferred from v3 by decision; **stays within KhundiConnect** (same product). Build-steps **54+**. **Do not build any of this during v3 (38–53).** Tenant isolation remains ABSOLUTE through v3; v3 only makes the data model forward-compatible (CNIC anchor, household grouping, per-member/household welfare history — build-steps 45/48).

**Problem.** Under OMJ, multiple support organizations serve all Khundis — e.g. **Okhai Memon Youth Services (OMYS)** (dowry/Jehaz, monthly ration, education), **Humaira Bai Foundation**, **Okhai Memon Anjuman (OMA)**, and OMJ itself. A member can collect from each separately and exceed/fabricate their stated need (e.g. needs Rs 5 lacs, collects 6.5 by going org-to-org), invisibly to any single organization.

**Concept.** A centralized, **CNIC-anchored** cross-organization welfare/support ledger: when any organization gives a member funds, it is recorded centrally, so any **authorized** organization (and the member's Khundi) can see the member's — and their family's — support history **before approving**, with a **need-vs-collected** tally per request.

**Altitude (refined).** GeekAxon (Pvt) Ltd (Vendor, sees all) → **OMJ** (sees all; **governs** what each support-org may see) → **support organizations** (scoped by OMJ; a **Welfare-only module**, not full accounting — they manage their own welfare funds/cases) → **Khundi** (tenant).

**Key components (need a dedicated design round).** (1) Support orgs as tenants with a **Welfare-only** module. (2) **CNIC** (tokenized/hashed) as the cross-org anchor; OMJ Card kept. (3) A **governed shared welfare-history domain** above tenants (deliberately designed — the opposite of strict per-tenant RLS — read by authorized orgs through OMJ-scoped, category-scoped, **dignity-preserving** views). (4) **Need-vs-collected** tracking per request. (5) **OMJ-governed visibility** (the build-step 49 "OMJ-governs / Khundi-consumes" pattern is the template). (6) **Sharia Compliance** (built per-Khundi in build-step 48) extended as a cross-org deterrent layer.

> **Recommendation:** a design-first architecture round (produce the cross-org data-domain design + ARCHITECTURE additions) before any build, given the isolation-model change and the CNIC cross-org privacy/legal weight.

---

## 28. Server & Infrastructure Architecture (Oracle Free Tier + aaPanel)

This mirrors the proven GarageBrainPro deployment pattern. The operator (Saad) operates the VPS side with step-by-step guidance; the agent prepares everything else.

### 28.1 Target Server
- **Oracle Cloud Free Tier — ARM (Ampere A1) VPS**, separate Oracle account dedicated to KhundiConnect (do NOT reuse other instances).
- Recommended shape: up to **4 OCPU / 24GB RAM / ~200GB** (Free Tier ARM allowance) — comfortably runs the full stack.
- **aaPanel** for server management — GUI for Nginx, databases, SSL, cron, file management, so a non-technical operator can manage it.
- **Ubuntu 22.04+**, **Node v22** (KhundiConnect targets v22 from the start).

### 28.2 Runtime Topology (single VPS, multiple processes via PM2)
PM2 `ecosystem.config.js` runs these apps:
1. **kc-api** — NestJS backend (REST API). *(exists from early build-steps)*
2. **kc-web** — Next.js frontend (SSR/SPA server). *(exists from early build-steps)*
3. **kc-worker** — BullMQ worker (cron fetches, notifications, AI generation, exports). *(arrives with notifications/cron build-steps)*
4. **kc-scheduler** — cron/queue scheduler (daily OMJ fetch, reminders). *(arrives with notifications/cron build-steps)*
- **PostgreSQL** and **Redis** run as services (via aaPanel or Docker).
- **Nginx** reverse-proxies: `kc-web` for app traffic, `kc-api` under `/api`, with WebSocket support.
- **Object storage:** Cloudflare R2 (or local uploads dir with backup) for documents, card images, census attachments.

> `deploy.sh` restarts only the PM2 processes that currently exist and gracefully skips absent ones — so early phase (api + web) and later phase (all four) both work with **no script edits.**

### 28.3 Domains, Subdomains & Cloudflare
- Base platform domain on **Cloudflare DNS**.
- **Wildcard subdomain** (`*.khundiconnect.com`) → VPS, so every tenant's default subdomain resolves with **one wildcard DNS record + wildcard TLS cert**.
- **Custom domains** (paid White-Label tenants) added individually in Cloudflare + Nginx + cert, as onboarded.
- Cloudflare set to **DNS-only (no proxy) during setup/staging** (GarageBrainPro convention); enable proxy/CDN later if desired.

### 28.4 Environments
- **Staging:** `staging.khundiconnect.com` — agent deploys here first; all testing + dummy data here.
- **Production:** base domain + wildcard — promoted **only** on the owner's Telegram command (see AGENT.md). Agent never triggers it.
- Separate PostgreSQL databases for staging vs the live environment. **Never test with live data.**

### 28.5 Environment Variables (.env — never committed)
A `.env.example` (committed, **placeholders only**) documents every key; the real `.env` lives **only on the VPS**:
```
NODE_ENV, PORT_API, PORT_WEB
DATABASE_URL (postgres), REDIS_URL
JWT_ACCESS_SECRET, JWT_REFRESH_SECRET
OMJ_API_BASE_URL          # OMJ member-fetch endpoint (platform-level) — placeholder in example
ENCRYPTION_KEY            # for CNIC + API-key encryption at rest
R2_ACCOUNT_ID, R2_ACCESS_KEY, R2_SECRET, R2_BUCKET
PLATFORM_BASE_DOMAIN      # placeholder in example
NEXT_PUBLIC_PLATFORM_BASE_DOMAIN   # PUBLIC — inlined into the web bundle at BUILD time (must match PLATFORM_BASE_DOMAIN)
# NOTE: tenant messaging keys & AI keys are stored encrypted in DB, entered via UI — NOT here
```

**WEB BUILD-time vars vs API RUNTIME vars (permanent build-correctness rule).**
There are two distinct classes of env var and confusing them breaks deploys:

- **API RUNTIME** — read by the NestJS process at run time (PM2 env). All server
  secrets (`DATABASE_URL`, `JWT_ACCESS_SECRET`, `JWT_REFRESH_SECRET`,
  `ENCRYPTION_KEY`, `R2_*`, any API keys) are RUNTIME-only and **never** reach the
  browser.
- **WEB BUILD** — `NEXT_PUBLIC_*` vars are **inlined into the client bundle at
  `next build` time**, not read at runtime. They are **public** (shipped to every
  browser). `next build` runs with `apps/web` as cwd and reads `.env` from the
  **app directory**, NOT the monorepo-root `.env` — and turbo's strict env mode
  strips undeclared vars. So a plain `turbo run build` does **not** automatically
  bake a root-`.env` `NEXT_PUBLIC_*` value into the bundle. **`deploy.sh` (§28.6
  step 5b) writes an explicit ALLOWLIST of `NEXT_PUBLIC_*` values to
  `apps/web/.env.local` before building**, and the same vars are declared in
  `turbo.json`'s build-task `env` for cache correctness. **The allowlist is the
  security boundary**: only public vars are copied; a server secret must NEVER be
  added to it or exposed via a `NEXT_PUBLIC_*` name. `deploy.sh` grep-asserts the
  expected value landed in `.next/static` AND scans `.next/static` for leaked
  secret values, failing the deploy on either fault.

### 28.6 Deployment Flow — `deploy.sh` Specification (the agent authors the file during the build)

> This section is the **authoritative spec for `deploy.sh`.** The agent creates the actual `deploy.sh` in the private code repo during the build (per AGENT.md), implementing exactly this contract. `deploy.sh` targets the **staging/working environment**; the **live promotion is owner-Telegram-gated only** and is never an auto-trigger in this script. No `production`/`prod` strings anywhere in the file (the controller's safety fence blocks them). No secrets in the script — real values come from the VPS `.env` only.

**Shell contract:** `#!/usr/bin/env bash` with `set -euo pipefail`. Must pass `bash -n` (syntax) and run idempotently. Resolve `APP_DIR` from the script's own location and `cd` into it. Source the VPS `.env` if present (never committed).

**Ordered steps the script MUST perform:**

1. **Universal Node-detection block** — locate a valid `node`/`npm` and prepend it to `PATH`, checking these locations **in order**, preferring the highest version where a glob matches:
   - **nvm:** `~/.nvm/versions/node/*/bin`
   - **aaPanel:** `/www/server/nodejs/*/bin`
   - **system:** `/usr/local/bin`, then `/usr/bin`
   - **fallback:** whatever `node` is already on `PATH`; if none found, error and exit.
   - After detection, log `node -v` / `npm -v`. KhundiConnect targets **Node v22** — **warn (do not block)** on a version mismatch so deploys aren't broken by a minor drift.

2. **`git pull`** — fast-forward only (`git pull --ff-only`) on the current branch (a `feat/*` branch or `staging`). Log the active branch.

3. **`npm ci`** — clean install for the monorepo.

4. **Database migrations** — `npx prisma migrate deploy` (apply committed migrations only; **never** `migrate reset` or any destructive DB op — those are fence-blocked and forbidden).

5. **Fresh `BUILD_ID` before build** — set `BUILD_ID` to a fresh value every run (timestamp + short git SHA), export it, log it, then `npm run build`. (Matches the GarageBrainPro learning that a stale `BUILD_ID` causes broken builds.) This step has three build-correctness sub-steps, all **permanent rules**:

   - **5a. Purge stale build artifacts BEFORE building.** `nest build` uses `deleteOutDir:true`; combined with a leftover `*.tsbuildinfo` the incremental compiler sees "no source changes", emits **zero** files, yet `dist/` was already wiped — an exit-0 build with an **empty `dist/`** that crashes the API with `Cannot find module './config/env.validation'`. We removed `"incremental": true` from `apps/api/tsconfig.json`, AND deploy.sh deletes all `*.tsbuildinfo` (outside `node_modules`), `apps/api/dist`, and the turbo cache (`.turbo`, `node_modules/.cache/turbo`) before building, so a clean full compile and a no-stale-cache run are guaranteed every deploy. (This is the **incremental-no-op trap** — a permanent build rule alongside *prisma-generate-before-build* and *npm-ci-include-dev*.)

   - **5b. Bake allowlisted `NEXT_PUBLIC_*` into the web build.** Before `npm run build`, write an explicit ALLOWLIST (`NEXT_PUBLIC_PLATFORM_BASE_DOMAIN`, `NEXT_PUBLIC_OTP_LOGIN_ENABLED`, `NEXT_PUBLIC_MESSAGING_PROVIDER`) from the sourced `.env` into `apps/web/.env.local` (Next.js reads `.env` from the **app** dir at build, not the monorepo root — see §28.5). ONLY allowlisted public vars are copied; **no server secret is ever written there**.

   - **5c. Build-output assertions (fail the deploy, never restart PM2 onto a broken build).** After building: assert `apps/api/dist/main.js` AND `apps/api/dist/config/env.validation.js` exist; assert the `NEXT_PUBLIC_PLATFORM_BASE_DOMAIN` value appears in `apps/web/.next/static` (the redirect is silently dead otherwise); and scan `apps/web/.next/static` for the literal values of server secrets (`DATABASE_URL`, `JWT_*`, `ENCRYPTION_KEY`, `R2_*`) and abort if any leaked. Any failure exits non-zero **before** the PM2 restart.

6. **Graceful per-process PM2 restart** — use **`pm2 restart`** (NOT `reload`), per project convention. Iterate the process list `kc-api`, `kc-web`, `kc-worker`, `kc-scheduler`:
   - if `pm2 describe <proc>` succeeds → `pm2 restart <proc> --update-env` and log it;
   - if absent → **skip with a warning** (no-op), so the early phase (api + web only) and the later phase (all four) both work with **no script edits**.
   - If **none** of the `kc-*` processes are registered yet and an `ecosystem.config.js` exists → first-time `pm2 start ecosystem.config.js --update-env`.
   - Require `pm2` on `PATH` (error if missing); `pm2 save` at the end (non-fatal on failure).

7. **Graceful curl smoke test** — real health-check, **non-fatal until endpoints exist** so it never blocks legitimate early deploys:
   - `GET http://localhost:${PORT_API}/health` and `GET http://localhost:${PORT_WEB}/` (ports from `.env`, with sensible defaults).
   - `200`/`204` → **pass**; `000` (unreachable) or `404` (route not built yet) → **warn and continue**; any other code → **warn** (review, still non-fatal at this layer — the CI smoke-test stage can hard-gate later once endpoints are stable).

8. **Final log** — print completion with the `BUILD_ID`.

**Invocation:** triggered by CI/CD over SSH on push to `staging`, or run manually by the operator. The agent must also generate the companion `ecosystem.config.js` (defining `kc-api`, `kc-web`, and — when their build-steps arrive — `kc-worker`, `kc-scheduler`) and `.env.example` (placeholders only).

---

## 29. GitHub Repository & Branching

- **Private monorepo** `khundiconnect` (pnpm/turbo: `apps/api`, `apps/web`, `packages/shared`).
- **Separate PUBLIC progress repo** holds only `ARCHITECTURE.md` + `PROGRESS.md` (per-project folder). No secrets ever.
- Branches: `main` (live), `staging` (auto-deploys to staging), feature branches per build-step (`feat/01-foundation`, `feat/04-permissions`, …) matching §27.
- **The agent works only on `feat/*` and `staging`. It never touches `main` and never triggers the live environment.**
- **Secrets NEVER in repo.** `.gitignore` covers `.env`, keys, certs. Only `.env.example` is committed.
- **Branch protection on `main`:** requires CI pass + human approval before merge — the security checkpoint gate.

---

## 30. CI/CD Pipeline (GitHub Actions → aaPanel VPS)

Two workflows. The agent generates these; the operator adds the GitHub Secrets.

### 30.1 `ci.yml` (on every push / PR)
Runs: install → lint → typecheck → **unit tests** → **integration tests (incl. tenant-isolation tests, see §31)** → build. PR cannot merge unless all pass.

### 30.2 `deploy-staging.yml` (on push to `staging`)
After CI passes → SSH into VPS → run `deploy.sh` on the staging environment → run smoke tests → report status.

### 30.3 Live promotion
The live environment is promoted **only on the owner's explicit Telegram command** — never automatically, never by the agent. (Deliberately not described as an auto-trigger here.)

### 30.4 SSH Deploy Mechanism
- Uses an **SSH deploy action** with a dedicated **deploy SSH key** (key pair generated by operator; public key on VPS `~/.ssh/authorized_keys`, private key in GitHub Secrets).
- **Required GitHub Secrets:** `VPS_HOST`, `VPS_USER`, `VPS_SSH_KEY`, `VPS_PORT`, plus any build-time public config. (DB/API secrets stay on the VPS `.env`, never in GitHub.)

### 30.5 Operator (Saad) One-Time VPS Setup — Guided Checklist
The agent will produce a step-by-step runbook; high level:
1. Provision Oracle ARM VPS + install aaPanel.
2. Install Node v22, PostgreSQL, Redis (via aaPanel or apt).
3. Create staging + live Postgres databases.
4. Point Cloudflare DNS: base domain, wildcard, `staging.` → VPS IP.
5. Issue TLS certs (wildcard) via aaPanel/Let's Encrypt.
6. Generate deploy SSH key; add public key to VPS, private key to GitHub Secrets.
7. Clone repo, create `.env` from `.env.example` with real values.
8. Install PM2 globally; first run of `deploy.sh`.
> Everything else (code, migrations, tests, workflows, seed data) is produced and run by the agent.

---

## 31. Agentic Build Methodology & Mandatory Verification Gates

**This project is built by an autonomous coding agent (Claude Code on the VPS, driven via Telegram, wrapped by the GeekAxon Builder controller). The agent MUST NOT attempt to build all modules in one pass.** It builds **build-step-by-build-step in the §27 order**, and **must stop at the verification gates and checkpoints below before proceeding.** Full agent operating rules are in **AGENT.md.**

### 31.1 Per-Module Loop (the agent repeats this for every build-step)
1. Read `ARCHITECTURE.md` (this file) + `AGENT.md` + `PROGRESS.md` (current state).
2. Plan the build-step; list data model, permission keys, module-flag, API, UI, tests.
3. Implement: migrations + RLS, backend, frontend (animated, Urdu-ready), audit hooks.
4. **Seed dummy/test data** for this build-step (see §31.3).
5. **Run the verification gate tests** (see §31.2). If any fail → fix → re-run. Do NOT proceed on red.
6. Produce Artifacts: test results, screenshots, a short summary.
7. Update `PROGRESS.md` (mark step done, note decisions, next step).
8. Commit to a feature branch → push → **print `[CHECKPOINT]` and wait** where required (§31.4 + general rule).

### 31.2 Mandatory Verification Gates (run automatically every build-step)
- **TENANT ISOLATION (critical, every data module):** automated tests that **prove Jafrani-B cannot see, query, edit, or fetch Toberiya-A's data** — and vice versa. Create ≥2 tenants in tests; attempt cross-tenant reads/writes through the API and assert they are **denied** by RLS AND by the app-layer guard. **This test runs for every module that touches tenant data. A failure blocks the build.**
- **PERMISSION GATE:** assert that each permission key denies the action when absent and allows it when present; assert scope (OWN_TENANT vs ALL_TENANTS) behaves correctly; assert OMJ read-only role cannot edit/impersonate.
- **MODULE-ENABLEMENT GATE:** assert a disabled module is fully unreachable (route 404/forbidden, API forbidden, no menu).
- **NO-AUTO-ACTION RULES:** assert the system does NOT auto-dissolve a Khundi below 20 members, does NOT auto-deduct/transfer the Rs 100 OMJ fee, does NOT auto-suspend, and does NOT silently overwrite on OMJ fetch (conflicts go to review queue).
- **SECRET HANDLING:** assert CNIC and API keys are encrypted at rest and never appear in logs, API responses (unless authorized), or URLs.
- **AI PRIVACY:** assert census data sent to LLM is aggregated/anonymized and contains no CNIC/identifying detail.
- **VOTING-ELIGIBILITY DERIVATION:** assert `is_voting_eligible` is computed (active + male + 18+ + Okhai-by-birth), never settable manually; assert females are uncounted toward the voting threshold (no "honorary" concept — split by gender); assert the 20-member threshold counts only eligible members.

### 31.3 Dummy Data Seeding (agent must create, for testing)
The agent generates a **seed script** producing a realistic test community so every feature is testable and demos look real:
- **≥3 tenants:** `Jafrani-B`, `Toberiya-A`, `Kath-A` (so isolation is always testable).
- Per tenant: **30–60 members** mixing male 18+ (voting), females, children (incl. a male child near 18 to test the turning-18 flag), seniors, a few suspended/inactive, a non-member "Okhai-by-birth" person (for the grave record case), and (v3.1) a Married-In non-Okhai female to test the derived marker + divorce→inactive path.
- **Family relationships** wired up (fathers, mothers, spouses, children, siblings) so the **family tree renders** with both required (filled + a few empty) and optional (present + absent) blocks.
- **Lawazim** payments across 2–3 years with some defaulters; one **OMJ remittance** expense entry.
- **Accounting** entries in both default heads; a tenant in Simple mode and one in Detailed mode.
- A **census snapshot** ("Census 2026") partially filled (to show progress %) with income bands incl. below-threshold cases (so **AI welfare-prioritization** has data).
- Sample welfare case, death/grave record, event with attendance, documents, notifications log.
- **Seed users with each seed role** (Khundi Super Admin, President, Gen.Sec, Councilor, Accountant, Operator, View-Only) + OMJ roles (Admin, read-only Analyst) + a Vendor owner — each with a known test login (OMJ card number) so every altitude is testable.
- Seed data is **staging/dev only**, clearly tagged, with a **reset/clear command**.

### 31.4 Human Checkpoints (where Saad must review before proceeding)
The agent **prints `[CHECKPOINT]` and pauses** for APPROVE/REJECT after these fixed build-steps:
- After **build-step 2 (multi-tenancy + RLS)** — verify isolation tests are green.
- After **build-step 3 (auth)** — verify login security.
- After **build-step 4 (permission engine)** — verify the dual-gate works.
- After **build-step 7 (OMJ fetch)** — verify conflict queue + no silent overwrite.

**Plus the general rule** (AGENT.md): the agent also checkpoints on ANY data-model change, RLS/`tenant_id` touch, auth/permission change, or production-affecting action. For ordinary CRUD steps with none of those, the agent may proceed after green tests with a lighter review. **Final full review + debug pass is done by Saad at the end on staging before the live promotion.**

### 31.5 What the Agent Must NOT Do
- Must not put real secrets in the repo or in any committed file (and never in the public ARCHITECTURE.md/PROGRESS.md).
- Must not weaken/skip the tenant-isolation or permission tests to make a build pass.
- Must not test against live data.
- Must not mark a build-step done in `PROGRESS.md` while any verification gate is red.
- Must not invent values marked **[DECIDE AT BUILD]** without noting the assumption in `PROGRESS.md` for review.
- Must not touch `main`, trigger the live environment, or emit commands the controller's safety fence blocks.

### 31.6 GeekAxon Builder Execution Model
- The agent runs as **Claude Code on the VPS**, driven via **Telegram**, wrapped by the **GeekAxon Builder controller**.
- The agent **reads `ARCHITECTURE.md` + `AGENT.md` + `PROGRESS.md` itself** at the start of every session (the controller does not auto-inject them; it runs the agent inside the project folder).
- **Controller guardrails:** STOP kill switch, 3-consecutive-error auto-pause, usage-limit auto-pause, a dangerous-command fence (blocks `rm -rf /`, drop database, force-push, anything matching "production"/"prod"), and **auto-commit + push after every successful task**.
- On **resume after any pause**, the agent **re-reads `PROGRESS.md` to recover state** — it never assumes continuity.
- Full operating rules, the checkpoint token format, the Telegram reporting format, and branch rules are in **AGENT.md.**

---

## 32. Shared Form-Builder Engine (v2 subsystem — build-step 20)

A single, reusable form-definition engine that powers **Census** (build-step 24) and **Prize Distribution** (build-step 29), and is available to future modules. It is the architectural answer to "ship a default form, let each Khundi customize it, keep core fields comparable."

**Core model (tenant-scoped, RLS):**
- **form_definitions** — `id`, `tenant_id`, `purpose` (`census` | `prize` | …), `version`, `label`, `is_default` (the shipped default this Khundi started from), `steps` (JSON: ordered multi-step layout), `status` (draft/active/archived), `created_by`, `created_at`. A Khundi can clone the default and edit per round; each census round / prize ceremony references a specific definition.
- **form_fields** — `id`, `form_definition_id`, `tenant_id`, `key`, `label`, `type` (text/number/date/select/multiselect/boolean/file/…), `is_locked_core` (bool — cannot be removed or have its key changed), `required`, `order`, `step_index`, `option_set_id` (nullable), `validation` (JSON).
- **option_sets** — `id`, `tenant_id`, `field_key`, `style` (`light` | `detailed`), `options` (JSON ordered list, each {value,label}), `allow_other` (bool). A Khundi authors its own option sets for Profession, Education, income, etc., choosing a **light** style (e.g. Below Average / Average / Above Average) or a **detailed** style (actual ranges/lists), with an optional "Other" free-text.
- **form_submissions** — the consuming module owns the answer rows (e.g. `census_entries.data`, prize submissions). The engine validates answers against the definition; it does not duplicate storage.

**Locked-core rule:** every `purpose` declares a **locked-core field set** (code-seeded) that a Khundi **cannot remove** (only restyle its options / reorder / mark light-vs-detailed). Locked core guarantees population math + cross-Khundi comparability. **Custom fields** are unlimited and Khundi-owned.

**Cross-Khundi rule (ties to §7.5):** OMJ receives **all** answers including custom fields (per-Khundi view), but **cross-Khundi aggregation / Jamat-AI uses ONLY locked-core fields** (custom fields aren't comparable). Custom fields still appear in that Khundi's own reports/insights.

**UX:** a builder UI for controllers (add/reorder/restyle fields, author option sets, arrange steps) and a **multi-step, low-literacy public renderer** (large targets, minimal typing, select-driven, Urdu/RTL). Versioned + non-destructive: editing a form never alters already-submitted answers.

## 33. Simple ↔ Pro Mode Framework (v2 subsystem — build-step 21)

A platform-wide progressive-disclosure framework (the GarageAxon pattern), so any applicable module presents a **Simple** or **Pro** face over the **same data**.

**Core rules (locked):**
- **Lossless:** mode is presentation only — every module always records full detail regardless of mode; switching never migrates or loses data.
- **Per-user preference (KhundiConnect's chosen scope):** mode is a **saved per-user preference per module**, persisted, so a Councilor can keep Accounting on Simple while the President keeps it on Pro, set once. (A Khundi-level default may seed the initial value; the user's own toggle wins for them.)
- **Role-aware ceiling:** effective mode = `min(userPreference, roleCeiling)` where a role may cap a user at Simple. Some modules are Simple-only or Pro-only by nature (declared in a registry).
- **Scope of modules:** Accounting, Members, Census, Welfare, Events, Prize, Documents, Reports get a toggle; Lawazim stays single-mode. (Registry-declared.)

**Data model (tenant-scoped, RLS):**
- **module_registry** (code-seeded, global): `module_key`, `label`, `supports_simple`, `supports_pro`, `default_mode`.
- **user_mode_preference** (tenant-scoped): `id`, `tenant_id`, `user_id`, `module_key`, `mode` (simple/pro), `updated_at`. The per-user saved preference.

**Contract:** a `useMode(moduleKey)` hook (web) returning the effective mode for the current user (applies the role-ceiling min + module nature); an API-side `getEffectiveMode(userId, moduleKey)`; each module exports a `Simple<Module>` and `Pro<Module>` view; a wrapper picks based on `useMode`. Documented in `packages/ui/MODE_CONTRACT.md` so every later module conforms. **Mode shapes presentation; permissions gate access — mode must never withhold data a user is permitted to see.**

## 34. Global UI/UX Layer (v2 subsystem — build-step 22)

The cross-cutting presentation standards every module must meet (the answer to "many things are in small letters / not professional / lists are crude").

- **Human-readable labels everywhere:** no raw enum/machine values ever reach the user. A central label-formatter maps stored values → proper display (e.g. `medical`→"Medical", `open`→"Open", `non_member_okhai`→derived "Non-Member (Okhai by birth)"), with **Urdu translations** of those labels. Stored values are unchanged; only display is formatted. Applies to every status chip, type dropdown, and enum surface.
- **Table ⇄ Card toggle on every listing:** each list (Members first, then everywhere applicable) offers a professional **table** view and a **card** view, user-toggleable.
- **Per-Khundi column chooser:** a controller setting in **Khundi Settings** selects which columns appear in each list; applied across all listings. (Column visibility is per-Khundi/controller-set; the Simple/Pro mode is per-user — this asymmetry is intentional.)
- **Professional list components:** collapsible filter panels, "show N entries", export, pagination/virtualization, skeleton loaders, consistent design tokens — matching the quality bar of the OMJ Lawazim reference software, not raw output.
- **Design bar is part of every module's definition of done** (tokens, light/dark, RTL, animation, accessibility). A module is not "done" if its UI is unstyled or exposes raw values.

---

*End of Architecture Document.*
