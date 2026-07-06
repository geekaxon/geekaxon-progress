# ARCHITECTURE.md — Binance Spot Trading Bot ("SpotEdge")

> PRIVATE PROJECT. Never public. No secrets in repo (`.env.example` placeholders only).
> This file is the lean, always-loaded table of contents. **Full detail per module lives in `/specs/NN-slug.md` — the agent reads ONLY the current module's spec each session and treats it as AUTHORITATIVE.** If a spec is missing → STOP with `[HUMAN_REQUIRED]`.

## 1. Overview

Fully automatic, long-only (Halal, spot-only, no leverage/margin/futures/shorting) trading bot for Binance Spot. Trades a tiered list of USDT pairs using three locked strategies (A: Trend-Pullback Continuation, B: Range Reclaim, C: Core Position Leveling) behind a shared risk engine. Controlled entirely via a dedicated Telegram bot and a web dashboard (full control parity). Runs 24/7 under PM2 on the existing Oracle VPS (shared with GarageBrainPro).

- Modes: **PAPER** (default; this project's "staging") and **LIVE** (this project's "production" — owner-gated, never enabled by the agent).
- Capital: bot operates on **allocated capital** ($11,000 paper, $11,000 live default), never the full wallet balance. Paper and live ledgers fully separate.
- All strategy logic acts on **closed candles** (1H entries, H4 bias); open positions monitored in real time via WebSocket.
- Display timezone **PKT (GMT+5)**; all internal logic/scheduling **UTC**.

## 2. Principles

1. **Seek excuses NOT to trade.** Capital preservation > exposure. Every filter is a veto.
2. **Everything net-of-fees.** 0.1% taker both sides simulated/recorded everywhere (0.075% when BNB fee payment active). Skip setups with net R:R < 1.5.
3. **Deterministic & backtestable.** No subjective inputs (no manual zones, no discretionary anything). Backtest, paper, and live run the SAME strategy code path.
4. **Strategies are plugins.** Common interface (`requiredIndicators`, `checkEntry()`, `manageExit()`, tier permissions) + registry. Adding Strategy D/E later = one file + registry entry (stubs in Appendix A).
5. **Never abandon a position.** Mode switches affect new entries only; open positions are managed to natural exit. Restart = reconcile DB vs real Binance balances/orders before acting. Exchange-side stop-limit + bot-side watchdog = two-layer stops.
6. **Owner-only control.** Telegram commands and dashboard restricted to owner. Money-affecting actions double-confirm. LIVE mode = owner command only, never automatic, never by the agent.
7. **One service layer.** Telegram and dashboard call identical backend functions — behavior can never diverge.
8. **No secrets in repo.** Binance keys: spot trade + read ONLY, withdrawals disabled, IP-whitelisted (operator guide).

## 3. Tech Stack

- **Node.js 20 + TypeScript**, pnpm monorepo (Turborepo): `apps/bot`, `apps/dashboard`, `packages/core` (indicators/filters/strategies/risk — shared by bot & backtest), `packages/db` (Prisma).
- **ccxt** (npm) for Binance Spot REST; **ws / binance streams** for real-time.
- **PostgreSQL + Prisma** (existing server Postgres). Nightly pg_dump backup, 7-day rotation.
- **Dashboard:** Next.js 15, Tailwind, shadcn/ui, Recharts, dark default + light mode, skeleton loaders, responsive. Auth: single owner account, session-based.
- **Telegram:** dedicated bot (separate from bot-4), grammY or node-telegram-bot-api, global `sendLongMessage()` splitter (≤3,900 chars, line-boundary splits, part labels, rate-limit-aware).
- **PM2:** processes `trading-bot`, `trading-dashboard`. Ports/subdomain: dashboard `trade.geekaxon.com`, port `[DECIDE AT BUILD]`.

## 4. Data Model Summary (full schema in /specs/02-database.md)

`settings` (key/value: mode, capital, risk %, summary time, toggles) • `pairs` (symbol, tier, active, exchange filters cache) • `positions` (open state, strategy, entries/SL/TP, mode) • `trades` (closed, full P&L net of fees) • `fills` (every execution, actual qty/price/fee) • `leveling_cycles` (Strategy C) • `equity_snapshots` • `events_log` • `filter_state` (per-pair pipeline status for FILTERS/dashboard).

## 5. Coin Tiers (default seed; Telegram-manageable)

| Tier | Pairs (vs USDT) | Cap | Strategies |
|---|---|---|---|
| 1 | BTC, ETH, BNB, SOL | 15% | A + B + C |
| 2 | DOT, LINK, AVAX, NEAR, TON, SUI, FIL, ATOM | 10% | A + B |
| 3 | XLM, DASH, ALGO | 5% | A only + $20M 24h-volume pre-check |

Excluded by decision: GRAM, CVX. No meme/gaming coins.

## 6. Strategy & Risk Summary (authoritative detail in specs 05–09)

- **Filter pipeline:** BTC market filter (H4 EMA50 + slope) → coin trend bias (H4 EMA50>EMA200, price above) → regime (ADX14 H4: ≥20 → A; <20 → B) → S/R validation (pivot-based body zones; nearest resistance ≥ 2× stop distance) → discount retracement (38.2–61.8% fib of last H4 swing OR EMA20/50 band).
- **Strategy A:** 1H trigger (bullish engulfing w/ 1.3× volume OR higher-low + minor-high break) → SL below 1H swing low with ATR(14,H4) floor → TP1 at 1:2 (sell 50%, SL→breakeven) → trail 1H higher lows → time stop 24 H4 candles → A+ setups (entry on H4 support) target 1:3 first.
- **Strategy B:** ADX<20 only; H4 range (height ≥3×ATR); floor touch (H4 RSI<35) + 1H reclaim close; TP mid (50%) + ceiling; disabled instantly on ADX≥20 or floor break.
- **Strategy C (Tier 1 only):** lever max 30% of core bag; sell only on H4 bearish rejection closing inside resistance (breakout close through resistance = sales blocked); rebuy at support/61.8% + 1H reclaim; **forced rebuy at +5% above sell price** (defined small loss instead of "left behind").
- **Risk engine:** size = `(allocated_capital × risk%) / SL%`; risk default 1% (hard bounds 0.25–2%); tier caps; net-R:R ≥1.5 fee filter (incl. minNotional check for both TP1 halves); −3% daily circuit breaker (UTC day); max 3 open positions (correlation cap); 3–5 loss streaks = normal, no mid-drawdown parameter changes.

