# AGENT.md — Instructions for the Autonomous Build Agent

You are Claude Code building the **SpotEdge** Binance Spot trading bot on the operator's VPS, one module per session, driven via Telegram by the GeekAxon Builder controller.

## Golden Rules

1. **Read first, every session:** ARCHITECTURE.md → this file → PROGRESS.md → then ONLY the current module's spec `/specs/NN-slug.md`. The spec is AUTHORITATIVE for that module. If the spec file is missing or unreadable → output nothing else and end with `[HUMAN_REQUIRED]`.
2. **One module per session, in build order.** Never skip ahead, never build parts of future modules, never "improve" completed modules unless the current spec requires an integration touch (log it in PROGRESS-HISTORY.md).
3. **No secrets, ever.** Only `.env.example` with placeholders enters the repo. Never print, log, or commit real keys/tokens/passwords. This repo is private, but private ≠ safe for secrets.
4. **NEVER enable LIVE trading.** LIVE mode, real API keys, and real orders are owner-gated (module 15 defines the gate; the owner flips it via Telegram). You build the mechanism; you never activate it. Treat anything matching LIVE-mode activation, `withdraw`, real-order placement outside the paper engine (before module 15 approval), `rm -rf /`, `drop database`, force push, or "production"/"prod" as fenced — do not execute.
5. **Work on `staging` branch.** Auto-commit + push after each successful task (the controller handles this; keep commits scoped to the module). PAPER mode + dummy/testnet data only.
6. **Update PROGRESS.md (one-liner per module status) and PROGRESS-HISTORY.md (detailed entry: what was built, decisions, deviations, verification results) after every module — before printing the checkpoint marker.**
7. **Money-math discipline:** all monetary values as decimals via a single money util (no float arithmetic on prices/quantities without the exchange-precision rounding helpers from module 03). Every quantity sent to the exchange must pass LOT_SIZE/PRICE_FILTER/minNotional rounding.
8. **Timezone discipline:** logic and schedules in UTC; only display formatting in PKT (GMT+5).

## Checkpoint Markers (print as the LAST line of the response)

- `[FIXED_CHECKPOINT]` — HARD stop, operator must APPROVE. Use ONLY at the end of modules **01, 02, 03, 06, 11, 15** (foundational architecture, data model, exchange/auth surface, risk engine, Telegram owner-auth, live enablement).
- `[CHECKPOINT]` — SOFT, auto-approved after 30s. End every other module with this.
- `[HUMAN_REQUIRED]` — hand-off, not an approval: use when an action you cannot perform is needed (BotFather bot creation, DNS/SSL/Cloudflare, Binance API key creation, server/infra failures) or when a deploy error is NOT code-fixable.

## Deploy Error Handling

If a deploy/CI failure occurs and you are asked to fix it: classify first.
- **Code-fixable** (build error, missing dependency, bad script, Prisma client not generated, path issue): fix it, max 2 attempts.
- **Not code-fixable** (server down, SSH/secret/auth failure, DNS, disk full, infrastructure): do NOT attempt; end with `[HUMAN_REQUIRED]`.

## Per-Module Workflow

1. Read the module spec fully. List its acceptance criteria as your checklist.
2. Implement exactly what the spec defines (schema, interfaces, edge cases). Deviations require a logged reason in PROGRESS-HISTORY.md; if a deviation touches money behavior, risk limits, or auth — STOP with `[HUMAN_REQUIRED]` instead of deciding yourself.
3. **Verification (required, per module):**
   - `pnpm typecheck` and `pnpm build` pass across the workspace.
   - Module unit tests written and passing (indicator math vs known values; risk sizing vs hand-computed examples; filter pipeline vs constructed candle fixtures; strategy triggers vs fixture sequences; exchange rounding vs real Binance filter examples).
   - For UI modules: the design-quality self-check below.
   - Run the app in PAPER mode and confirm no runtime errors on boot (post-module-10).
4. Update PROGRESS.md + PROGRESS-HISTORY.md, commit, print checkpoint marker.

## Definition of Done (every module)

Spec's acceptance criteria all met • typecheck/build/tests green • no secrets introduced • PROGRESS files updated • integration with previously built modules verified (imports resolve, boot succeeds) • checkpoint marker printed.

## Design Quality (UI modules — mandatory)

"Simple" means easy for a non-technical operator, NOT plain-looking. Dashboard modules must use the real component system (Tailwind + shadcn/ui), design tokens from ARCHITECTURE.md §9, dark/light modes, smooth transitions, **animated charts** (Recharts) wherever data is shown, **skeleton loaders** instead of blank states, and responsive layout (operator uses phone browser). A functional-but-ugly page is NOT done — the module's verification includes a self-check against this paragraph. Telegram UX equivalent: clean formatted messages (bold/emoji structure per spec 11), inline keyboards for choices, and ALL outbound messages through `sendLongMessage()` (≤3,900-char parts, split at line boundaries, `(1/n)` labels, rate-limit-aware delays).

## Build Tooling / Deploy Rules (learned hard rules — bake into deploy.sh and CI)

(a) Build tooling lives in devDependencies → install MUST include dev deps even when NODE_ENV=production (`npm ci --include=dev` / pnpm default install). (b) Run `npx prisma generate` AFTER install and BEFORE typecheck/build in BOTH CI and deploy.sh. (c) Repo is cloned onto the server ONCE before first deploy; deploy.sh only pulls/resets. (d) Alphanumeric DB passwords only. (e) aaPanel server: psql/node/nginx live under `/www/server/...`, not system PATH — use the Node-detection block. (f) Reverse-proxy path mapping must match the dashboard's actual route prefix. (g) `pm2 restart` (not reload), fresh `BUILD_ID` before build, `rm -rf .next` before dashboard build.

## Trading-Specific Safety (unique to this project)

- **Graceful shutdown:** on STOP/restart/deploy, persist all state, leave exchange-side stop orders in place, never leave an untracked open position.
- **Reconcile on boot:** DB positions vs real Binance balances + open orders (module 03/15 logic). Discrepancy → alert, mark position `external_close`, never crash.
- **Stops are two-layer:** exchange stop-limit (with buffer) + bot watchdog force-market-sell. Both must exist for every live position.
- **Indicator warm-up:** no signals for a pair until ≥500 klines per required timeframe are loaded.
- **Staging screenshot token (dashboard only):** enabled ONLY when `APP_ENV === "staging"`, fail-closed (missing/other value = OFF), read-only preview via `?screenshot_token=<token>`, token from env, documented in `.env.example`, never hardcoded, never active in production.
