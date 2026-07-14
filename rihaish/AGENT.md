# AGENT.md — Rihaish

Build-loop mechanics live in `CLAUDE.md` (controller-managed). This file is **what "correct" means for this project**.

## 1. Golden rules (violating any = stop and fix, not "note in PR")
1. **Never hard-delete.** Soft delete + `AuditLog` entry. Applies to money, users, passes, roles, entitlements — and to platform staff, whose audit rows must always still resolve to a name.
2. **Never write a query without `societyId` scope.** Use the enforced data layer (Prisma extension). A raw unscoped query is a build failure.
3. **Never merge platform billing with society billing.** Different tables, different modules, different UI.
4. **Never mutate financial history.** Correct by reversal (new ledger entry), never by edit or delete. A refund is a reversal.
5. **Money = BigInt minor units** + society `currency`/`minorUnitDigits`. No floats, ever. PKR has 0 decimals. A JS `number` anywhere near money is a bug.
6. **Timestamps stored UTC**, rendered in society timezone.
7. **No secrets in the repo.** `.env.example` placeholders only. Secret values stay **alphanumeric and single-line** (`deploy.sh` sources `.env` as shell — a PEM with newlines breaks the deploy). Society SMS/WhatsApp credentials encrypted at rest, never logged.
8. **No auto-production deploy.** Staging on push; production only by owner Telegram command.
9. **Every module registers a feature code + dependency edges** in the feature registry. A module that can't be toggled by L0 is unfinished.
10. **Utility-bill amounts never enter the maintenance ledger.** Notices are reminders, not receivables.
11. **CNIC and other sensitive PII**: encrypted at rest, masked in UI (last 4), never in logs, never exported without an audit entry.
12. **Cross-tenant reads are impossible.** A tenant host can never reach a platform route, and vice versa — including via the screenshot token.
13. **Never assert exact equality against a growing global registry** (cron, features, nav, templates, permissions). Assert that required entries are present, plus invariants (uniqueness, validity, handler exists). Registries are append-only by design; a test that forbids appending is a broken test.
14. **Society-scoped work runs inside `withSociety`.** Any worker handler, job or script that touches society data must run inside `withSociety(societyId, …)`. `db.unscoped()` is for platform-level and cross-society work **only**. If you find yourself reaching for `unscoped()` inside a society handler, you have the wrong `societyId` — not the wrong scope. *(This exact mistake was made three times: worker, lifts, taxonomy.)*
15. **A user never sees a machine value.** Enums, slugs, feature codes, job kinds, audit actions and roles all render through the label registry (`lib/labels/`), in **en and ur**. Every ID shown to a human is replaced by that entity's **name**, anchor-linked. A missing label **fails the build** — never fall back to the raw slug.
16. **No hex colour outside `app/globals.css`.** The brand is `#023029` / `#d8a03e`, defined once. A literal hex in a component fails `lib/design/tokens.test.ts`.
17. **Never issue a session before 2FA is verified.** The platform login is: password → challenge token (not a session) → TOTP code → session. No platform route opens without `totpVerifiedAt`.
18. **Never let a raw Zod message reach a user**, and **never validate on mount**. Forms validate on blur (touched) and on submit. Messages name the field and say what to do, in the user's language.
19. **A payment webhook is the source of truth, never the browser redirect** — and it must verify its signature (constant-time) and be **idempotent**. A replayed webhook credits exactly once.
20. **A metric with no data renders "—", never a number.** A dashboard that reports 100 % collection on an empty database will be believed, and it is a lie.
21. **Residents can never self-signup.** Societies self-provision (spec 64); residents are invited or imported by their society (spec 04). This is asserted by a test, not by convention.

## 2. Per-module workflow
**The build loop, branch naming (`feature/<NN-slug>` / `fix/<slug>`), the mandatory `WORK TYPE:` line, the no-self-merge rule and the PROGRESS-file update rules all live in `CLAUDE.md`. Follow it — do not re-derive them from here.** This section only adds what is specific to Rihaish:

