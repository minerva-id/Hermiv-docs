# Architecture Overview

Hermiv is architected around a strict **Decoupled Dual-Pipeline** pattern that completely isolates ultra-low-latency bot execution (**Fast-Path**) from heavy telemetry enrichment and human notification (**Slow-Path**).

```
                             [ RAW ON-CHAIN INGEST ]
                   Phantom SSE (Solana) • WS Trenches (RH / BSC)
                                       │
                                       ▼
                       ┌───────────────────────────────┐
                       │    PRE-FLIGHT VALIDATION      │
                       │ • On-Chain Bytecode Vaccine   │
                       │ • Trader Blacklist (O(1))     │
                       │ • Min USD Threshold Filter    │
                       └───────────────┬───────────────┘
                                       │
             ┌─────────────────────────┴─────────────────────────┐
             ▼ (< 0.1 ms Latency Budget)                         ▼ (Detached Background Async)
   ⚡ FAST-PATH ENGINE                                 🐢 SLOW-PATH WORKER
   • Canonical Transformer (`alpha.v1`)                • Token-Deduplicated Cache (30s window)
   • Unified WebSocket Gateway (`/v1/ws`)              • DEXScreener Pair & Liquidity Fetch
   • Multiplexed SSE Streams (`/v1/stream`)            • GMGN KOL & Smart Holder Intelligence
   • In-Memory Event Bus (`SignalHub`)                 • Rate-Limited Discord Queue Serializer
   • Sub-millisecond Bot Dispatches                    • Multi-Channel Alert Formatting
```

---

## ⚡ 1. The Fast-Path (< 0.1 ms)

The Fast-Path is dedicated entirely to **algorithmic trading bots and execution daemons**:

* **Zero External HTTP Network Delays:** The engine never awaits external third-party HTTP endpoints (DEXScreener, GMGN, CoinGecko) during the fast-path loop. All processing runs purely in memory.
* **Deterministic Execution:** Once a swap is validated against our in-memory honeypot bytecode cache and trader blacklist, it is instantly converted to the standard `alpha.v1` schema.
* **Non-Blocking Fanout:** Sockets receive the raw buffer write immediately. If a consumer socket drops or encounters network backpressure, it is purged without blocking sibling connections.

---

## 🐢 2. The Slow-Path (Enrichment & Forensics)

The Slow-Path handles heavy off-chain analytical intelligence for human traders and monitoring dashboards:

* **Token-Deduplicated Worker:** When an aggressive accumulation wave occurs (e.g. 5 smart wallets dogpiling a newly bonded coin within 30 seconds), the engine issues only **1 query** to DEXScreener and GMGN, reusing the metadata across all alerts to prevent event-loop starvation and HTTP 429 rate limits.
* **Background Isolation:** Any failure in the slow-path (e.g. Discord API outage or network timeout) has zero impact on live WebSocket and SSE consumers.

---

## 🔒 3. Enterprise Hardening & Isolation

* **Edge Armor:** Global DDoS protection and TLS 1.3 encryption terminating at the Cloudflare edge.
* **Memory Bounds:** All in-memory caches enforce strict LRU eviction ceilings (`MAX_CACHE_SIZE = 1,000`) to guarantee stability over weeks and months of continuous operation.
* **No Leaky Abstractions:** Upstream vendor endpoints and private data sources are completely masked behind the **Hermiv Engine** identity.
