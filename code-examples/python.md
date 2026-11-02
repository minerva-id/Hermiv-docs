# Python Sniper Bot Implementation

A production-ready Python client using `asyncio` and `websockets` that connects to the Hermiv Gateway, handles auto-reconnects, subscribes to high-conviction buys, and executes orders.

---

## 1. Prerequisites

```bash
pip install websockets aiohttp
```

---

## 2. Complete Sniper Client (`sniper.py`)

```python
import asyncio
import json
import logging
import sys
import websockets

logging.basicConfig(
    level=logging.INFO,
    format="%(asctime)s [%(levelname)s] %(message)s",
    handlers=[logging.StreamHandler(sys.stdout)]
)

GATEWAY_WS_URL = "wss://api.minerva-id.xyz/v1/ws"
MIN_TRADE_USD = 500  # Only trigger on smart buys >= $500
TARGET_CHAINS = ["solana", "robinhood"]

async def execute_trade(signal: dict):
    """
    Sub-millisecond execution hook.
    Connect your Jupiter swap, Jito bundle, or private RPC here.
    """
    token_ca = signal["token"]
    symbol = signal.get("symbol", signal.get("token_info", {}).get("symbol", "UNKNOWN"))
    chain = signal["chain"]
    trader = signal.get("trader", {})
    volume = signal.get("trade", {}).get("volume_usd", 0)

    logging.info(f"🚀 [BUY TRIGGER] Chain: {chain.upper()} | Symbol: {symbol} | "
                 f"Volume: ${volume:,.2f} | Trader: @{trader.get('handle', 'whale')} | CA: {token_ca}")
    
    # Example: Send swap instruction to your local Jito / Solana engine
    # await jito_client.send_swap(mint=token_ca, amount_sol=0.5)

async def run_hermiv_consumer():
    reconnect_delay = 1
    
    while True:
        try:
            logging.info(f"Connecting to Hermiv Gateway: {GATEWAY_WS_URL} ...")
            async with websockets.connect(
                GATEWAY_WS_URL,
                ping_interval=30,
                ping_timeout=10
            ) as ws:
                logging.info("Connected to Hermiv WebSocket Gateway!")
                reconnect_delay = 1  # Reset backoff on successful connection
                
                # 1. Listen for Welcome Handshake
                handshake_raw = await ws.recv()
                handshake = json.loads(handshake_raw)
                logging.info(f"Handshake received: provider={handshake.get('provider')}, spec={handshake.get('spec')}")

                # 2. Dispatch Dynamic Topic Subscription
                subscribe_payload = {
                    "action": "subscribe",
                    "chains": TARGET_CHAINS,
                    "min_usd": MIN_TRADE_USD,
                    "sides": ["BUY"]
                }
                await ws.send(json.dumps(subscribe_payload))
                logging.info(f"Subscription dispatched: {subscribe_payload}")

                # 3. Main Stream Processing Loop
                async for raw_message in ws:
                    try:
                        msg = json.loads(raw_message)
                        event_type = msg.get("event") or msg.get("type")

                        if event_type == "signal":
                            canonical_signal = msg.get("data", {})
                            # Non-blocking async execution
                            asyncio.create_task(execute_trade(canonical_signal))
                        
                        elif event_type == "subscribed":
                            logging.info(f"✅ Subscription active! Filter: {msg.get('filter')}")

                        elif event_type == "pong":
                            pass  # Heartbeat ack

                    except json.JSONDecodeError:
                        logging.warning("Received non-JSON frame from gateway")

        except (websockets.ConnectionClosedError, websockets.ConnectionClosedOK) as e:
            logging.warning(f"Connection closed by server ({e}). Reconnecting in {reconnect_delay}s...")
        except Exception as e:
            logging.error(f"Unexpected connection error: {e}. Retrying in {reconnect_delay}s...")

        await asyncio.sleep(reconnect_delay)
        reconnect_delay = min(reconnect_delay * 2, 30)  # Exponential backoff capped at 30s

if __name__ == "__main__":
    try:
        asyncio.run(run_hermiv_consumer())
    except KeyboardInterrupt:
        logging.info("Sniper daemon stopped by user.")
```
