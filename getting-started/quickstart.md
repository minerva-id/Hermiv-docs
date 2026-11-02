# Quickstart in 3 Minutes

Get up and running with live smart money alpha streams in three minutes or less.

---

## 1. Quick Test with cURL (Server-Sent Events)

You don't need to install any SDK or client library to test Hermiv. Open your terminal and run:

```bash
# Stream all verified smart money buys on Solana with trade size >= $500
curl -N "https://api.minerva-id.xyz/v1/stream?chain=solana&min_usd=500&side=buy"
```

You will immediately receive an initial handshake followed by real-time trade signals as they execute on-chain:

```text
event: connected
data: {"status":"connected","provider":"Hermiv Engine","filter":{"chainFilter":"solana","minUsd":500,"side":"buy"},"timestamp":1793627500000}

event: signal
data: {"spec":"alpha.v1","id":"solana_5wvncrXyNw...","chain":"solana","event":"SWAP","type":"BUY","token":"EKpQGS...zcjm","token_info":{"address":"EKpQGS...zcjm","symbol":"WIF","name":"dogwifhat"},"trade":{"price_usd":0.2334,"amount":15000,"volume_usd":3500,"tx_hash":"5wvncrXyNw...","dex":"Pump.fun"},"trader":{"address":"mDR91ufq...","handle":"smol_intern","win_rate":0.551},"timestamps":{"block_time":1789040000,"ingest_time":1789040000120,"lag_ms":120},"provider":"Hermiv Engine"}
```

---

## 2. Connect via WebSocket in Python

For high-speed bot execution, use the WebSocket Gateway:

### Install Dependency
```bash
pip install websockets
```

### Run Client (`sniper_listener.py`)
```python
import asyncio
import json
import websockets

GATEWAY_URL = "wss://api.minerva-id.xyz/v1/ws"

async def listen_alpha():
    async with websockets.connect(GATEWAY_URL) as ws:
        print("Connected to Hermiv WebSocket Gateway!")
        
        # 1. Receive Initial Welcome Handshake
        handshake = await ws.recv()
        print("Engine Handshake:", handshake)
        
        # 2. Subscribe to desired chains & size filters
        subscription = {
            "action": "subscribe",
            "chains": ["solana", "robinhood"],
            "min_usd": 250,
            "sides": ["BUY"]
        }
        await ws.send(json.dumps(subscription))
        print("Subscription dispatched!")

        # 3. Process Live Canonical Signals
        async for message in ws:
            payload = json.loads(message)
            if payload.get("event") == "signal":
                sig = payload["data"]
                print(f"⚡ [SIGNAL] {sig['chain'].upper()} ${sig['symbol']} | "
                      f"Vol: ${sig['trade']['volume_usd']:,.0f} | "
                      f"Trader: @{sig['trader']['handle']} (WR: {sig['trader']['win_rate']*100:.0f}%) | "
                      f"CA: {sig['token']}")

if __name__ == "__main__":
    asyncio.run(listen_alpha())
```

---

## 3. Query Recent Historical Signals via REST

If your trading bot is cold-starting and needs recent signals to populate its initial state cache, call the REST snapshot endpoint:

```bash
curl -s "https://api.minerva-id.xyz/v1/signals?chain=solana&limit=10&min_usd=500"
```

Response:
```json
{
  "spec": "alpha.v1",
  "status": "ok",
  "provider": "Hermiv Engine",
  "count": 10,
  "signals": [
    {
      "spec": "alpha.v1",
      "id": "solana_2wdJDuFk3pxSheyh",
      "chain": "solana",
      "event": "SWAP",
      "type": "BUY",
      "token": "DRAMjSWR7HRfJKjRkvQWYL2bcaejaVhuxEcjf4pAY4Cw",
      "trade": {
        "volume_usd": 1250.0,
        "dex": "Pump.fun"
      },
      "trader": {
        "handle": "gfree",
        "win_rate": 0.65
      }
    }
  ]
}
```

Next, obtain your API Key from the [Hermiv Developer Portal (dash.hermiv.tech)](https://dash.hermiv.tech) and learn how to configure [Authentication & Rate Limits](../authentication/api-keys.md) for production bot deployments.
