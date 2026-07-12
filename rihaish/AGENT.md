# AGENT.md — Rihaish

Build-loop mechanics live in `CLAUDE.md` (controller-managed). This file is **what "correct" means for this project**.

## 1. Golden rules (violating any = stop and fix, not "note in PR")
1. **Never hard-delete.** Soft delete + `AuditLog` entry. Applies to money, users, passes, roles, entitlements.
2. **Never write a query without `societyId` scope.** Use the enforced data layer (Prisma extension). A raw unscoped query is a build failure.
3. **Never merge platform billing with society billing.** Different tables, different modules, different UI.
4. **Never mutate financial history.** Correct by reversal (new ledger entry), never by edit or delete.
5. **Money = BigInt minor units** + society `currency`/`minorUnitDigits`. No floats, ever. PKR has 0 decimals.
6. **Timestamps stored UTC**, rendered in society timezone.
7. **No secrets in the repo.** `.env.example` placeholders only. Society SMS/WhatsApp credentials encrypted at rest, never logged.
8. **No auto-production deploy.** Staging on push; production only by owner Telegram command.
9. **Every module registers a feature code + dependency edges** in the feature registry. A module that can't be toggled by L0 is unfinished.
10. **Utility-bill amounts never enter the maintenance ledger.** Notices are reminders, not receivables.
11. **CNIC and other sensitive PII**: encrypted at rest, masked in UI (last 4), never in logs, never exported without an audit entry.
12. **Cross-tenant reads are impossible.** A tenant host can never reach a platform route, and vice versa — including via the screenshot token.
13. **Never assert exact equality against a growing global registry** (cron, features, nav, templates, permissions). Assert that required entries are present, plus invariants (uniqueness, validity, handler exists). Registries are append-only by design; a test that forbids appending is a broken test.

## 2. Per-module workflow
**The build loop, branch naming (`feature/<NN-slug>` / `fix/<slug>`), the mandatory `WORK TYPE:` line, the no-self-merge rule and the PROGRESS-file update rules all live in `CLAUDE.md`. Follow it — do not re-derive them from here.** This section only adds what is specific to Rihaish:

1. The spec at `/specs/NN-slug.md` is **authoritative**. Missing spec → `[HUMAN_REQUIRED]`, never guess.
2. Implement the module **and register its feature code + dependency edges** in the feature registry (§1.9). A module that L0 cannot toggle is unfinished.
3. Write the migration. Write the tests demanded by §3.
4. Run every verification gate in §3. All must pass before the checkpoint.
5. End the step with `[CHECKPOINT]` — always, for every module. It is a marker, not a gate (§4).
6. **Never stop for approval.** There is no approval gate in this project. Record the decision in `PROGRESS-HISTORY.md` and keep building.

## 3. Verification gates (every module)

**The agent does NOT run these. The controller runs them after the step, in Node, for free.** Running a growing test suite inside the agent's session costs tokens on every later turn and re-sends the whole reporter output. Your job is to **write the code and the tests**; the controller executes them and, if one fails, hands you a targeted repair task carrying only the failure output. A gate failure blocks the merge to staging.

Gates the controller runs, in order — `pnpm lint` → `pnpm typecheck` → `pnpm test:unit` → `pnpm build`:

- **Mandatory Vitest coverage** (write these; they will be run for you) on: tenant-scoping layer, entitlement resolver + DAG, charge engine, ledger, payment reversal, invoice numbering.
- **Playwright smoke** (`pnpm test:e2e`) — write these only where the spec's acceptance criteria demand them: auth, invoice generation, gate-pass verification.
- **UI design self-check** — this one IS yours, done by inspection, before you finish (UI modules — all must be true, else the module is not done):
  - [ ] shadcn/ui + design tokens only — no raw HTML, no hand-rolled table, no hand-rolled input
  - [ ] Light **and** dark verified
  - [ ] **RTL/Urdu** verified — logical properties (`ms-/me-/ps-/pe-`), never `ml-/mr-`
  - [ ] Responsive: guard's phone → tablet → desktop
  - [ ] Skeleton loader, empty state, error state all present
  - [ ] Every list uses the **Data Table kit** (sort, column visibility, search, filters, pagination, list/card toggle, export)
  - [ ] Every input uses the **Form kit** (searchable select for >10 options, proper date/time picker, intl phone, masks)
  - [ ] Client-side navigation — **no full page reload**
  - [ ] Simple **and** Pro variants implemented where applicable
  - [ ] Charts animated, accessible, exportable
  - [ ] Nav + actions gated by role × entitlement, verified against the server guard
  - [ ] Toasts for feedback; no browser `alert()`
  - [ ] Module still works correctly with its optional features **disabled** (graceful degradation, never a 500)
  - [ ] Screenshot attached to the checkpoint

