# RAMO Smart Money Base v1.2.0

Base Mainnet-only Smart Money discovery + analysis + live monitoring agent.

## What changed in v1.2

V1.2 is a stability/recovery release for long GitHub Actions historical scans.

- **Global RPC pacing** is enabled. Default: `RPC_MAX_REQUESTS_PER_SECOND=5`.
- **429-specific exponential backoff** is separate from ordinary RPC retries.
- A `Retry-After` header is respected when the provider sends one.
- `symbol()` is no longer called during historical discovery. Discovery needs token decimals, not symbols, so this removes a large amount of unnecessary RPC traffic.
- Successful token decimals are cached in SQLite across GitHub runner windows.
- Failed token metadata lookups are negatively cached for `TOKEN_METADATA_FAILURE_TTL` (default 6 hours), preventing the same non-standard token from repeatedly exhausting RPC quota.
- Direct-router `tx.from` is treated as the top-level transaction origin by default, so discovery no longer performs an `eth_getCode` for every new wallet. Optional code verification can be enabled with `VERIFY_EOA_CODE=1`; those results are persisted in SQLite.
- V1.2 repairs the classic Aerodrome checkpoint using the last fully completed V1 batch confirmed in the supplied GitHub log. It **does not** move new Uniswap/Aerodrome entrypoint cursors forward.
- Initial historical discovery uses a persistent anchor so a multi-day GitHub backfill does not silently shrink as the rolling 30-day timestamp moves forward.
- Per-router checkpoints, INFO progress telemetry, FIFO PnL, Smart Money scoring and live monitoring remain compatible with the existing SQLite cache.

## Expected startup lines

```text
RAMO Smart Money Base v1.2.0 starting
Connected to Base (chain_id=8453)
RPC pacing enabled: max 5.00 requests/second, exponential 429 backoff enabled
V1.2 checkpoint recovery: classic Aerodrome cursor ...
Scanning Base historical blocks ...
```

## Progress telemetry

Every `DISCOVERY_PROGRESS_BLOCKS` (default 10,000) scanned blocks, the agent reports:

```text
DISCOVERY PROGRESS | run_blocks=10000 router_txs=... successful_router_txs=... run_BUY=... run_SELL=... run_UNKNOWN=... run_unique_wallets=... eoa_skipped=... DB_trades=... candidates=...
DEX ROUTER CALLS | aerodrome=... uniswap_v3=... uniswap_universal=...
```

`UNKNOWN` is intentional. Ambiguous swaps or swaps whose token decimals cannot be proved are excluded from Smart Money scoring and alerts.

## Active Base DEX entrypoints

Built-in verified defaults remain enabled for:

- Uniswap V2 Router02
- Uniswap V3 SwapRouter02
- Uniswap Universal Router
- Uniswap Universal Router 2.1.1
- Aerodrome classic Router
- Aerodrome Universal Router
- Aerodrome Slipstream SwapRouter

BaseSwap remains operator-supplied with `DEX_BASESWAP_ROUTER`.

## Historical vs live architecture

Historical discovery and real-time monitoring are separate. Historical discovery uses the RPC provider and durable per-router cursors. Once the initial catch-up finishes, Smart Money analysis runs and the live Base monitor starts.

The historical provider interface can later be replaced by an indexer without changing scoring, FIFO, database, monitoring, cluster detection or Telegram alert logic.

## Smart Money scoring

A wallet must satisfy the configured minimum trade count, realized ROI, win rate and profit factor. The score also considers consistency, drawdown and concentration of profits so a wallet that made most of its money from one lucky trade receives a penalty.

## PnL

Realized PnL is FIFO. USD cost/proceeds are only used where the stablecoin leg is directly proven by on-chain wallet flows. Unknown price basis is not fabricated. Generic RPC mode does not invent historical unrealized PnL.

## Important V1.2 RPC settings

Defaults used by the GitHub workflow:

```text
RPC_MAX_REQUESTS_PER_SECOND=5
RPC_MAX_RETRIES=6
RPC_429_BACKOFF_BASE=4
RPC_BACKOFF_MAX=120
TOKEN_METADATA_FAILURE_TTL=21600
VERIFY_EOA_CODE=0
```

If the RPC provider still returns frequent 429 errors, lower `RPC_MAX_REQUESTS_PER_SECOND` to `3` or `4`. If it is stable for many hours, the rate can later be raised cautiously.

## Run locally

Python 3.10+:

```bash
pip install -r requirements.txt
export BASE_RPC_URL='YOUR_BASE_RPC_URL'
export TELEGRAM_BOT_TOKEN='YOUR_BOT_TOKEN'
export TELEGRAM_CHAT_ID='YOUR_CHAT_ID'
python -u main.py
```

GitHub runs use `--runtime-minutes 340`, allowing a graceful shutdown before the hosted runner hard timeout so SQLite and checkpoints can be saved.
