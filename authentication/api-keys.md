# API Key Authorization

Hermiv uses cryptographic API keys to authenticate and authorize requests across our streaming and REST gateways.

---

## 1. Obtaining Your API Key

API keys are managed through the **Hermiv Command Center**:
1. Log in to [https://hermiv.tech](https://hermiv.tech).
2. Navigate to **Developer Settings ➔ API Keys**.
3. Generate a new key for your bot or execution daemon.

Keys follow a standardized prefix format:
* Production Keys: `hmv_live_<32-hex-characters>`
* Test / Sandbox Keys: `hmv_test_<32-hex-characters>`

> ⚠️ **Security Warning:** Never commit your API key to public version control or expose it in client-side browser code. Always inject it via environment variables (`HERMIV_API_KEY`).

---

## 2. Authentication Methods

Hermiv supports three flexible authorization methods depending on your transport protocol:

### Method A: HTTP Request Header (Recommended for REST)
Send your key in the `X-Hermiv-Key` or standard `Authorization` header:

```bash
# Using X-Hermiv-Key
curl -H "X-Hermiv-Key: hmv_live_9a7d8e6c4b2a1f0e8d7c6b5a4f3e2d1c" \
     "https://api.minerva-id.xyz/v1/signals"

# Using Authorization Bearer
curl -H "Authorization: Bearer hmv_live_9a7d8e6c4b2a1f0e8d7c6b5a4f3e2d1c" \
     "https://api.minerva-id.xyz/v1/signals"
```

### Method B: URL Query Parameter (Recommended for SSE & Web Browsers)
Because native browser `EventSource` cannot send custom HTTP headers, you can append your key via the `api_key` query parameter:

```bash
curl -N "https://api.minerva-id.xyz/v1/stream?api_key=hmv_live_9a7d8e6c4b2a1f0e8d7c6b5a4f3e2d1c"
```

### Method C: WebSocket Handshake Message
When connecting to `wss://api.minerva-id.xyz/v1/ws`, you can authenticate immediately after connection:

```json
{
  "action": "auth",
  "key": "hmv_live_9a7d8e6c4b2a1f0e8d7c6b5a4f3e2d1c"
}
```

The gateway confirms your authentication status:
```json
{
  "type": "authenticated",
  "status": "ok",
  "tier": "pro",
  "max_connections": 2
}
```

---

## 3. Account Tiers & Permissions

| Feature | Trial / Sandbox | Pro Tier ($99/mo) | Institutional / Quant ($299/mo) |
| :--- | :--- | :--- | :--- |
| **Ingest Latency** | 30s Artificial Delay | **Zero-Delay (< 0.1 ms)** | **Zero-Delay (< 0.1 ms)** |
| **Max Concurrent Sockets** | 1 connection | 2 connections | 5 connections |
| **REST Rate Limit** | 10 requests / min | 60 requests / min | 300 requests / min |
| **Chains Monitored** | Solana | Solana, Robinhood L2, BSC | Solana, Robinhood L2, BSC |
| **Anti-Honeypot Gate** | Basic | Full Bytecode Vaccine | Full Bytecode Vaccine + Priority Ingress |
