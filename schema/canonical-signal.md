# Canonical Signal Schema (`alpha.v1`)

To eliminate parsing friction across different blockchains, Hermiv transforms every raw on-chain transaction into the **`alpha.v1` Canonical Specification**.

Regardless of whether a swap happened on Solana (Pump.fun/Raydium), Robinhood L2 (Relay.link), or BSC (PancakeSwap/Four.meme), your bot always receives this exact JSON structure:

---

## 1. Schema Breakdown

```json
{
  "spec": "alpha.v1",
  "id": "solana_5wvncrXyNwH5d9yZcM2B4yZk8t5K9q8n7w3",
  "chain": "solana",
  "event": "SWAP",
  "type": "BUY",
  "token": "EKpQGSJtjMFqKZ9KQanSqYXRcF8fBopzLHYxdM65zcjm",
  "token_info": {
    "address": "EKpQGSJtjMFqKZ9KQanSqYXRcF8fBopzLHYxdM65zcjm",
    "symbol": "WIF",
    "name": "dogwifhat"
  },
  "trade": {
    "price_usd": 0.2334,
    "amount": 15000.0,
    "volume_usd": 3501.0,
    "tx_hash": "5wvncrXyNwH5d9yZcM2B4yZk8t5K9q8n7w3...",
    "dex": "Pump.fun",
    "liquidity_usd": 85000.0,
    "mcap_usd": 233400.0
  },
  "trader": {
    "address": "mDR91ufq4S2uSHa9E2PYL7xg4LwcQXaQFP7YK864a1z",
    "handle": "smol_intern",
    "display_name": "Smol Intern",
    "followers": 12450,
    "win_rate": 0.68,
    "pnl_7d_usd": 45200.0,
    "total_pnl_usd": 128400.0,
    "tier": "alpha"
  },
  "timestamps": {
    "block_time": 1789040000,
    "ingest_time": 1789040000120,
    "lag_ms": 120
  },
  "provider": "Hermiv Engine"
}
```

---

## 2. Field Definitions

### Root Level
* `spec` *(string)*: Always `"alpha.v1"`.
* `id` *(string)*: Unique deduplication ID (`<chain>_<tx_hash>`).
* `chain` *(string)*: Monitored blockchain (`"solana"`, `"robinhood"`, or `"bsc"`).
* `event` *(string)*: Trade event type (default: `"SWAP"`).
* `type` *(string)*: Order direction: `"BUY"` or `"SELL"`.
* `token` *(string)*: Fast-path string accessor for the contract address / mint.
* `provider` *(string)*: Always `"Hermiv Engine"`.

### `token_info` Object
* `address` *(string)*: Primary contract address or token mint.
* `symbol` *(string)*: Ticker symbol (e.g. `"WIF"`).
* `name` *(string)*: Human-readable token name.

### `trade` Object
* `price_usd` *(number)*: Execution price in USD at the moment of swap.
* `amount` *(number)*: Number of token units purchased or sold.
* `volume_usd` *(number)*: Total fiat transaction value in USD (`price_usd * amount`).
* `tx_hash` *(string)*: On-chain transaction hash or signature.
* `dex` *(string)*: Liquidity venue where execution completed (e.g. `"Pump.fun"`, `"Raydium"`, `"Meteora DLMM"`, `"PancakeSwap"`).
* `liquidity_usd` *(number)*: Estimated pool liquidity in USD (0 if unbonded).
* `mcap_usd` *(number)*: Estimated market cap in USD at fill time.

### `trader` Object
* `address` *(string)*: Public wallet address of the smart money trader.
* `handle` *(string)*: Profiler handle or alias (e.g. `"smol_intern"`).
* `win_rate` *(number | null)*: Historical win rate (0.0 to 1.0) based on on-chain closed trade analysis.
* `pnl_7d_usd` *(number | null)*: Realized and unrealized net profit in USD over the last 7 days.
* `tier` *(string)*: Profiler tier rating (`"alpha"`, `"elite"`, `"whale"`).

### `timestamps` Object
* `block_time` *(integer)*: Unix epoch timestamp in seconds when the block was confirmed.
* `ingest_time` *(integer)*: Unix epoch timestamp in milliseconds when Hermiv processed the trade.
* `lag_ms` *(integer)*: Difference in milliseconds between block confirmation and ingest (`ingest_time - (block_time * 1000)`). Used to benchmark sub-second execution speed.
