# PROGRESS.md — SpotEdge Trading Bot

**Status:** Module 03 (Exchange connector) DONE/APPROVED (2026-07-06). NEXT → Module 04 Indicator engine (EMA/RSI/ADX/ATR + ≥500-kline warm-up) — spec /specs/04-indicators.md.
**Mode policy:** PAPER only until module 15 approved + owner enables LIVE via Telegram
**Branch:** staging | **Server:** Oracle VPS (GarageBrainPro host) | **Repo:** PRIVATE
**HARD checkpoints ([FIXED_CHECKPOINT]):** modules 01, 02, 03, 06, 11, 15
**[HUMAN_REQUIRED] hand-offs expected:** 11 (BotFather), 14 (DNS/SSL/noindex), 15 (Binance API key + LIVE pre-flight)

## Build Order Checklist

- [x] 01 Foundation — DONE/APPROVED 2026-07-06 — monorepo + config/logging/scheduler/state shell; typecheck/build/28 tests green; boots + graceful shutdown verified
- [x] 02 Database — DONE/APPROVED 2026-07-06 — Prisma schema (10 models/7 enums), init migration, seed (settings + 15 pairs), pnl.ts, in-bot 03:30-UTC backup; typecheck/build green, 46 tests (db 14 + bot 32), boot+shutdown verified
- [x] 03 Exchange connector — DONE/APPROVED 2026-07-06 — BinanceConnector (ccxt+ws) in @spotedge/core: filter compliance (roundQty/roundPrice/minNotional/TP1-split), clientOrderId idempotency, market fills w/ USD fees, stop-limit+watchdog seam, WS resilience+backoff, clock sync, BNB reserve, reconcile (EXTERNAL_CLOSE), PAPER/LIVE ExecutionRouter; typecheck/build green, 88 tests (core 42 + bot 32 + db 14), boot+shutdown verified; testnet integration deferred to server deploy
- [ ] 04 Indicator engine — not started
- [ ] 05 Filter pipeline — not started
- [ ] 06 Risk engine — not started — [FIXED_CHECKPOINT]
- [ ] 07 Strategy A (Trend-Pullback) — not started
- [ ] 08 Strategy B (Range Reclaim) — not started
- [ ] 09 Strategy C (Core Leveling) — not started
- [ ] 10 Paper-trading engine — not started
- [ ] 11 Telegram control panel — not started — [FIXED_CHECKPOINT]
- [ ] 12 Backtest module — not started
- [ ] 13 Reporting engine — not started
- [ ] 14 Web dashboard — not started
- [ ] 15 Live-trading enablement — not started — [FIXED_CHECKPOINT]

## Decisions / Assumptions Log

- Stack: Node 20 + TS, pnpm/Turborepo, ccxt, PostgreSQL + Prisma, Next.js 15 dashboard, dedicated Telegram bot.
- Capital: $11,000 paper / $11,000 live (allocated-capital model; wallet remainder untouched for manual trading).
- Risk: 1% per trade default (hard bounds 0.25–2%), tier caps 15/10/5%, net R:R ≥1.5 after fees, −3% daily circuit breaker, max 3 open positions.
- Tiers: T1 BTC/ETH/BNB/SOL • T2 DOT/LINK/AVAX/NEAR/TON/SUI/FIL/ATOM • T3 XLM/DASH/ALGO. GRAM + CVX excluded.
- Timezone: UTC engine, PKT display; daily summary default 21:00 PKT (operator-editable setting).
- Dashboard: trade.geekaxon.com, noindex (header+robots+meta+login wall), port [DECIDE AT BUILD].
- Strategy D/E: designed stubs only (ARCHITECTURE.md Appendix A), [DECIDE AT BUILD] after paper data.
- LIVE capital source & Binance sub-account: [DECIDE AT BUILD] (operator guide covers optional sub-account).
