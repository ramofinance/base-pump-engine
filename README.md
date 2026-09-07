# RAMO Smart Money Base v1.4

A Base Mainnet Smart-Money discovery and monitoring agent designed for GitHub Actions.

## What changed in v1.4

V1.4 removes `eth_getLogs` from the default historical path. V1.3 proved that some hosted Base RPC plans reject broad Swap-log queries at the HTTP layer, even when the requested block range is reduced.

The new default is **Batched Full-Block Discovery**:

1. Fetch Base blocks with full transaction envelopes using JSON-RPC batches.
2. Keep only transactions whose `to` address is one of the configured DEX router entrypoints.
3. Fetch receipts only for those router transactions, also in JSON-RPC batches.
4. Run the same conservative wallet-flow BUY/SELL detector.
5. Save a checkpoint only after the whole block window is completely processed.

This keeps the correctness of v1.2 while avoiding one HTTP request per block.

## Important reliability behavior

- No broad `eth_getLogs` query is required.
- If the RPC rejects a JSON-RPC batch, the batch automatically splits into smaller batches.
- If one block is missing from a batch response, that block is retried through scalar RPC.
- If one receipt is missing from a batch response, that receipt is retried through scalar RPC.
- If the scalar fallback also fails, discovery stops and **does not advance the checkpoint past incomplete data**.
- 429 responses use exponential backoff.
- SQLite state is restored/saved through GitHub Actions cache.
- Existing v1/v1.1/v1.2/v1.3 database state remains compatible.

## Expected startup log

```text
RAMO Smart Money Base v1.4.0 starting
Connected to Base (chain_id=8453)
Historical provider: BATCHED FULL-BLOCK discovery (no eth_getLogs)
BATCH BLOCK RANGE ...
BATCH BLOCK DONE ...
```

Every 10,000 completed blocks the workflow prints:

```text
DISCOVERY PROGRESS | run_blocks=... router_txs=... successful_router_txs=... run_BUY=... run_SELL=... run_UNKNOWN=... candidates=...
BATCH DISCOVERY STATS | windows=... block_misses=... scalar_block_fallbacks=... scalar_receipt_fallbacks=...
DEX ROUTER CALLS | ...
```

## Required GitHub Secrets

Keep the same existing secrets:

- `SMART_MONEY_PACKAGE_KEY`
- `BASE_RPC_URL`
- `TELEGRAM_BOT_TOKEN`
- `TELEGRAM_CHAT_ID`

No secret needs to be changed when upgrading from v1.3.

## Upgrade from v1.3

Replace only:

- `smart_money_payload.enc`
- `README.md`
- `.github/workflows/smart-money-24x7.yml`

Then run the workflow manually. The previous SQLite cache/checkpoints are reused.

## Smart-Money rules

The scoring and alert rules are unchanged from the conservative design:

- Unknown/ambiguous swaps are excluded from scoring and alerts.
- A wallet must have enough evaluated trades and meet ROI, win-rate and profit-factor thresholds.
- FIFO is used for realized PnL.
- One lucky trade is penalized in the Smart Money score.
- Three distinct Smart Money wallets buying the same token inside the configured alert window can trigger Telegram after quality/risk checks.

## Historical RPC limitation

This remains RPC-only discovery, not an indexer. A 30-day Base backfill is still substantial. V1.4 is meant to be robust on ordinary hosted RPC while using batching to reduce wall-clock latency. At much larger scale, a dedicated indexer remains the appropriate historical provider.
