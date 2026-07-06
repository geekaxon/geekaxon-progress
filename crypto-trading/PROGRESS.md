# PROGRESS.md — SpotEdge Trading Bot

**Status:** NOT STARTED — planning complete, specs uploaded, awaiting module 01
**Mode policy:** PAPER only until module 15 approved + owner enables LIVE via Telegram
**Branch:** staging | **Server:** Oracle VPS (GarageBrainPro host) | **Repo:** PRIVATE
**HARD checkpoints ([FIXED_CHECKPOINT]):** modules 01, 02, 03, 06, 11, 15
**[HUMAN_REQUIRED] hand-offs expected:** 11 (BotFather), 14 (DNS/SSL/noindex), 15 (Binance API key + LIVE pre-flight)

## Build Order Checklist

- [ ] 01 Foundation — not started — [FIXED_CHECKPOINT]
- [ ] 02 Database — not started — [FIXED_CHECKPOINT]
- [ ] 03 Exchange connector — not started — [FIXED_CHECKPOINT]
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
