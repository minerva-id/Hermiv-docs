# WebSocket Gateway (`/v1/ws`)

The WebSocket Gateway is the primary low-latency entrypoint for automated trading bots and sniper systems. It provides sub-millisecond push notifications of verified smart money fills across all supported blockchains.

* **Endpoint:** `wss://api.minerva-id.xyz/v1/ws`
* **Protocol:** WSS (TLS encrypted WebSocket)
* **Average Delivery Lag:** $< 1\text{ ms}$

---

## 1. Connection & Handshake

Upon establishing a TCP/TLS connection, the gateway immediately responds with a `hello` handshake frame describing server capabilities and default subscription state:

```json
{
  "type": "hello",
  "spec": "alpha.v1",
  "provider": "Hermiv Engine",
  "dashboard": "https://hermiv.tech",
  "supported_chains": ["solana", "robinhood", "bsc"],
  "active_filter": {
    "chains": ["solana", "robinhood", "bsc"],
    "min_usd": 0,
    "sides": ["BUY", "SELL"]
  },
  "server_ts": 1793627500000
}
```

---

## 2. Dynamic Topic Subscription

Clients can update their trade filters at any time without closing or reconnecting the socket. Send a JSON message with `action: "subscribe"`:

```json
{
  "action": "subscribe",
  "chains": ["solana", "robinhood"],
  "min_usd": 500,
  "sides": ["BUY"]
}
```

### Parameters:
* `chains` *(Array of Strings, optional)*: Blockchains to listen to. Supported values: `"solana"`, `"robinhood"`, `"bsc"`. Default: all chains.
* `min_usd` *(Number, optional)*: Minimum trade volume in USD. Any swap below this threshold is dropped before wire transmission. Default: `0`.
* `sides` *(Array of Strings, optional)*: Order sides to stream. Options: `["BUY"]`, `["SELL"]`, or `["BUY", "SELL"]`. Default: both.

### Gateway Confirmation:
```json
{
  "type": "subscribed",
  "status": "ok",
  "filter": {
    "chains": ["solana", "robinhood"],
    "min_usd": 500,
    "sides": ["BUY"]
  }
}
```

---

## 3. Real-Time Signal Frame (`alpha.v1`)

Whenever a qualifying swap executes, the gateway pushes an event frame containing the canonical signal:

```json
{
  "event": "signal",
  "data": {
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
}
```

---

## 4. Heartbeat (Ping/Pong)

To prevent idle TCP timeouts across intermediate routers and proxies, clients can send periodic pings:

### Client Send:
```text
p
```
or JSON:
```json
{
  "action": "ping"
}
```

### Gateway Response:
```json
{
  "type": "pong",
  "ts": 1793627500123
}
```

> 💡 **Recommendation:** Transmit a heartbeat every 30 to 45 seconds to keep your socket warm.