1. The spec at `/specs/NN-slug.md` is **authoritative**. Missing spec → `[HUMAN_REQUIRED]`, never guess.
2. If the spec has a companion `NN-NN-CODEREF.md`, **read it first**. It names the exact files, tells you what to reuse, and lists what not to do. Treat its "Do NOT" section as binding.
3. Implement the module **and register its feature code + dependency edges** (§1.9). A module L0 cannot toggle is unfinished.
4. Write the migration. Write the tests demanded by §3.
5. Run every verification gate in §3. All must pass before the checkpoint.
6. End the step with `[CHECKPOINT]` — always, for every module. It is a marker, not a gate (§4).
7. **Never stop for approval.** There is no approval gate in this project. Record the decision in `PROGRESS-HISTORY.md` and keep building.

## 3. Verification gates (every module)

**The agent does NOT run these. The controller runs them after the step, in Node, for free.** Running a growing test suite inside the agent's session costs tokens on every later turn and re-sends the whole reporter output. Your job is to **write the code and the tests**; the controller executes them and, if one fails, hands you a targeted repair task carrying only the failure output. A gate failure blocks the merge to staging.

Gates the controller runs, in order — `pnpm lint` → `pnpm typecheck` → `pnpm test:unit` → `pnpm build` → **`pnpm test:e2e`**:

- **Mandatory Vitest coverage** (write these; they will be run for you) on: tenant-scoping layer, entitlement resolver + DAG, charge engine, ledger, payment reversal, invoice numbering, TOTP verification, webhook idempotency, pricing bands.
- **Mandatory Playwright coverage.** This is no longer optional and no longer "smoke". **`pnpm test:e2e` runs in CI against the built app and blocks the deploy** (spec 62).

  > **Read this once and remember it: 1,315 unit tests passed while every page of the website returned 404.** Not one of them loaded a page over HTTP. A test suite that never renders a page cannot tell you the product works — it can only tell you the code compiles.

  So: **if your module has a screen, your module has a Playwright test that calls `page.goto` and asserts what a human would see.** Not an imported function. A page.
- **UI design self-check** — this one IS yours, done by inspection, before you finish (UI modules — all must be true, else the module is not done):
  - [ ] shadcn/ui + design tokens only — no raw HTML, no hand-rolled table, no hand-rolled input, **no hex literal**
  - [ ] Light **and** dark verified
  - [ ] **RTL/Urdu** verified — logical properties (`ms-/me-/ps-/pe-`), never `ml-/mr-`
  - [ ] Responsive: guard's phone → tablet → desktop
  - [ ] Skeleton loader, empty state, error state all present
  - [ ] Every list uses the **Data Table kit** (sort, column visibility, search, filters, pagination, **working** list/card toggle, export)
  - [ ] Every input uses the **Form kit** (searchable select for >10 options, date/time picker, intl phone, masks) — **no validation on mount, no raw Zod message**
  - [ ] **No slug, no enum value, no raw ID is visible anywhere.** Names, linked to their entity.
  - [ ] Client-side navigation — **no full page reload**
  - [ ] Simple **and** Pro variants implemented where applicable
  - [ ] Charts animated, accessible, exportable; **a metric with no data shows "—"**
  - [ ] Nav + actions gated by role × entitlement, verified against the server guard
  - [ ] Toasts for feedback; no browser `alert()`
  - [ ] The **logo** is present wherever the design calls for it (`components/brand/logo.tsx` — never a raw file path)
  - [ ] Module still works correctly with its optional features **disabled** (graceful degradation, never a 500)
  - [ ] Screenshot attached to the checkpoint

