# Node.js Fast Consumer Implementation

A production-grade Node.js / TypeScript WebSocket consumer using `ws` with built-in automatic reconnects, ping/pong heartbeats, and canonical trade dispatching.

---

## 1. Prerequisites

```bash
npm install ws
```

---

## 2. Complete Consumer Client (`consumer.js`)

```javascript
import WebSocket from 'ws';

const GATEWAY_URL = 'wss://api.minerva-id.xyz/v1/ws';
const MIN_USD = 250;
const CHAINS = ['solana', 'robinhood'];

class HermivConsumer {
  constructor(url = GATEWAY_URL) {
    this.url = url;
    this.ws = null;
    this.reconnectTimeout = 1000;
    this.pingInterval = null;
  }

  connect() {
    console.log(`[Hermiv] Connecting to ${this.url}...`);
    this.ws = new WebSocket(this.url);

    this.ws.on('open', () => this.handleOpen());
    this.ws.on('message', (data) => this.handleMessage(data));
    this.ws.on('close', (code, reason) => this.handleClose(code, reason));
    this.ws.on('error', (err) => this.handleError(err));
  }

  handleOpen() {
    console.log('[Hermiv] ⚡ Connected to Hermiv WebSocket Gateway!');
    this.reconnectTimeout = 1000;

    // Send heartbeat ping every 30 seconds
    this.pingInterval = setInterval(() => {
      if (this.ws && this.ws.readyState === WebSocket.OPEN) {
        this.ws.send(JSON.stringify({ action: 'ping' }));
      }
    }, 30000);
  }

  handleMessage(rawData) {
    try {
      const msg = JSON.parse(rawData.toString('utf8'));

      // 1. Initial Handshake -> Send Subscription
      if (msg.type === 'hello') {
        console.log(`[Hermiv] Handshake received. Provider: ${msg.provider} (Spec: ${msg.spec})`);
        
        const subPayload = {
          action: 'subscribe',
          chains: CHAINS,
          min_usd: MIN_USD,
          sides: ['BUY'],
        };
        this.ws.send(JSON.stringify(subPayload));
        console.log(`[Hermiv] Subscribed to [${CHAINS.join(', ')}] >= $${MIN_USD}`);
        return;
      }

      // 2. Subscription Ack
      if (msg.type === 'subscribed') {
        console.log('[Hermiv] ✅ Active filter confirmed:', msg.filter);
        return;
      }

      // 3. Live Canonical Signal (alpha.v1)
      if (msg.event === 'signal') {
        this.onTradeSignal(msg.data);
      }
    } catch (err) {
      console.error('[Hermiv] Error parsing message:', err.message);
    }
  }

  onTradeSignal(signal) {
    const symbol = signal.symbol || signal.token_info?.symbol || 'UNKNOWN';
    const volume = signal.trade?.volume_usd || 0;
    const trader = signal.trader?.handle || 'whale';
    const winRate = signal.trader?.win_rate != null ? `${(signal.trader.win_rate * 100).toFixed(0)}%` : 'N/A';

    console.log(`⚡ [SIGNAL] ${signal.chain.toUpperCase()} $${symbol} | Vol: $${volume.toFixed(0)} | Trader: @${trader} (WR: ${winRate})`);
    console.log(`   Mint / CA: ${signal.token}`);

    // Trigger your execution logic here:
    // executeFastSwap(signal.chain, signal.token, 0.5);
  }

  handleClose(code, reason) {
    clearInterval(this.pingInterval);
    console.warn(`[Hermiv] Connection closed (${code}). Reconnecting in ${this.reconnectTimeout / 1000}s...`);
    setTimeout(() => this.connect(), this.reconnectTimeout);
    this.reconnectTimeout = Math.min(this.reconnectTimeout * 2, 30000);
  }

  handleError(err) {
    console.error('[Hermiv] WebSocket error:', err.message);
  }
}

// Start consumer
const client = new HermivConsumer();
client.connect();
```
