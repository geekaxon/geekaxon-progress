# PROGRESS.md — SpotEdge Trading Bot

**Status:** Module 09 (Strategy C — Core Leveling) DONE/APPROVED (2026-07-06). NEXT → Module 10 Paper-trading engine — spec /specs/10-paper-engine.md.
**Mode policy:** PAPER only until module 15 approved + owner enables LIVE via Telegram
**Branch:** staging | **Server:** Oracle VPS (GarageBrainPro host) | **Repo:** PRIVATE
**HARD checkpoints ([FIXED_CHECKPOINT]):** modules 01, 02, 03, 06, 11, 15
**[HUMAN_REQUIRED] hand-offs expected:** 11 (BotFather), 14 (DNS/SSL/noindex), 15 (Binance API key + LIVE pre-flight)

## Build Order Checklist

- [x] 01 Foundation — DONE/APPROVED 2026-07-06 — monorepo + config/logging/scheduler/state shell; typecheck/build/28 tests green; boots + graceful shutdown verified
- [x] 02 Database — DONE/APPROVED 2026-07-06 — Prisma schema (10 models/7 enums), init migration, seed (settings + 15 pairs), pnl.ts, in-bot 03:30-UTC backup; typecheck/build green, 46 tests (db 14 + bot 32), boot+shutdown verified
- [x] 03 Exchange connector — DONE/APPROVED 2026-07-06 — BinanceConnector (ccxt+ws) in @spotedge/core: filter compliance (roundQty/roundPrice/minNotional/TP1-split), clientOrderId idempotency, market fills w/ USD fees, stop-limit+watchdog seam, WS resilience+backoff, clock sync, BNB reserve, reconcile (EXTERNAL_CLOSE), PAPER/LIVE ExecutionRouter; typecheck/build green, 88 tests (core 42 + bot 32 + db 14), boot+shutdown verified; testnet integration deferred to server deploy
- [x] 04 Indicator engine — DONE/APPROVED 2026-07-06 — @spotedge/core indicators/: math (SMA-seeded EMA, Wilder RSI/ATR/ADX, VolumeAvg20+ratio, emaRising), structure (no-repaint pivots, body-based S/R zones + srFlip, swings/fibZone, range detector w/ 3×ATR + invalidation, bullish engulfing, higher-low+minor-high break, H4 bearish rejection), CandleStore (600-window ×TF, 500-kline warm-up gate on 1h/4h, onClose append + gap-detect/refetch, snapshot → FilterState sink); typecheck/build green, 127 tests (core 81 = +39 indicator, bot 32, db 14); 15-pair snapshot smoke green
- [x] 05 Filter pipeline — DONE/APPROVED 2026-07-06 — @spotedge/core filters/pipeline.ts: ordered veto chain L0 system gates → L1 BTC market (skips BTCUSDT) → L2 trend bias → L3 ADX regime (routes A/B) → L4 S/R 1:2 geometry → L5 discount (WAITING/IN_ZONE/OVERSHOT); first hard-veto short-circuits + named in `blocking`; single FilterState JSON for both renderers; runFilterCycle persists per-pair, exceptions → DATA veto, never throws; typecheck/build green, 157 tests (core 111 = +30 pipeline, bot 32, db 14)
- [x] 06 Risk engine — DONE/APPROVED 2026-07-06 — @spotedge/core risk/: PURE sizing.ts (PDF formula, tier caps T1 15/T2 10/T3 5%, floored qty so effective risk ≤ intent, computeNetRR w/ round-trip fees, clamp 0.25–2%, SETRISK/SETCAPITAL guards) + engine.ts (ordered 7-check approveEntry veto chain, approveLevelingSell ≤30% + one-owner rule, RiskEngine orchestrator: compounding capital, breaker trip/reset, reporting-only streak, hourly equity + LEDGER_DRIFT); 193 tests
- [x] 07 Strategy A (Trend-Pullback) — DONE/APPROVED 2026-07-06 — @spotedge/core strategies/: StrategyPlugin interface + StrategyRegistry (central ENABLE/DISABLE + allowedTiers gate before checkEntry) + strategyA.ts (PURE checkEntry — 1H engulfing[vol≥1.3]/higher-low-break trigger inside discount zone, structural stop floored at 1×ATR(H4), TP1 2×/A+ 3× risk, A+ H4-support proximity; PURE manageExit — TP1 50%+breakeven, higher-low trail −0.25×ATR(1H), SL, 24-H4 time stop); Position.trailHistory Json migration; typecheck/build green, 214 tests (core 168 = +21 strategy-A, bot 32, db 14)
- [x] 08 Strategy B (Range Reclaim) — DONE/APPROVED 2026-07-06 — @spotedge/core strategies/strategyB.ts: PURE checkEntry (resolveRange → floor/ceiling zones from H4 pivots, ≥2 touches, height≥3×ATR via detectRange, freshness gate; armed floor TOUCH RSI(14,H4)<35 within 12 H4; fresh 1H RECLAIM above floor-zone top; two-layer stop min(floorBottom−0.25×ATR, entry−1×ATR), TP1=range midpoint, TP2=ceiling-zone bottom) + PURE manageExit (SL / RANGE_TP+breakout / TP2 RANGE_TP / REGIME_FLIP kill-switch ADX≥20 on H4 close / TP1 50%+breakeven / TIME_STOP 24-H4); allowedTiers=[T1,T2] (registry-enforced); EntryProposal.tp2Price/ceilingTop + Position.tp2Price/ceilingTop migration; typecheck/build green, 241 tests (core 195 = +27 strategy-B, bot 32, db 14)
- [x] 09 Strategy C (Core Leveling) — DONE/APPROVED 2026-07-06 — @spotedge/core strategies/strategyC.ts (T1-only LevelingPlugin, distinct from A/B): PURE checkLevelingSell (bearish H4 rejection inside a resistance zone → sell 30% of core bag; RUNAWAY-BULL BLOCK when this/prev H4 body-closed above the zone) + PURE checkRebuy (FORCED insurance ≥sellPrice×1.05 on any TICK / STRUCTURAL 61.8%-or-support discount + fresh 1H reclaim on close / TIMEOUT 90-H4 → CYCLE_TIMEOUT) + chunkPnl (proceeds−cost−fees, decimal-safe); forcedRebuyTrigger/inRebuyDiscount/reclaimedC helpers; risk engine's approveLevelingSell (module 06) sizes the chunk + one-owner/breaker/minNotional; CycleStatus+EXTERNAL & LevelingCycle.exitReason migration (20260706000003); typecheck/build green, 268 tests (core 222 = +27 strategy-C, bot 32, db 14)
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
