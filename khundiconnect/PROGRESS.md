> ⚠️ **PUBLIC FILE** — pushed to a separate public progress repo. **NO secrets, hostnames, real subdomains, API endpoints, credentials, or keys — ever.** Placeholders only.

# PROGRESS.md — KhundiConnect Build Tracker (SHORT)

> **This is the SHORT working tracker the build agent reads at the start of every step.** It holds only: current state, the next-step pointer, the checkpoint rule, and the last few steps as one-liners. The COMPLETE step-by-step detail is in **PROGRESS-HISTORY.md** (append-only archive).
>
> **At the END of the step you just finished (before `[CHECKPOINT]`, as part of finishing the spec — never deferred to the next step):** (1) mark the step you just finished as DONE/APPROVED in **Current Status** (do not leave it PENDING), (2) prepend a ONE-LINE entry to **Recent steps**, (3) drop the now-4th recent one-liner, and (4) append the FULL detailed entry for the step you just finished to **PROGRESS-HISTORY.md**. This recording is tied to the SPEC being done, NOT to any deploy/merge — record it regardless of deploy. Never let this file grow long.
>
> **Numbering:** build-step numbers are canonical (ARCHITECTURE.md §27). "Module N" = build-step N.
>
> **Checkpoint rule (continuous mode):** end every step with `[CHECKPOINT]` on its own final line — the controller records PROGRESS, then SILENTLY merges the branch to staging + pushes (deploy runs in the background; check it with `DEPLOY STATUS`) and continues to the next step. Do NOT use `[FIXED_CHECKPOINT]` — even data-model / RLS / auth changes just end with `[CHECKPOINT]`. Stop ONLY by ending with `[HUMAN_REQUIRED]` when the step's spec file is missing/empty or an infra problem is not code-fixable.

## Current Status
- **Project:** KhundiConnect (multi-tenant community SaaS)
- **Phase:** **v3.1 program underway (build-steps 54+, see `specs/README-v3.1.md`).** v3 (38–53) + v2 (19–37) complete. Now in **Wave 1 — Core UI primitives & cross-cutting (54–60)** — 54 + 55 + 56 + 57 done, 58 next. The cross-org welfare ledger stays deferred to v4; the data model is built forward-compatible.
- **Last completed build-step:** **57 — Toast notifications + inline form validation** ✅ **DONE** (built 2026-06-27) · FEATURE · branch `feat/57-toast-validation-v31` · NEW `apps/web/app/ui/toast.tsx` (`<ToastProvider>`+`useToast`, mounted inside `I18nProvider` in `layout.tsx`; fixed logical `top-4 end-4` stack, badge-token intent colours/icons, in-memory only, auto-dismiss errors-longer + close button) + `field.tsx` inline validation (`invalid?` prop on Input/Select/Textarea + `<FormField label error required>` + `useFieldErrors` hook with `focusFirst`). Per-action feedback moved from `<Notice>` banners to `toast.*` across members/welfare/events/prize/census; inline required-field validation on the members/welfare/events/prize create-edit modals (submit gates removed so the error is reachable + focused). New i18n keys `common.notifications/saved/saveFailed/fieldRequired` (en+ur). Tests 1199/1199, typecheck/lint/build green. Isolation ➖ (presentation only — no data-model/endpoint change).
- **Current build-step:** **58 — CNIC masking + international phone input** (`specs/58-*.md`, CODEREF `specs/54-60-CODEREF.md`). Reuse existing `normalizeCnic`/`isValidCnic`/`CNIC_FULLY_MASKED` + the audited reveal path; add masking in `field.tsx`/new `masked-input.tsx`, a new `phone-input.tsx` (default PK +92 → E.164), and a shared pure E.164 validator used by both web input + API DTOs. End with `[CHECKPOINT]` (continuous mode — NOT `[FIXED_CHECKPOINT]`). If the spec file is missing/empty, end with `[HUMAN_REQUIRED]` and stop.
- **Branch state:** build-steps 54 + 55 + 56 merged to staging; **57 built on `feat/57-toast-validation-v31`** (off the staging head after 56) — the controller merges it to staging at the checkpoint. Start build-step 58 on a fresh `feat/58-*` branch off the resulting staging head.

### Recent steps (one-liners — FULL detail in PROGRESS-HISTORY.md)
- **57 — Toast + inline validation** ✅ DONE · FEATURE · branch `feat/57-toast-validation-v31` · tests 1199 · NEW `ui/toast.tsx` (`<ToastProvider>`+`useToast`, badge-token intents, RTL `end-*` stack) + `field.tsx` `invalid?`/`<FormField>`/`useFieldErrors`; per-action feedback → `toast.*` and inline required-field validation across members/welfare/events/prize/census. Presentation only.
- **56 — dynamic filter engine** ✅ DONE · FEATURE · branch `feat/56-filter-engine-v31` · tests 1199 · shared filter engine (`FilterSpec`/`ListFilterConfig`/`applyFilters`/`filtersToQuery`) + `<FilterBar>` in `<DataList>`, auto client/server by row-count + debounced; demoed on members. Same state shape reused by audience builder (78).
- **55 — per-user column preferences** ✅ DONE · FIX · branch `feat/55-per-user-columns-v31` · adds `user_id` to the list-column-preference model (partial-unique migration) → per-USER columns over the per-Khundi default; `PUT /ui/lists/:listKey/me`. _(Built prior session; full detail now in PROGRESS-HISTORY.md as a recovery entry.)_

> _Note: build-step 52 (Notifications channels — PWA/SMS/WhatsApp providers) was the step between 51 and 53 in the build order; see PROGRESS-HISTORY.md for the full sequence and every step's detail._

---

> Older steps (everything before the three above) are archived in **PROGRESS-HISTORY.md**. Read that file only when you need to recover detail about an older step — never read it cover-to-cover each step.
