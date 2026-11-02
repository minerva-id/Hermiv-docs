# Server-Sent Events (`/v1/stream`)

For environments that prefer native HTTP streaming over WebSockets (such as serverless edge workers, cURL debugging, and browser applications), Hermiv provides a high-throughput **Server-Sent Events (SSE)** endpoint.

* **Endpoint:** `https://api.minerva-id.xyz/v1/stream`
* **Transport:** HTTP/1.1 or HTTP/2 chunked transfer (`Content-Type: text/event-stream`)

---

## 1. Query Parameters

Filters are supplied directly in the URL query string:

| Parameter | Type | Required | Description | Example |
| :--- | :--- | :--- | :--- | :--- |
| `chains` or `chain` | string | No | Comma-separated list of blockchains to stream. | `solana`, `solana,robinhood` |
| `min_usd` | number | No | Minimum volume threshold in USD. | `500` |
| `side` | string | No | Order side filter (`buy` or `sell`). | `buy` |
| `api_key` | string | Optional* | Your Hermiv API key. | `hmv_live_...` |

---

## 2. Event Stream Structure

### Event: `connected`
Sent immediately upon connection establishment:
```text
event: connected
data: {"status":"connected","provider":"Hermiv Engine","filter":{"chainFilter":"solana","minUsd":500,"side":"buy"},"timestamp":1793627500000}
```

### Event: `signal`
Sent whenever a smart money swap passes your filter criteria:
```text
event: signal
data: {"spec":"alpha.v1","id":"solana_5wvncrXyNw...","chain":"solana","event":"SWAP","type":"BUY","token":"EKpQGS...zcjm","token_info":{"address":"EKpQGS...zcjm","symbol":"WIF","name":"dogwifhat"},"trade":{"price_usd":0.2334,"amount":15000,"volume_usd":3500,"tx_hash":"5wvncr...","dex":"Pump.fun"},"trader":{"address":"mDR91u...","handle":"smol_intern","win_rate":0.551},"timestamps":{"block_time":1789040000,"ingest_time":1789040000120,"lag_ms":120},"provider":"Hermiv Engine"}
```

### Event: `ping`
Heartbeat sent by the server every 15 seconds to keep the HTTP connection alive:
```text
event: ping
data: {"ts":1793627515000}
```

---

## 3. Usage Examples

### cURL
```bash
curl -N "https://api.minerva-id.xyz/v1/stream?chain=solana&min_usd=1000&side=buy"
```

### Browser JavaScript (`EventSource`)
```javascript
const eventSource = new EventSource('https://api.minerva-id.xyz/v1/stream?chain=solana&min_usd=250&api_key=YOUR_KEY');

eventSource.addEventListener('signal', (event) => {
  const signal = JSON.parse(event.data);
  console.log(`New Alpha: ${signal.token_info.symbol} | Volume: $${signal.trade.volume_usd}`);
});

eventSource.onerror = (err) => {
  console.error('Stream encountered error, reconnecting...', err);
};
```
