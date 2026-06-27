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
- **Phase:** **v3.1 program underway (build-steps 54+, see `specs/README-v3.1.md`).** v3 (38–53) + v2 (19–37) complete. Now in **Wave 1 — Core UI primitives & cross-cutting (54–60)** — 54 + 55 + 56 done, 57 next. The cross-org welfare ledger stays deferred to v4; the data model is built forward-compatible.
- **Last completed build-step:** **56 — dynamic filter engine** ✅ **DONE** (built 2026-06-27) · FEATURE · branch `feat/56-filter-engine-v31` · reusable filter engine in `@kc/shared` (`FilterSpec`/`ListFilterConfig`/`applyFilters`/`filtersToQuery`+`queryToFilters`, PURE+tested) + web `<FilterBar>` integrated into `<DataList>` with auto client-vs-server by row-count + 300ms-debounced text; demoed on the members roster (gender/status/votingOnly via FilterBar, age-band as a toolbar extra). Same `FilterState` shape the audience builder reuses at build-step 78. Tests 1199/1199. Isolation ➖ (presentation/read; server path only narrows within the existing tenant-scoped endpoint).
- **Current build-step:** **57 — Toast notifications + inline form validation** (`specs/57-*.md`, CODEREF `specs/54-60-CODEREF.md`). NEW `apps/web/app/ui/toast.tsx` (`<ToastProvider>` + `useToast`, mounted inside `I18nProvider` in `layout.tsx`) + inline validation on `field.tsx` (`invalid?` prop / `<FormField>`); roll out across members/welfare/events/prize/census. End with `[CHECKPOINT]` (continuous mode — NOT `[FIXED_CHECKPOINT]`). If the spec file is missing/empty, end with `[HUMAN_REQUIRED]` and stop.
- **Branch state:** build-steps 54 + 55 merged to staging; **56 built on `feat/56-filter-engine-v31`** (off the staging head after 54+55) — the controller merges it to staging at the checkpoint. Start build-step 57 on a fresh `feat/57-*` branch off the resulting staging head.

### Recent steps (one-liners — FULL detail in PROGRESS-HISTORY.md)
- **56 — dynamic filter engine** ✅ DONE · FEATURE · branch `feat/56-filter-engine-v31` · tests 1199 · shared filter engine (`FilterSpec`/`ListFilterConfig`/`applyFilters`/`filtersToQuery`) + `<FilterBar>` in `<DataList>`, auto client/server by row-count + debounced; demoed on members. Same state shape reused by audience builder (78).
- **55 — per-user column preferences** ✅ DONE · FIX · branch `feat/55-per-user-columns-v31` · adds `user_id` to the list-column-preference model (partial-unique migration) → per-USER columns over the per-Khundi default; `PUT /ui/lists/:listKey/me`. _(Built prior session; full detail now in PROGRESS-HISTORY.md as a recovery entry.)_
- **54 — DataList actions redesign** ✅ APPROVED · FIX · branch `feat/54-datalist-actions-v31` · tests 1165 · first-class per-row actions (first column / card footer, responsive label↔colored-icon) + fail-loud on bad listKey + shared `RowAction`; retrofit across members/welfare/census/prize/documents/accounting/events. Shared-UI only.

> _Note: build-step 52 (Notifications channels — PWA/SMS/WhatsApp providers) was the step between 51 and 53 in the build order; see PROGRESS-HISTORY.md for the full sequence and every step's detail._

---

> Older steps (everything before the three above) are archived in **PROGRESS-HISTORY.md**. Read that file only when you need to recover detail about an older step — never read it cover-to-cover each step.
