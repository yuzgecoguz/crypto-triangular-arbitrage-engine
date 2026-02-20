# Architecture Deep Dive

## System Overview

The triangular arbitrage engine operates as a high-performance, event-driven system that continuously monitors cryptocurrency markets for arbitrage opportunities and executes trades with minimal latency.

## Core Design Principles

1. **Latency minimization**: Every component is optimized for speed. WebSocket feeds provide real-time prices, and the execution path is kept as short as possible.

2. **Reliability**: Automatic reconnection, error recovery, and health monitoring ensure 24/7 operation.

3. **Safety**: Multiple validation layers prevent bad trades — profit threshold checks, slippage estimation, and order book depth verification.

## Data Flow

```
Exchange WebSocket → Price Cache → Path Calculator → Profit Filter → Executor → Exchange REST API
                                                                         │
                                                                         ▼
                                                                  Telegram Alert
```

## Price Feed Architecture

The price feed engine maintains a single persistent WebSocket connection that subscribes to multiple streams (one per trading pair). When a price update arrives:

1. Parse the message (< 0.1ms)
2. Update the in-memory price cache
3. Trigger arbitrage calculation for all paths involving this pair
4. If profitable path found → execute immediately

## Profit Calculation

For a triangular path A → B → C → A:

```
effectiveRate = (bidB/askA) * (bidC/askB) * (bidA/askC)
grossProfit = effectiveRate - 1.0
netProfit = grossProfit - (3 * tradingFee) - estimatedSlippage
```

Where:
- `bidX` and `askX` are the best bid/ask from the orderbook
- `tradingFee` is the maker/taker fee (typically 0.1%)
- `estimatedSlippage` is calculated from orderbook depth

## Execution Strategy

Orders are submitted as **market orders** for speed. The engine:
1. Checks available balance
2. Validates the opportunity is still profitable (prices may have changed)
3. Submits all 3 orders sequentially with HMAC-signed requests
4. Logs the result and sends notification

## Risk Management

- **Minimum profit threshold**: Only executes if net profit exceeds configurable minimum
- **Maximum position size**: Limits exposure per trade
- **Cooldown period**: Prevents rapid-fire execution on stale signals
- **Balance verification**: Confirms sufficient funds before execution
