# Welcome to Hermiv Developer Documentation

**Hermiv** is an institutional-grade, zero-delay multi-chain alpha signal and smart money telemetry infrastructure. 

Designed specifically for **quantitative trading firms, sniper bot engineers, and automated copy-trading systems**, Hermiv continuously ingests, filters, and broadcasts on-chain swaps executed by proven profitable smart money cohorts across **Solana, Robinhood L2, and BNB Smart Chain (BSC)**.

---

## ⚡ The Hermiv Edge

| Problem in Traditional Bot Architecture | The Hermiv Solution |
| :--- | :--- |
| **High RPC Costs ($300 - $1,000/mo):** Builders burn budget polling Solana/EVM RPCs. | **Zero RPC Overhead:** Connect to a single WebSocket (`/v1/ws`) or SSE stream (`/v1/stream`) and receive clean, pre-filtered smart money fills. |
| **Noise & Decoy Attacks:** 90% of raw on-chain events are airdrops, wash trades, or MEV decoys. | **Pre-Flight On-Chain Sanitation:** Ingestion rejects zero-balance airdrops, stranger funding, and toxic scalpers in $<0.1\text{ ms}$. |
| **Predatory Honeypots:** Sniping bots get trapped in tokens with maximum transaction limits or locked sells. | **On-Chain Bytecode Vaccine:** Automated bytecode and storage inspection blocks predatory honeypot contracts before signals reach the API. |
| **Data Disparity Across Chains:** Different nodes emit incompatible data shapes. | **Unified Canonical Schema (`alpha.v1`):** All trades across Solana, BSC, and Robinhood L2 share the exact same JSON format. |

---

## 🌐 Global Endpoint Topology

Hermiv operates behind a globally distributed Cloudflare Edge network providing automatic DDoS armor, TLS 1.3 encryption, and sub-millisecond edge routing:

* **Production WebSocket Gateway:** `wss://api.minerva-id.xyz/v1/ws`
* **Production SSE Streaming Endpoint:** `https://api.minerva-id.xyz/v1/stream`
* **REST Snapshot & Metadata API:** `https://api.minerva-id.xyz/v1`
* **Developer Portal & API Keys Vault:** [https://dash.hermiv.tech](https://dash.hermiv.tech)
* **Command Center & Visual Dashboard:** [https://hermiv.tech](https://hermiv.tech)

---

## 📊 Live Cohort Footprint

Hermiv continuously profiles and monitors:
* **1,840+ Curated Solana Smart Wallets** (Tier 1 & Top 50 profitable traders profiled from on-chain data lakes).
* **310 Verified Robinhood L2 Traders** (Exclusive low-latency Relay.link copytrade flow).
* **370+ High-Volume BSC Wallets** (Early bonding curve snipers on PancakeSwap / Four.meme).

Ready to build? Jump straight to the [Quickstart Guide](getting-started/quickstart.md) to receive your first live trade stream in under 3 minutes.
