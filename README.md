# RAMO Smart Money Base v1.5

Base Mainnet Smart-Money discovery and real-time monitoring for GitHub Actions.

## Why v1.5 exists

The v1.4 batch transport itself worked correctly, but the restored SQLite state contained poisoned per-DEX checkpoints left by the failed v1.3 `eth_getLogs` path. That made v1.4 start close to the chain tip, scan only 489 blocks, and then analyze an incomplete historical database.

V1.5 fixes the checkpoint layer, not the trade database:

- Performs a one-time rollback of only the corrupted checkpoint state to the last confirmed safe floors.
- Keeps all previously collected trades; duplicate replay is safe because trades use idempotent inserts.
- Uses the reliable v1.4 **batched full-block** historical provider; no `eth_getLogs` dependency.
- Marks historical discovery complete only if the final fully processed batch actually reaches the target block.
- An exception, RPC failure, GitHub runtime stop, or generator early-return can no longer mark discovery complete from a `finally` block.
- Per-DEX checkpoints are advanced only after a completely processed batch.

## One-time safe checkpoint repair

On the first v1.5 run you should see a warning similar to:

```text
V1.5 checkpoint integrity repair applied | ...
V1.5 historical catch-up re-opened from safe common floor 49708292; classic Aerodrome remains eligible from 49812754
```

The common/new-router floor is `49,708,291`, the block immediately before the confirmed v1.3 start. Classic Aerodrome retains its separately confirmed safe floor `49,812,753`.

Existing recent trades from v1.4 are not deleted. When backfill later reaches those blocks, `INSERT OR IGNORE` prevents duplicate rows.

## Expected startup

```text
RAMO Smart Money Base v1.5.0 starting
Connected to Base (chain_id=8453)
Historical provider: BATCHED FULL-BLOCK discovery (no eth_getLogs)
V1.5 checkpoint integrity repair applied | ...
Scanning Base historical blocks 49708292 -> ...
BATCH BLOCK RANGE ...
BATCH BLOCK DONE ...
```

Historical completion is now logged only as:

```text
Historical discovery VERIFIED complete through block ...
```

If a runner stops before that point, it prints:

```text
Historical discovery NOT complete; last fully completed batch=... target=.... Checkpoints preserved for resume
```

## GitHub upgrade

Keep the same Repository Secrets. Replace only:

- `smart_money_payload.enc`
- `README.md`
- `.github/workflows/smart-money-24x7.yml`

The current SQLite cache is intentionally reused.

## Smart Money thresholds

Default conservative thresholds remain:

- minimum evaluated trades: 30
- minimum ROI: 20%
- minimum win rate: 55%
- minimum profit factor: 1.5

A high ROI on only 5–8 trades is therefore not enough to qualify a wallet as Smart Money. The database must finish historical catch-up before the Smart Money population is meaningful.