## 4. Checkpoints — markers, not gates
- **`[CHECKPOINT]`** — the only checkpoint marker. Ends every module. It is a **progress marker, not an approval gate**: the controller auto-approves, merges to staging and continues. Never wait, never ask, never pause.
- **`[FIXED_CHECKPOINT]` is abolished.** Do not emit it. If an older spec still contains it, treat it as a plain `[CHECKPOINT]`.
- **`[HUMAN_REQUIRED]`** — the **only** stop condition, reserved for things the agent physically cannot do:
  - a missing or empty spec,
  - DNS / wildcard SSL / custom-domain SSL,
  - Oracle iptables + Security List port opening, aaPanel site creation,
  - a credential the agent does not have (SafePay keys, Turnstile secret, SMTP),
  - a deploy failure that is **not** code-fixable (after the 2-attempt rule in §5).
- **Never** emit `[HUMAN_REQUIRED]` for an approval, a design review, a security sign-off, or a money-model decision. Those are yours to make. Make them, log the reasoning in `PROGRESS-HISTORY.md`, and continue.

## 5. Deploy-error triage
- **Code-fixable** (typecheck, build, lint, migration conflict, missing Prisma client, bad import, failing test): fix and retry. **Maximum 2 attempts**, then `[HUMAN_REQUIRED]` with the exact error.
- **Infra** (DNS, SSL, port/firewall, aaPanel, disk, DB unreachable, Node missing, permissions): do **not** retry-loop. Emit `[HUMAN_REQUIRED]` immediately with the failing command and output.
- **Bisect before you theorise.** When a whole-site failure appears (every page 404s, every asset 403s), replace the suspect layer with a trivial pass-through and see if the failure survives. Days were lost blaming nginx, `.env` and cwd for a middleware bug that a two-line bisect exposed in minutes.
- Never `rm -rf`, never `DROP DATABASE`, never `--force` push, never touch anything named `production`/`prod` outside the sanctioned deploy path.

## 6. Build-tooling & deploy hard rules
- `pnpm install --frozen-lockfile --prod=false` — dev deps are required for typecheck/build. **`NODE_ENV` must never appear in `.env`.**
- `prisma generate` runs **after install, before typecheck/build** — in CI *and* in `deploy.sh`. Same order in both.
- Prisma `binaryTargets = ["native","linux-arm64-openssl-3.0.x"]` — the VPS is **ARM64**. `sharp` must resolve its arm64 build.
- Fresh `BUILD_ID` every deploy. `pm2 restart` (never `reload`). The health gate must check `/api/health` **and** that `GET /` renders.
- `deploy.sh` **never runs a test suite.** Tests run in CI, against a CI database — not against live staging data.
- Two processes: `rihaish-web` (**fork**) + `rihaish-worker` (**fork**, via `node_modules/tsx/dist/cli.mjs`). `ecosystem.config.cjs` derives `cwd` from `__dirname`.
- **next-intl's middleware is not used.** Locale routing is hand-rolled in `middleware.ts`. Do not reintroduce `createMiddleware` — under Next 15 its `x-middleware-rewrite` header leaks to the client and every page 404s. Never touch that header.

## 7. Screenshot token
`?screenshot_token=` grants authenticated **read-only** preview. Enabled **only** when `APP_ENV === "staging"`. **Fail-closed** everywhere else. Never in production. Validated **after** the society is resolved and **scoped to that society** — it can never read across tenants. Documented in `.env.example`.

## 8. Design quality is not optional
Polished, modern, "VIP" UI is part of the definition of done for every UI module — not a follow-up task. Plain HTML, an unstyled table, a native `<select>` with 300 options, a missing logo, a visible slug, a raw Zod error, or a missing dark/RTL variant means the module **fails** its checkpoint, regardless of whether the logic works.

## 9. The failure mode to watch in yourself
Once, a working fix was deleted and replaced with a confident, well-written justification that was simply **false** (it claimed `req.nextUrl` carries the public host; it does not). Nothing errored. The prose was persuasive. The site broke.

**A fluent explanation is not evidence.** If you are about to remove code that someone put there deliberately — especially code with a comment saying "do not remove this" — the burden is on you to *demonstrate* it is unnecessary with a test, not to *argue* it. Write the test that would fail if you were wrong. If you cannot write that test, leave the code alone.
