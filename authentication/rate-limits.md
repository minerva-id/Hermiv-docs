# Rate Limits & Concurrency

To protect infrastructure availability and ensure zero-delay performance for all active trading bots, Hermiv enforces automated concurrency and rate-limiting policies at the edge.

---

## 1. Concurrency Limits (Streaming Sockets)

Concurrency limits apply to simultaneous open connections across **WebSocket (`/v1/ws`)** and **Server-Sent Events (`/v1/stream`)**:

* **Trial Key:** 1 active socket.
* **Pro Tier:** 2 active sockets (e.g. 1 primary bot + 1 hot-standby).
* **Quant / VIP Tier:** 5 active sockets (e.g. multi-region distributed bots).

### What happens when you exceed concurrency limits?
If your client attempts to open an additional socket beyond your tier allowance:
* **HTTP / SSE:** Returns `HTTP 429 Too Many Requests`.
* **WebSocket:** Handshake is rejected with WebSocket close code `4429 (Concurrency Limit Exceeded)`.

```json
{
  "error": "Concurrency Limit Exceeded",
  "message": "Your tier allows a maximum of 2 active socket connections.",
  "active_sockets": 2,
  "allowed_sockets": 2
}
```

---

## 2. REST Rate Limits

REST endpoints (`/v1/signals`, `/v1/stats`, `/v1/wallets/top`) enforce sliding-window rate limits:

| Tier | Rate Limit | Reset Window |
| :--- | :--- | :--- |
| **Trial** | 10 requests | 60 seconds |
| **Pro** | 60 requests | 60 seconds |
| **Quant / VIP** | 300 requests | 60 seconds |

Every REST response includes standard rate-limit headers:
```http
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 58
X-RateLimit-Reset: 1793627560
```

When you exceed the limit:
```http
HTTP/1.1 429 Too Many Requests
Retry-After: 15
Content-Type: application/json

{
  "error": "Rate limit exceeded. Please wait 15 seconds before retrying.",
  "status": 429
}
```

---

## 3. Best Practices for Bot Engineers

1. **Prefer Persistent WebSockets over REST Polling:** Do not poll `/v1/signals` in an infinite loop. Use `/v1/ws` or `/v1/stream` for zero-latency, event-driven trade notifications.
2. **Handle Reconnection with Exponential Backoff:** When a connection drops due to network blips, reconnect after $1\text{s}, 2\text{s}, 4\text{s}, 8\text{s}$ up to a maximum backoff to avoid tripping IP rate limits.
3. **Reuse Single Socket for Multi-Chains:** Subscribe to multiple chains (`solana`, `robinhood`, `bsc`) over a single WebSocket connection rather than opening a separate socket per chain.
