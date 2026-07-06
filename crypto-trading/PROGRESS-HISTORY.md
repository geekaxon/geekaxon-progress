# PROGRESS-HISTORY.md — SpotEdge Trading Bot (detailed log)

> PROGRESS.md keeps one-liners; this file keeps the full story. The agent appends a dated entry after EVERY module: what was built, files touched, decisions/deviations (with reasons), verification results, and any open items handed to later modules.

## Entry template

```
## [YYYY-MM-DD] Module NN — <name> — <DONE | PARTIAL | BLOCKED>
**Built:** …
**Files:** …
**Decisions/Deviations:** … (none, or reason logged; money/risk/auth deviations require [HUMAN_REQUIRED] instead)
**Verification:** typecheck ✅/❌ · build ✅/❌ · tests <n> passing · UI design self-check (if UI) ✅/❌ · boot in PAPER ✅/❌
**Open items → later modules:** …
**Checkpoint printed:** [FIXED_CHECKPOINT] / [CHECKPOINT] / [HUMAN_REQUIRED]
```

## History

### [2026-07-06] Module 01 — Foundation — DONE
**Built:** pnpm/Turborepo monorepo scaffold with four workspaces (`apps/bot`, `apps/dashboard`, `packages/core`, `packages/db`). Bot engine shell that later modules plug into:
- `config.ts` — dotenv + zod, fail-fast with a NAMED list of missing/invalid vars; env holds only secrets/infra; secret-masked `summarizeConfig()`; refuses `MODE=LIVE` from env.
- `logger.ts` — pino, levels debug/info/warn/error, JSON to `logs/bot.log` (daily rotation via pino-roll, keep 14) + pretty console in dev, `category` field (engine|exchange|strategy|risk|telegram|scheduler|reconcile), `logEvent()` with a deferred `events_log` DB sink seam (`attachEventSink()` for module 02).
- `scheduler.ts` — UTC EventEmitter emitting `close_1h` (+5s grace), `close_4h` (00/04/08/12/16/20 UTC +5s), `daily_rollover` (00:00 UTC), `summary_time` (PKT setting → UTC minutes); self-correcting, ref'd timers (engine heartbeat).
- `state.ts` — `StateMachine` over PAPER/LIVE/PAUSED/STOPPED + PAPER_DRAINING/LIVE_DRAINING; persistence seam (`StateStore`, in-memory now / DB in module 02); transition legality table; **LIVE gate guard** — LIVE/LIVE_DRAINING refused unless the module-15 owner gate is open.
- `utils/money.ts` (decimal.js, no raw float math; `roundToStep` for LOT_SIZE), `utils/time.ts` (UTC↔PKT, `pktTimeToUtcMinutes`), `lock.ts` (PID single-instance guard, stale-lock reclaim), `shutdown.ts` (SIGINT/SIGTERM → quiesce → persist → stop scheduler → release lock → exit 0, with hard-kill backstop), `index.ts` boot sequence.
- Root: `package.json`, `pnpm-workspace.yaml`, `turbo.json`, `tsconfig.base.json`, `.env.example` (all placeholders), `.gitignore`, `deploy.sh` (ARCHITECTURE.md §8: node-detection block, prisma steps as commented placeholders until module 02, BUILD_ID, pm2 restart+save, health-check notes), `ecosystem.config.js` (trading-bot active; trading-dashboard commented for module 14). Dashboard = Next.js 15 placeholder (noindex header + robots metadata, single scaffold page).
**Files:** `package.json`, `pnpm-workspace.yaml`, `turbo.json`, `tsconfig.base.json`, `.env.example`, `.gitignore`, `deploy.sh`, `ecosystem.config.js`; `apps/bot/{package.json,tsconfig.json,src/{index,config,logger,scheduler,state,lock,shutdown}.ts,src/utils/{money,time}.ts,src/__tests__/{config,money,time,scheduler,state}.test.ts}`; `apps/dashboard/{package.json,next.config.mjs,tsconfig.json,next-env.d.ts,app/{layout,page}.tsx}`; `packages/core/{package.json,tsconfig.json,src/index.ts}`; `packages/db/{package.json,tsconfig.json,src/index.ts}`; `pnpm-lock.yaml`.
**Decisions/Deviations:** (1) Spec header says end with `[FIXED_CHECKPOINT]`, but continuous-development mode is enabled by the controller → ended with `[CHECKPOINT]` per controller directive (no owner hard-stop). (2) decimal.js imported as named `{ Decimal }` (NodeNext default import resolves to a namespace, not the class). (3) Logger uses default sync stdout + silent level under `VITEST` to avoid worker-thread transports in tests. No money/risk/auth behavior deviations.
**Verification:** typecheck ✅ · build ✅ (bot tsc + dashboard `next build` both green) · tests 28 passing (config/money/time/scheduler/state) ✅ · boot in PAPER ✅ (logs secret-masked config summary, scheduler armed with next UTC fire times, stays alive as daemon, SIGINT → graceful shutdown persists state + removes lock + exit 0) · double-boot guard ✅ (2nd instance exits 1) · `bash -n deploy.sh` ✅ · UI design self-check N/A (scaffold only).
**Open items → later modules:** module 02 injects the DB-backed `StateStore` + `events_log` sink (`attachEventSink`) and uncomments the prisma steps in deploy.sh/CI; module 03 replaces the clock-drift stub with the real NTP check; runtime settings (capital, risk %, summary time, toggles) move into DB `settings`; module 14 builds the real dashboard + uncomments the PM2 dashboard entry; DASHBOARD_PORT value still `[DECIDE AT BUILD]`.
**Checkpoint printed:** [CHECKPOINT]

### [2026-07-06] Planning — DONE
Strategy layer locked (A/B/C reworked from source PDF: candle-color bias, 5m order blocks, body-midpoint equilibrium, 100% leveling sells, manual TradingView zones all removed; BTC filter, 1% sizing, structural stops, fee filter, drawdown discipline kept). Build order (15 modules), tiers, risk parameters, Telegram command set, dashboard scope, mode-switch state machine, and 7 real-world hardening items (exchange filters, stop-limit+watchdog, WS resilience+NTP, BNB fee reserve, indicator warm-up, dust/fill tracking, DB backups) folded into specs. Deliverables produced: ARCHITECTURE.md, AGENT.md, PROGRESS.md, PROGRESS-HISTORY.md, /specs/01–15, /specs/README-OPERATOR-GUIDE.md.
