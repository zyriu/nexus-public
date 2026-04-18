# Nexus — Overview

**Context:** Self-hosted crypto / DeFi portfolio and trading system
**Purpose:** A personal Bloomberg terminal — wallet tracking, DeFi positions, live price streams, token scanning, and paper/live trading, all running on your own machine
**Key idea:** Dozens of small services orchestrated from one place; everything surfaced through a single dashboard

---

## What it is

Nexus watches your crypto wallets, streams prices from exchanges, fetches DeFi positions across protocols, scans for new tokens, and runs trading strategies against the live market. Everything is local — no third party sees your addresses, keys, or trades — and everything is visible through one dashboard.

A central orchestrator supervises a few dozen small services, each responsible for one thing: pulling balances, streaming prices, auditing a new token, evaluating a strategy, executing a trade, backing up a spreadsheet. Some run continuously; others run on a schedule. The orchestrator keeps them healthy, handles updates, and brings them down cleanly when you stop.

## Why it exists

Off-the-shelf portfolio trackers are either too limited (they don't understand most DeFi), too expensive (paywalls on basics), or trust-dependent (they want your addresses and API keys). Nexus keeps all of that on your machine. You decide what gets fetched, how often, and where it goes — and you own every byte of data it produces.

---

## What it does

### Portfolio tracking

- Any number of EVM and Solana wallets, each labeled and individually configurable
- Native + token balances across every supported chain
- CEX balances (Kraken spot) and Hyperliquid account state — spot, perps, vaults
- DeFi yield positions across Pendle (YT / PT / LP), IPOR, Kamino, and Etherfi (eETH + sETHFI)
- NFT holdings with USD floor prices
- **Per-wallet schedules** — you set how often each kind of check runs per wallet, so cheap lookups can run often and expensive ones stay rare

### Real-time prices

- A live ticker streaming crypto and tradfi prices from multiple venues
- Sources: Binance, Hyperliquid, Finnhub — the ticker shows which source a given symbol is on, with a fallback suffix when it's on a backup feed
- Historical all-time-high tracking per asset
- Price history stored so indicators can be computed over any window

### Token scanning and safety auditing

- Discovers new and trending tokens across Solana, Ethereum, Arbitrum, Base, and Hyperliquid
- Audits every token through two independent safety sources and merges their signal — mint / freeze authority, mutability, honeypot, rug status, LP lock, holder concentration, insider and bundled buys, sniper activity, phishing patterns, dev holdings, accumulated fees
- **Scan profiles** are named threshold sets (market cap, volume, liquidity, age, authority rejections, concentration limits, etc.) configurable from the UI
- Tokens that fail a profile are remembered so the scanner doesn't re-check them forever; rules that can plausibly flip (volume, fees, LP) get periodic re-audits
- Scan profiles connect directly to trading strategies — a strategy can be scoped to "only fire on tokens matching profile X"

### Trading

- **Paper and live modes per strategy** — run a strategy against simulated fills while you validate it, then flip it to live when you trust it
- **Strategy builder** — compose entry and exit conditions from an indicator library: RSI, EMA, SMA, MACD (with signal-line crossovers), Bollinger bands, ATR, ATR %, SuperTrend, VWAP, price %-moves over a window, ATH / ATL, plus price action
- **Sustained conditions** — any rule can require N consecutive candles before it fires, to cut noise
- **Stop-loss and take-profit per strategy** — each fixed or trailing; trailing take-profit only arms once the trade is already in profit
- **Daily loss cap per strategy** — includes open losing positions, not just realized ones, so bleeding trades can't keep stacking new entries
- **Max open positions** and a **duplicate-position guard** — the same strategy can't open the same market twice
- **Manual overrides** — close any open trade from the UI, or edit a single open trade's exit rules without touching the strategy (handy for tightening stops on one specific position)
- **Live execution on Meteora DLMM** (Solana concentrated-liquidity pools, one-sided entries) — the system owns the signing, broadcasts through its own RPC path, waits for on-chain confirmation, and records the real transaction hash on the trade
- **Safety gates around live execution** — stale-price rejection, authoritative blacklist re-check at execution time, wallet binding snapshotted at entry (so wallet renames can't redirect an exit), and crash recovery (if the system dies mid-entry, it picks up on next boot and either finishes the trade or flags a stranded position for you)
- **Signal audit log** — every decision (accepted, rejected, and why) is logged and browsable
- **Strategy performance** page — per-strategy hit rate, realized PnL, full signal trail

### DeFi yield and rewards

- Tracks positions across Pendle, IPOR, Kamino, and Etherfi
- Captures deposit amount, current value, implied yield, claimable and already-claimed rewards, deposit date, expiry
- On-chain reward verification for Solana positions — you see the real claimable balance, not a cached estimate

### Spreadsheet export (Grist)

- Pushes everything Nexus tracks into a Grist document — prices, positions, yield positions, NFTs, the token list
- The spreadsheet can drive configuration (what assets to track) while Nexus drives the data
- Scheduled backups of the Grist document to your disk

### Dashboard

- One page for every moving piece:
  - **Services** — live health of everything running, with countdowns to the next scheduled run
  - **Wallets** — add, remove, relabel, set per-wallet schedules
  - **Prices** — live ticker across crypto and tradfi
  - **Trading** — active trades, trade history, strategies, per-strategy performance, signal audit log
  - **Scanner** — scan profiles and the list of discovered / audited tokens
  - **Database** — raw table explorer for anything Nexus stores
  - **Logs** — live per-service logs
  - **Settings** — appearance, secrets (API keys), system
- **Boot screen** that tracks actual service readiness rather than a guessed timer
- **Themes** (Nexus and Brutal) with light / dark modes, a per-theme display font, persisted locally
- **Update progress** surfaces in the UI — when a new release is installing, every step is visible
- **Accessibility** — reduced-motion respected, responsive down to mobile widths, forced-colors compatible
- **Lock screen** — if an API key is set, the dashboard requires it and can be locked from the header
- **Shutdown** — bring the whole stack down cleanly from the UI

### Orchestration, updates, resilience

- **One manifest file** defines every service — its schedule, its dependencies, which machines and profiles it runs under
- **Machine profiles** — the same codebase can run the full stack, only the portfolio subset, or only the trading subset, per machine
- **Dependency-aware startup and shutdown** — a task waits for what it depends on to be healthy before running; the orchestrator keeps dependencies alive until dependent tasks finish (up to 10 minutes)
- **Self-updating** — watches a release feed, downloads, checksum-verifies, swaps binaries in place, rolls back on failure, and surfaces progress in the UI
- **Secrets in one place** — API keys live in a single store; services pick up rotations within a minute, no restart
- **API key protection** on every endpoint except health
- **Localhost-only by default** — nothing is exposed to the internet unless you tunnel it yourself

---

## What it talks to

- **Chains** — Ethereum, Arbitrum, Base, Polygon, BSC, Optimism, Solana, Hyperliquid
- **DeFi protocols** — Pendle, IPOR, Kamino, Etherfi, Meteora DLMM
- **Exchanges** — Kraken, Hyperliquid
- **Token discovery and safety** — DexScreener, RugCheck, GMGN, CoinGecko
- **Price feeds** — Binance, Hyperliquid, Finnhub
- **Wallet / balance data** — Zerion, CoinStats
- **RPC providers** — Alchemy, dRPC, Infura, Helius
- **Spreadsheet** — Grist

---

## What it doesn't do yet

- Swap-router execution (Jupiter on Solana, 0x / 1inch on EVM) — Meteora LP entries are live, but swap-based live trading isn't
- CEX order placement — live trading is LP-only today; CEX strategies stay in paper mode
- Historical backtesting — strategies can run in paper mode against the live feed, but there's no replay suite yet
- Alerting — no push notifications or webhooks on price moves, outages, or trade events
- Remote access — the dashboard is localhost-only; bring your own tunnel