## 7. BUILD ORDER (one module per session, in order)

| # | Module | Spec | Checkpoint |
|---|---|---|---|
| 01 | Foundation (monorepo, config, logging, scheduler skeleton) | /specs/01-foundation.md | **[FIXED_CHECKPOINT]** |
| 02 | Database (Prisma schema, migrations, backup job) | /specs/02-database.md | **[FIXED_CHECKPOINT]** |
| 03 | Exchange connector (ccxt, filters, WS, stops, clock, BNB) | /specs/03-exchange-connector.md | **[FIXED_CHECKPOINT]** |
| 04 | Indicator engine (EMA/ADX/ATR/RSI/pivots/fibs/volume, warm-up) | /specs/04-indicators.md | [CHECKPOINT] |
| 05 | Filter pipeline | /specs/05-filter-pipeline.md | [CHECKPOINT] |
| 06 | Risk engine | /specs/06-risk-engine.md | **[FIXED_CHECKPOINT]** |
| 07 | Strategy A | /specs/07-strategy-a.md | [CHECKPOINT] |
| 08 | Strategy B | /specs/08-strategy-b.md | [CHECKPOINT] |
| 09 | Strategy C | /specs/09-strategy-c.md | [CHECKPOINT] |
| 10 | Paper-trading engine | /specs/10-paper-engine.md | [CHECKPOINT] |
| 11 | Telegram control panel | /specs/11-telegram.md | **[FIXED_CHECKPOINT]** |
| 12 | Backtest module | /specs/12-backtest.md | [CHECKPOINT] |
| 13 | Reporting engine | /specs/13-reporting.md | [CHECKPOINT] |
| 14 | Web dashboard | /specs/14-dashboard.md | [CHECKPOINT] (+ [HUMAN_REQUIRED] for DNS/SSL) |
| 15 | Live-trading enablement | /specs/15-live-enable.md | **[FIXED_CHECKPOINT]** |

HARD stops: **01, 02, 03, 06, 11, 15** (recorded in PROGRESS.md). [HUMAN_REQUIRED] hand-offs expected: BotFather setup (11), Nginx/subdomain/SSL/noindex (14), Binance API key creation + LIVE pre-flight (15). Operator steps consolidated in `/specs/README-OPERATOR-GUIDE.md`.

## 8. Deployment (deploy.sh spec — agent authors the real script per AGENT.md)

Branching: agent works on `staging` (auto-deploys on push). "Production" for this project = **LIVE trading mode**, enabled only by owner Telegram command — there is no auto path to LIVE. Repo cloned onto server ONCE manually before first deploy (deploy.sh only pulls/resets).

`deploy.sh` (repo root) must:
1. **Node detection block** — resolve node/npm/pnpm across: nvm (`~/.nvm/versions/node/*/bin`), aaPanel (`/www/server/nodejs/*/bin`), system (`/usr/local/bin`, `/usr/bin`); export PATH accordingly. (aaPanel server: psql/nginx also under `/www/server/...`.)
2. `git fetch && git reset --hard origin/staging` (or the deploy branch).
3. `pnpm install` including dev deps (build tooling lives in devDependencies — never prune dev for build; npm equivalent `npm ci --include=dev`).
4. `npx prisma generate` (AFTER install, BEFORE typecheck/build — in CI too), then `npx prisma migrate deploy`.
5. Set fresh `BUILD_ID` env before build; `pnpm build` (turbo: core → bot → dashboard; dashboard `rm -rf .next` first).
6. **Graceful bot restart:** signal `trading-bot` to quiesce (persist state; leave exchange-side stops in place; never leave an untracked position), then `pm2 restart trading-bot trading-dashboard` (restart, not reload) and `pm2 save`.
7. Health check: bot heartbeat within 60s + dashboard HTTP 200; on failure print logs and exit non-zero.
Notes: alphanumeric DB password (no `$`/backticks/quotes); Nginx proxy path must match the dashboard's actual route prefix; Cloudflare DNS Only on `trade.geekaxon.com` until SSL verified. Env: `APP_ENV` gates the staging-only screenshot token (fail-closed; dashboard only; never in LIVE/production context).

## 9. Design Tokens (dashboard)

Dark default. Tailwind + shadcn/ui defaults with: background zinc-950/zinc-900 cards, primary emerald-500 (profit), red-500 (loss), amber-500 (warnings/paper mode badge), violet-500 accents for Strategy identifiers (A=emerald, B=sky, C=violet). Inter font. Animated Recharts, skeleton loaders on every data view. Polished UI is part of Definition of Done (see AGENT.md).

## Appendix A — Deferred strategy stubs [DECIDE AT BUILD, post paper-data review]

- **Strategy D — Weekly DCA Accumulator:** fixed USDT weekly buy of BTC/ETH only when price < weekly EMA20; no SL (accumulation); separate allocation.
- **Strategy E — Volatility Breakout:** buy 20-day-high break with volume ≥1.5× 20-avg; SL below breakout level; only if backtests show Strategy A misses runners.
Both plug into the strategy registry; no core changes required.
