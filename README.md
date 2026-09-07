# RAMO Smart Money Base v1.3.0

Base Mainnet-only Smart Money discovery + analysis + live monitoring agent.

## What changed in v1.3

V1.3 replaces the slow one-block-at-a-time historical path with an **event-first batch discovery path** while preserving the existing SQLite database and per-DEX checkpoints.

Historical discovery now:

1. calls `eth_getLogs` for known V2/V3/Solidly/v4 `Swap` event signatures over a block range;
2. automatically splits the range if the RPC provider says the log result is too large;
3. deduplicates transaction hashes;
4. batch-fetches transaction envelopes;
5. keeps only transactions whose top-level `to` is one of the configured Base DEX entrypoints;
6. batch-fetches receipts only for those router transactions;
7. batch-fetches the needed block headers for timestamps;
8. sends the proven transaction + receipt through the same conservative wallet-flow detector used before.

This is materially faster on GitHub Actions because it avoids downloading every full Base block and avoids one HTTP request per receipt/header.

## Why this does not weaken BUY/SELL quality

The V1.2 detector only classified a transaction as BUY/SELL when a known pool `Swap` event was present. Transactions without a recognized Swap event could only become `UNKNOWN` and were excluded from Smart Money scoring and alerts.

Therefore the V1.3 event-first historical provider skips work that was not score-eligible under the existing detector rules. The strict wallet token-flow detector remains unchanged.

## Checkpoint compatibility

V1.3 uses the same SQLite schema and the same keys:

- `discovery_last_block:<dex>`
- `discovery_last_block`
- initial discovery anchor
- token metadata cache
- wallets/trades/positions/alerts

A V1.2 GitHub Actions cache can be restored directly. Completed V1.2 progress is not intentionally reset.

## Fast discovery settings

The GitHub workflow defaults are:

```text
DISCOVERY_PROVIDER=event_logs
DISCOVERY_EVENT_BATCH_BLOCKS=2000
DISCOVERY_EVENT_MIN_BLOCKS=25
RPC_BATCH_SIZE=25
RPC_BATCH_HTTP_REQUESTS_PER_SECOND=1.5
RPC_MAX_REQUESTS_PER_SECOND=5
```

If `eth_getLogs` returns too many results, the range is split automatically. If a minimum-sized range still cannot be queried, only that slice falls back to full-block scanning instead of crashing the whole run.

If the provider returns HTTP 429, the existing exponential backoff remains active.

## Expected startup lines

```text
RAMO Smart Money Base v1.3.0 starting
Connected to Base (chain_id=8453)
RPC pacing enabled: scalar max 5.00 req/s; batch max 1.50 HTTP req/s; batch_size=25
Historical provider: FAST EVENT-LOG discovery (eth_getLogs + batch tx/receipt/header fetch)
Scanning Base historical blocks ...
Fast historical event range ...
FAST EVENT BATCH ... | swap_logs=... unique_swap_txs=... router_txs=... successful=...
```

## Progress telemetry

Every 10,000 completed historical blocks the agent reports:

```text
DISCOVERY PROGRESS | run_blocks=10000 swap_event_logs=... unique_swap_txs=... router_txs=... successful_router_txs=... run_BUY=... run_SELL=... run_UNKNOWN=... DB_trades=... candidates=...
FAST DISCOVERY STATS | event_windows=... splits=... fallback_blocks=... inserted=... duplicates=...
DEX ROUTER CALLS | ...
```

`UNKNOWN` is still deliberately excluded from Smart Money scoring and alerts.

## Active Base DEX entrypoints

Built-in defaults remain enabled for:

- Uniswap V2 Router02
- Uniswap V3 SwapRouter02
- Uniswap Universal Router
- Uniswap Universal Router 2.1.1
- Aerodrome classic Router
- Aerodrome Universal Router
- Aerodrome Slipstream SwapRouter

BaseSwap remains operator-supplied through `DEX_BASESWAP_ROUTER`.

## Smart Money analysis

After the first historical catch-up finishes, candidate wallets are analyzed using FIFO realized PnL, ROI, win rate, profit factor, trade count, consistency, drawdown and lucky-wallet concentration penalties. Only wallets satisfying configured minimums become `is_smart_money=1`.

The real-time Base monitor and Telegram 3-wallet cluster alerts remain separate from historical discovery.

## RPC limitation

This remains an RPC-only architecture. `eth_getLogs` is much more efficient than full block crawling, but a provider can still impose response-size, compute-unit or rate limits. V1.3 adapts log ranges and batch sizes conservatively. At larger scale, the `HistoricalProvider` interface can still be replaced by an indexer without changing scoring, database, FIFO, monitoring or alerts.
