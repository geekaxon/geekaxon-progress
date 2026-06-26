> ⚠️ **PUBLIC FILE** — pushed to a separate public progress repo. **NO secrets, hostnames, real subdomains, API endpoints, credentials, or keys — ever.** Placeholders only.

# PROGRESS.md — KhundiConnect Build Tracker (SHORT)

> **This is the SHORT working tracker the build agent reads at the start of every step.** It holds only: current state, the next-step pointer, the checkpoint rule, and the last few steps as one-liners. The COMPLETE step-by-step detail is in **PROGRESS-HISTORY.md** (append-only archive).
>
> **After finishing a step:** (1) update **Current Status** below, (2) prepend a ONE-LINE entry to **Recent steps**, (3) drop the now-4th recent one-liner, and (4) append the FULL detailed entry for the step to **PROGRESS-HISTORY.md**. Never let this file grow long.
>
> **Numbering:** build-step numbers are canonical (ARCHITECTURE.md §27). "Module N" = build-step N.
>
> **Checkpoint rule (continuous mode):** end every step with `[CHECKPOINT]` on its own final line — the controller auto-approves, merges the branch to staging, deploys staging, and continues. Do NOT use `[FIXED_CHECKPOINT]`. Stop ONLY by ending with `[HUMAN_REQUIRED]` when the step's spec file is missing/empty or an infra problem is not code-fixable.

## Current Status
- **Project:** KhundiConnect (multi-tenant community SaaS)
- **Phase:** **v3.1 program underway (build-steps 54+, see `specs/README-v3.1.md`).** v3 (38–53) + v2 (19–37) complete. Now in **Wave 1 — Core UI primitives & cross-cutting (54–60)**. The cross-org welfare ledger stays deferred to v4; the data model is built forward-compatible.
- **Last completed build-step:** **54 — DataList actions redesign** ✅ FIX · branch `feat/54-datalist-actions-v31` · first-class per-row actions (first table column / card footer, responsive label↔colored-icon) + fail-loud on an unregistered listKey + shared `RowAction` contract; retrofit moved primary actions out of `renderExpanded` across members/welfare/census/prize/documents/accounting/events. Shared-UI + shared-type change only (no schema/RLS/auth). tests 1165/1165 green.
- **Current build-step:** **55 — per-user column preferences** (`specs/55-per-user-columns-v31.md`, CODEREF `specs/54-60-CODEREF.md`). NOTE: 55 ADDS `user_id` to the preference model (data-model + a hand-written migration) → end it with `[FIXED_CHECKPOINT]`. If the next spec file is missing/empty, end with `[HUMAN_REQUIRED]` and stop.
- **Branch state:** work through build-step 53 merged to staging; build-step 54 on `feat/54-datalist-actions-v31` (off staging head). Start each new build-step on a fresh `feature/<NN-slug>` or `fix/<slug>` branch off the current staging head.

### Recent steps (one-liners — FULL detail in PROGRESS-HISTORY.md)
- **54 — DataList actions redesign** ✅ done (awaiting auto-approve) · FIX · branch `feat/54-datalist-actions-v31` · tests 1165 · first-class per-row actions (first column / card footer, responsive label↔colored-icon) + fail-loud on bad listKey + shared `RowAction`; retrofit across members/welfare/census/prize/documents/accounting/events. Shared-UI only.
- **53 — Census report/presentation generator** ✅ approved · FEATURE · branch `feat/53-census-report` · tests 1165 · v3 capstone (facts-only census + derived vulnerability + welfare-outreach view).
- **51 — Form Builder per-instance binding + default forms** ✅ approved · FIX · branch `feat/51-form-binding` · tests 1135 · Census/Prize bind a named form; one default per purpose.

> _Note: build-step 52 (Notifications channels — PWA/SMS/WhatsApp providers) was the step between 51 and 53 in the build order; see PROGRESS-HISTORY.md for the full sequence and every step's detail._

---

> Older steps (everything before the three above) are archived in **PROGRESS-HISTORY.md**. Read that file only when you need to recover detail about an older step — never read it cover-to-cover each step.
