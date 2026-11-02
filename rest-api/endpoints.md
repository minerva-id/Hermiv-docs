# REST Snapshot Endpoints

Hermiv provides high-speed analytical REST endpoints to query historical signal tapes, monitor engine telemetries, and inspect top-performing smart wallets.

Base URL: `https://api.minerva-id.xyz/v1`

---

## 1. Get Recent Signals Snapshot

Retrieve the most recent canonical signals recorded across all monitored blockchains.

```http
GET /v1/signals
```

### Query Parameters:
* `chain` *(string, optional)*: Filter by chain (`solana`, `robinhood`, `bsc`).
* `min_usd` *(number, optional)*: Filter by minimum USD volume.
* `limit` *(integer, optional)*: Maximum items to return (default: `50`, max: `100`).

### Request:
```bash
curl -s "https://api.minerva-id.xyz/v1/signals?chain=solana&limit=5&min_usd=500"
```

### Response:
```json
{
  "spec": "alpha.v1",
  "status": "ok",
  "provider": "Hermiv Engine",
  "count": 5,
  "signals": [
    {
      "spec": "alpha.v1",
      "id": "solana_2wdJDuFk3pxSheyh",
      "chain": "solana",
      "event": "SWAP",
      "type": "BUY",
      "token": "HckPgvvNVNnFRZsYRNGMbeSs362dLLhZGjzNmqpgf9gk",
      "token_info": {
        "address": "HckPgvvNVNnFRZsYRNGMbeSs362dLLhZGjzNmqpgf9gk",
        "symbol": "TOKEN",
        "name": "TOKEN"
      },
      "trade": {
        "price_usd": 0.0042,
        "amount": 100000.0,
        "volume_usd": 420.0,
        "dex": "Raydium"
      },
      "trader": {
        "handle": "gfree",
        "win_rate": 0.65
      },
      "timestamps": {
        "block_time": 1793627447,
        "ingest_time": 1793627447770,
        "lag_ms": 770
      },
      "provider": "Hermiv Engine"
    }
  ],
  "timestamp": "2026-11-02T14:15:00.000Z"
}
```

---

## 2. Get Gateway Telemetry & Stats

Check server uptime, live connected client counts, total signals emitted, and system status.

```http
GET /v1/stats
```

### Request:
```bash
curl -s "https://api.minerva-id.xyz/v1/stats"
```

### Response:
```json
{
  "status": "ok",
  "provider": "Hermiv Engine",
  "uptime_sec": 8420,
  "active_clients": {
    "v1_websocket": 4,
    "legacy_websocket": 1,
    "sse_streams": 6
  },
  "total_signals_emitted": 1420,
  "supported_chains": ["solana", "robinhood", "bsc"],
  "timestamp": "2026-11-02T14:15:00.000Z"
}
```

---

## 3. Get Top Smart Money Wallets

Inspect the top curated smart money wallets monitored by the ingestion engine, including tracked win rates and 7-day PnL.

```http
GET /v1/wallets/top
```

### Query Parameters:
* `limit` *(integer, optional)*: Number of wallets to return (default: `50`, max: `100`).
* `active` *(boolean, optional)*: Set to `true` to only return wallets active in the last 24h.

### Request:
```bash
curl -s "https://api.minerva-id.xyz/v1/wallets/top?limit=3"
```

### Response:
```json
{
  "status": "ok",
  "provider": "Hermiv Engine",
  "chain": "solana",
  "count": 3,
  "wallets": [
    {
      "address": "B1ZMARC6qhJYJrvBJywUYdtf6hq2WBkTgvbbnGuQsQnX",
      "handle": "alpha_scout",
      "win_rate": 0.772,
      "pnl_7d_usd": 182400.0,
      "tier": "elite",
      "trades_count": 34
    }
  ]
}
```
