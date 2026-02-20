# Crypto Triangular Arbitrage Engine

[![JavaScript](https://img.shields.io/badge/JavaScript-Node.js-F7DF1E?logo=javascript)](https://nodejs.org/)
[![Trading](https://img.shields.io/badge/Trading-Algorithmic-green)](https://github.com/yuzgecoguz/crypto-triangular-arbitrage-engine)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Built in](https://img.shields.io/badge/Built%20in-2022-blue)](https://github.com/yuzgecoguz/crypto-triangular-arbitrage-engine)

> High-performance triangular arbitrage detection and execution engine for cryptocurrency exchanges. Sub-40ms execution across 3 trading pairs.

## Overview

Originally built in **2022**, this repository documents the architecture and design of a real-time triangular arbitrage engine built for cryptocurrency markets. The system monitors orderbook data via WebSocket feeds, detects profitable triangular arbitrage paths, and executes trades with ultra-low latency.

> **Note**: Core trading logic is proprietary. This repository serves as an architecture showcase and technical documentation.

## What is Triangular Arbitrage?

Triangular arbitrage exploits price discrepancies between three trading pairs on the same exchange. If the cross-rates don't perfectly align, a profit opportunity exists.

```
Example: BTC/USDT → ETH/BTC → ETH/USDT

Step 1: Buy ETH with USDT     (ETH/USDT)
Step 2: Sell ETH for BTC       (ETH/BTC)
Step 3: Sell BTC for USDT      (BTC/USDT)

If: USDT → ETH → BTC → USDT yields more USDT than started with → Profit!
```

## Performance

| Metric | Value |
|--------|-------|
| **Execution time** (3 pairs) | 30-40 ms |
| **Price feed latency** | < 5 ms (WebSocket) |
| **Path calculation** | < 1 ms |
| **Pairs monitored** | All valid triangular paths |
| **Uptime** | 24/7 autonomous operation |

## Architecture

```
┌─────────────────────────────────────────────────────┐
│                   EXCHANGE APIs                      │
│         WebSocket Feeds    +    REST APIs             │
└──────────┬──────────────────────┬────────────────────┘
           │ Real-time prices     │ Order execution
           ▼                      ▲
┌─────────────────────┐  ┌───────────────────────┐
│   Price Feed Engine  │  │  Execution Engine      │
│                      │  │                        │
│  • WebSocket client  │  │  • Signed REST API     │
│  • Orderbook cache   │  │  • HMAC authentication │
│  • Latency monitor   │  │  • Rate limit manager  │
│  • Reconnect logic   │  │  • Order validation    │
└──────────┬──────────┘  └───────────▲────────────┘
           │                         │
           ▼                         │
┌─────────────────────┐              │
│  Arbitrage Detector  │             │
│                      │             │
│  • Path enumeration  │  execute    │
│  • Profit calculator │─────────────┘
│  • Fee deduction     │
│  • Slippage estimate │
│  • Min profit filter │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│  Notification System │
│                      │
│  • Telegram alerts   │
│  • Trade logging     │
│  • P&L tracking      │
└─────────────────────┘
```

## Technical Stack

| Component | Technology |
|-----------|-----------|
| Runtime | Node.js |
| Language | JavaScript |
| WebSocket | `ws` library for real-time data |
| REST Client | `axios` with HMAC-SHA256 signing |
| Notifications | Telegram Bot API |
| Logging | Custom file-based trade logger |

## Key Components

### 1. Price Feed Engine
- Maintains persistent WebSocket connections to exchange orderbook streams
- Caches top-of-book prices for all monitored pairs
- Handles automatic reconnection with exponential backoff
- Monitors WebSocket latency for execution timing decisions

### 2. Arbitrage Detector
- Enumerates all valid triangular paths from available trading pairs
- Calculates effective rates accounting for:
  - Maker/taker fees
  - Bid-ask spread
  - Estimated slippage based on orderbook depth
- Filters opportunities by minimum profit threshold

### 3. Execution Engine
- Submits signed orders via REST API with HMAC-SHA256 authentication
- Manages API rate limits to avoid throttling
- Validates order parameters before submission
- Handles partial fills and order book changes

### 4. Notification System
- Real-time Telegram alerts for detected opportunities and executed trades
- Trade history logging with P&L tracking
- Error and system health monitoring

## Cross-Exchange Arbitrage Module

In addition to triangular arbitrage, a separate module monitors price differences between exchanges (e.g., Binance vs Paribu) and sends Telegram notifications when profitable spreads are detected.

| Feature | Detail |
|---------|--------|
| Exchanges | Binance, Paribu (extensible) |
| Data source | REST API polling |
| Alerts | Telegram Bot |
| Language | Python |

## Configuration Example

```json
{
  "exchange": "binance",
  "websocket": {
    "baseUrl": "wss://stream.binance.com:9443/ws",
    "streams": ["btcusdt@bookTicker", "ethbtc@bookTicker", "ethusdt@bookTicker"]
  },
  "execution": {
    "minProfitPercent": 0.15,
    "maxSlippagePercent": 0.05,
    "enableTrading": false
  },
  "notifications": {
    "telegram": {
      "enabled": true,
      "chatId": "YOUR_CHAT_ID"
    }
  }
}
```

## Related Projects

- [ethereum-smart-contract-security-audit](https://github.com/yuzgecoguz/ethereum-smart-contract-security-audit) — Smart contract vulnerability analysis
- [oracle-manipulation-attack-demo](https://github.com/yuzgecoguz/oracle-manipulation-attack-demo) — DeFi oracle attack demonstration

## License

MIT License - see [LICENSE](LICENSE) for details.

## Author

**Oguzhan Yuzgec** — Blockchain Security & Quant Developer

- GitHub: [@yuzgecoguz](https://github.com/yuzgecoguz)
- LinkedIn: [oguzhan-yuzgec](https://www.linkedin.com/in/oguzhan-yuzgec-a72988182/)
