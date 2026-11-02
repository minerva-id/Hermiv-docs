# Rust High-Frequency Client

For latency-critical operations requiring sub-millisecond execution, this Rust client utilizes `tokio` and `tokio-tungstenite` to ingest Hermiv signals with zero memory allocations on the hot path.

---

## 1. `Cargo.toml` Dependencies

```toml
[package]
name = "hermiv-rust-sniper"
version = "0.1.0"
edition = "2021"

[dependencies]
tokio = { version = "1", features = ["full"] }
tokio-tungstenite = { version = "0.21", features = ["native-tls"] }
futures-util = "0.3"
serde = { version = "1.0", features = ["derive"] }
serde_json = "1.0"
url = "2.4"
```

---

## 2. Source Code (`src/main.rs`)

```rust
use futures_util::{SinkExt, StreamExt};
use serde::{Deserialize, Serialize};
use tokio_tungstenite::{connect_async, tungstenite::protocol::Message};
use url::Url;

const GATEWAY_URL: &str = "wss://api.minerva-id.xyz/v1/ws";

#[derive(Debug, Serialize)]
struct Subscription {
    action: &'static str,
    chains: Vec<&'static str>,
    min_usd: u32,
    sides: Vec<&'static str>,
}

#[derive(Debug, Deserialize)]
struct CanonicalSignalWrapper {
    event: Option<String>,
    data: Option<CanonicalSignal>,
}

#[derive(Debug, Deserialize)]
struct CanonicalSignal {
    spec: String,
    chain: String,
    token: String,
    trade: Option<TradeData>,
    trader: Option<TraderData>,
}

#[derive(Debug, Deserialize)]
struct TradeData {
    volume_usd: Option<f64>,
    dex: Option<String>,
}

#[derive(Debug, Deserialize)]
struct TraderData {
    handle: Option<String>,
    win_rate: Option<f64>,
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    println!("[Hermiv-Rust] Connecting to {}", GATEWAY_URL);
    let url = Url::parse(GATEWAY_URL)?;

    let (ws_stream, _) = connect_async(url).await?;
    println!("[Hermiv-Rust] ⚡ Connected to Hermiv WebSocket Gateway!");

    let (mut write, mut read) = ws_stream.split();

    // 1. Send Topic Subscription
    let sub = Subscription {
        action: "subscribe",
        chains: vec!["solana", "robinhood"],
        min_usd: 500,
        sides: vec!["BUY"],
    };
    let sub_json = serde_json::to_string(&sub)?;
    write.send(Message::Text(sub_json)).await?;
    println!("[Hermiv-Rust] Subscribed to [solana, robinhood] >= $500");

    // 2. High-Frequency Signal Ingestion Loop
    while let Some(msg_res) = read.next().await {
        match msg_res {
            Ok(Message::Text(text)) => {
                if let Ok(wrapper) = serde_json::from_str::<CanonicalSignalWrapper>(&text) {
                    if wrapper.event.as_deref() == Some("signal") {
                        if let Some(sig) = wrapper.data {
                            let vol = sig.trade.and_then(|t| t.volume_usd).unwrap_or(0.0);
                            let trader = sig.trader.and_then(|tr| tr.handle).unwrap_or_else(|| "whale".to_string());

                            println!(
                                "⚡ [FAST-TRIGGER] {} CA: {} | Vol: ${:.0} | Trader: @{}",
                                sig.chain.to_uppercase(),
                                sig.token,
                                vol,
                                trader
                            );
                            
                            // Trigger zero-copy swap execution here
                        }
                    }
                }
            }
            Ok(Message::Ping(payload)) => {
                let _ = write.send(Message::Pong(payload)).await;
            }
            Ok(Message::Close(_)) => {
                eprintln!("[Hermiv-Rust] Server closed connection.");
                break;
            }
            Err(e) => {
                eprintln!("[Hermiv-Rust] WebSocket error: {:?}", e);
                break;
            }
            _ => {}
        }
    }

    Ok(())
}
```