## 4. Checkpoints — markers, not gates
- **`[CHECKPOINT]`** — the only checkpoint marker. Ends every module. It is a **progress marker, not an approval gate**: the controller auto-approves, merges to staging and continues. Never wait, never ask, never pause.
- **`[FIXED_CHECKPOINT]` is abolished.** Do not emit it. If an older spec still contains it, treat it as a plain `[CHECKPOINT]`.
- **`[HUMAN_REQUIRED]`** — the **only** stop condition, and it is reserved for things the agent physically cannot do:
  - a missing or empty spec,
  - DNS / wildcard SSL / custom-domain SSL,
  - Oracle iptables + Security List port opening, aaPanel site creation,
  - a credential the agent does not have,
  - a deploy failure that is **not** code-fixable (after the 2-attempt rule in §5).
- **Never** emit `[HUMAN_REQUIRED]` for an approval, a design review, a security sign-off, or a money-model decision. Those are yours to make. Make them, log the reasoning in `PROGRESS-HISTORY.md`, and continue.
- The steps that were formerly HARD (tenancy, auth, entitlements, the UI kits, the data model, the money modules, gate pass) are still the **highest-risk** steps: they demand the strictest verification gates in §3 and the fullest `PROGRESS-HISTORY.md` entry. They do **not** demand a human.

## 5. Deploy-error triage
- **Code-fixable** (typecheck, build, lint, migration conflict, missing Prisma client, bad import, failing test): fix and retry. **Maximum 2 attempts**, then `[HUMAN_REQUIRED]` with the exact error.
- **Infra** (DNS, SSL, port/firewall, aaPanel, disk, DB unreachable, Node missing, permissions): do **not** retry-loop. Emit `[HUMAN_REQUIRED]` immediately with the failing command and output.
- Never `rm -rf`, never `DROP DATABASE`, never `--force` push, never touch anything named `production`/`prod` outside the sanctioned deploy path.

## 6. Build-tooling & deploy hard rules
- `pnpm install --frozen-lockfile` **including dev deps** — typecheck/build need them.
- `prisma generate` runs **after install, before typecheck/build** — in CI *and* in `deploy.sh`. Same order in both.
- Prisma `binaryTargets = ["native","linux-arm64-openssl-3.0.x"]` — the VPS is **ARM64**. `sharp` must resolve its arm64 build.
- Fresh `BUILD_ID` every deploy. `pm2 restart` (never `reload`). Health check must gate success.
- `.env` loaded at repo root via `set -a; . ./.env; set +a`. Keep secret values alphanumeric.
- Two processes: `rihaish-web` (cluster) + `rihaish-worker` (fork). Nothing heavy runs in a request.

## 7. Screenshot token
`?screenshot_token=` grants authenticated **read-only** preview. Enabled **only** when `APP_ENV === "staging"`. **Fail-closed** everywhere else. Never in production. Validated **after** the society is resolved and **scoped to that society** — it can never read across tenants. Documented in `.env.example`.

## 8. Design quality is not optional
Polished, modern, "VIP" UI is part of the definition of done for every UI module — not a follow-up task. Plain HTML, an unstyled table, a native `<select>` with 300 options, or a missing dark/RTL variant means the module **fails** its checkpoint, regardless of whether the logic works.
