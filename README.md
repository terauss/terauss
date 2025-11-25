# 🤖 Nadfun Trading Bots

<div align="center">

**Professional Trading Bots for Nad.fun Platform on Monad Blockchain**

[![TypeScript](https://img.shields.io/badge/TypeScript-5.3+-blue.svg)](https://www.typescriptlang.org/)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Node](https://img.shields.io/badge/node-%3E%3D18.0.0-brightgreen.svg)](https://nodejs.org/)

*Automated trading bots built with the Nad.fun SDK for instant token sniping and batch operations*

[Features](#-features) • [Installation](#-installation) • [Quick Start](#-quick-start) • [Documentation](#-documentation)

</div>

---

## 🎯 Overview

I've built two powerful, production-ready trading bots for the **Nad.fun** platform (Monad's version of Pump.fun). These bots enable automated trading strategies with instant execution, smart filtering, and comprehensive error handling.

### What I Built

- **🎯 Sniper Bot**: Automatically detects and buys new tokens the moment they're created on the bonding curve
- **📦 Bundler Bot**: Executes multiple buy/sell operations in sequence for portfolio management and advanced strategies

Both bots are fully integrated with the Nad.fun SDK and handle all the complexity of gas estimation, slippage protection, and transaction management.

---

## ✨ Features

### 🎯 Sniper Bot Features

- ⚡ **Real-time Detection**: WebSocket-based monitoring for zero-latency token detection
- 🎯 **Auto-Buy**: Automatically executes buy transactions on new token creation
- 🛡️ **Smart Filtering**: Whitelist, blacklist, or custom filter functions
- ⏱️ **Rate Limiting**: Built-in protection against excessive transactions
- 📊 **Real-time Statistics**: Track events, successful buys, and failures
- ⚙️ **Fully Configurable**: Customize buy amount, slippage, gas settings, and more
- 🔄 **Auto-Reconnection**: Handles WebSocket disconnections gracefully

### 📦 Bundler Bot Features

- 📦 **Batch Operations**: Execute multiple buys/sells in one go
- 🔄 **Mixed Operations**: Combine buys, sells, and permit-based sells
- ✅ **Auto-Approvals**: Automatically handles token approvals for sell operations
- 🎯 **Permit Support**: Gasless sells using EIP-2612 permit signatures
- 📊 **Detailed Results**: Track success/failure for each operation
- ⚡ **Optimized Execution**: Smart gas management and transaction ordering
- 🛡️ **Error Handling**: Continues execution even if individual operations fail

---

## 🚀 Quick Start

### Prerequisites

- Node.js >= 18.0.0
- A Monad testnet RPC endpoint
- A wallet with MON tokens for trading

### Installation

```bash
# Clone the repository
git clone https://github.com/yourusername/nadfun-sdk-typescript.git
cd nadfun-sdk-typescript

# Install dependencies
bun install
# or
npm install
```

### Configuration

Create a `.env` file:

```bash
RPC_URL=https://your-rpc-endpoint
WS_RPC_URL=wss://your-ws-endpoint
PRIVATE_KEY=0x-your-private-key
BUY_AMOUNT=0.1
SLIPPAGE=5.0
GAS_MULTIPLIER=1.2
```

### Run the Bots

#### 🎯 Sniper Bot

```bash
# Start the sniper bot
bun run example:sniper -- \
  --rpc-url https://your-rpc-endpoint \
  --ws-url wss://your-ws-endpoint \
  --private-key 0x... \
  --buy-amount 0.1 \
  --slippage 5.0 \
  --gas-multiplier 1.2
```

#### 📦 Bundler Bot

```bash
# Execute multiple trades
bun run example:bundler -- \
  --rpc-url https://your-rpc-endpoint \
  --private-key 0x... \
  --operations "buy:0xToken1:0.1,buy:0xToken2:0.2,sell:0xToken1:1000"
```

---

## 💻 Usage Examples

### Sniper Bot - Basic Usage

```typescript
import { SniperBot } from '@nadfun/sdk'

const sniper = new SniperBot({
  rpcUrl: 'https://your-rpc-endpoint',
  wsUrl: 'wss://your-ws-endpoint',
  privateKey: '0x-your-private-key',
  buyAmount: '0.1',        // MON to spend per buy
  slippagePercent: 5.0,    // 5% slippage tolerance
  gasMultiplier: 1.2,      // 20% higher gas for faster execution
})

// Success callback
sniper.onBuySuccess((event, txHash) => {
  console.log(`✅ Sniped ${event.name} (${event.symbol})`)
  console.log(`   Token: ${event.token}`)
  console.log(`   TX: ${txHash}`)
})

// Error callback
sniper.onBuyError((event, error) => {
  console.error(`❌ Failed to snipe ${event.name}:`, error.message)
})

// Start monitoring
await sniper.start()

// Get statistics
const stats = sniper.getStats()
console.log(`Events: ${stats.totalEvents}`)
console.log(`Successful: ${stats.successfulBuys}`)
```

### Sniper Bot - Advanced Filtering

```typescript
const sniper = new SniperBot({
  // ... basic config
  tokenWhitelist: ['0xToken1', '0xToken2'], // Only buy these tokens
  tokenBlacklist: ['0xToken3'],             // Skip these tokens
  tokenFilter: async (event) => {
    // Custom filter logic
    return event.name.length > 3 && 
           event.symbol.length <= 5 &&
           event.virtualMon > parseEther('1000')
  },
})
```

### Bundler Bot - Basic Usage

```typescript
import { BundlerBot } from '@nadfun/sdk'

const bundler = new BundlerBot({
  rpcUrl: 'https://your-rpc-endpoint',
  privateKey: '0x-your-private-key',
  defaultSlippagePercent: 5.0,
  gasMultiplier: 1.0,
})

// Execute multiple operations
const results = await bundler.execute([
  { type: 'buy', token: '0xToken1', amountIn: '0.1' },
  { type: 'buy', token: '0xToken2', amountIn: '0.2' },
  { type: 'sell', token: '0xToken1', amountIn: '1000' },
  { type: 'sellPermit', token: '0xToken2', amountIn: '500' },
])

// Process results
results.forEach((result, i) => {
  if (result.success) {
    console.log(`✅ Operation ${i + 1}: ${result.txHash}`)
  } else {
    console.error(`❌ Operation ${i + 1}: ${result.error}`)
  }
})
```

### Bundler Bot - Portfolio Rebalancing

```typescript
// Buy multiple tokens
const buyResults = await bundler.execute([
  { type: 'buy', token: '0xToken1', amountIn: '0.1' },
  { type: 'buy', token: '0xToken2', amountIn: '0.2' },
  { type: 'buy', token: '0xToken3', amountIn: '0.15' },
])

// Wait for all buys to confirm
await Promise.all(
  buyResults
    .filter(r => r.success && r.txHash)
    .map(r => bundler.trade.publicClient.waitForTransactionReceipt({ 
      hash: r.txHash as `0x${string}` 
    }))
)

// Then sell some tokens
const sellResults = await bundler.execute([
  { type: 'sell', token: '0xToken1', amountIn: '500' },
])
```

---

## 🏗️ Architecture

### Bot Structure

```
src/bots/
├── sniper.ts      # Sniper Bot implementation
├── bundler.ts     # Bundler Bot implementation
└── index.ts       # Exports
```

### Key Components

- **Event Streaming**: Real-time WebSocket monitoring for token creation
- **Gas Estimation**: Network-based gas calculation with smart buffers
- **Slippage Protection**: Configurable tolerance with automatic minimum calculations
- **Error Handling**: Comprehensive error catching and recovery
- **Transaction Management**: Automatic nonce handling and confirmation waiting

### Technology Stack

- **TypeScript**: Full type safety and IntelliSense support
- **Viem**: Ethereum library for blockchain interactions
- **WebSocket**: Real-time event streaming
- **Nad.fun SDK**: Core trading and token operations

---

## 📊 Features in Detail

### Sniper Bot

#### Real-time Monitoring
- Uses WebSocket connections for instant event detection
- Automatically reconnects on connection loss
- Filters events at network and client level for efficiency

#### Smart Execution
- Pre-calculates gas estimates before token creation
- Applies configurable gas multipliers for faster execution
- Handles slippage protection automatically
- Rate limiting to prevent excessive transactions

#### Filtering System
- **Whitelist**: Only buy specific tokens
- **Blacklist**: Skip specific tokens
- **Custom Filters**: Advanced logic based on token metadata
- **Rate Limiting**: Minimum time between buys

### Bundler Bot

#### Batch Execution
- Executes operations sequentially with error handling
- Continues execution even if individual operations fail
- Returns detailed results for each operation

#### Operation Types
- **Buy**: Purchase tokens with MON
- **Sell**: Sell tokens for MON (auto-approval)
- **SellPermit**: Gasless sell using EIP-2612 permits

#### Smart Management
- Automatic token approvals for sell operations
- Gas optimization across multiple transactions
- Transaction confirmation waiting (optional)

---

## 🎓 Use Cases

### Sniper Bot Use Cases

- 🚀 **Early Token Entry**: Buy tokens immediately upon creation
- 📈 **Momentum Trading**: Catch trending tokens before they pump
- 🎯 **Selective Sniping**: Target specific token types with filters
- 📊 **Portfolio Building**: Automatically diversify with new launches
- ⚡ **Speed Trading**: Execute trades faster than manual methods

### Bundler Bot Use Cases

- 💼 **Portfolio Rebalancing**: Buy/sell multiple tokens simultaneously
- 🔄 **Arbitrage Strategies**: Execute multiple trades across tokens
- 📈 **DCA Strategies**: Dollar-cost average across multiple tokens
- ⚡ **MEV Operations**: Bundle transactions for optimal execution
- 🎯 **Batch Operations**: Efficiently manage multiple positions

---

## 🔧 Configuration

### Sniper Bot Configuration

```typescript
interface SniperConfig {
  rpcUrl: string              // HTTP RPC endpoint
  wsUrl: string               // WebSocket RPC endpoint
  privateKey: string          // Wallet private key
  buyAmount: string            // MON to spend per buy (e.g., "0.1")
  slippagePercent: number     // Slippage tolerance (e.g., 5.0)
  gasMultiplier?: number      // Gas price multiplier (default: 1.2)
  maxGasPrice?: bigint        // Maximum gas price in gwei
  minBuyInterval?: number     // Minimum ms between buys (default: 1000)
  autoBuy?: boolean           // Enable/disable auto-buy (default: true)
  tokenWhitelist?: Address[]  // Only buy these tokens
  tokenBlacklist?: Address[]  // Skip these tokens
  tokenFilter?: (event) => boolean  // Custom filter function
}
```

### Bundler Bot Configuration

```typescript
interface BundlerConfig {
  rpcUrl: string              // HTTP RPC endpoint
  privateKey: string          // Wallet private key
  defaultSlippagePercent?: number  // Default slippage (default: 5.0)
  gasMultiplier?: number      // Gas price multiplier (default: 1.0)
  maxGasPrice?: bigint        // Maximum gas price in gwei
  waitForConfirmation?: boolean  // Wait for TX confirmation (default: true)
}
```

---

## 📈 Performance

### Sniper Bot Performance

- **Detection Latency**: < 1 second from token creation to buy execution
- **Gas Optimization**: 20% buffer for reliable execution
- **Success Rate**: Handles network conditions and errors gracefully
- **Resource Usage**: Efficient WebSocket connection management

### Bundler Bot Performance

- **Batch Execution**: Processes multiple operations sequentially
- **Error Recovery**: Continues execution even if operations fail
- **Gas Efficiency**: Optimized gas estimation across operations
- **Transaction Speed**: Configurable confirmation waiting

---

## 🛡️ Security & Best Practices

### Security Features

- ✅ **Private Key Management**: Never logs or exposes private keys
- ✅ **Input Validation**: Validates all token addresses and amounts
- ✅ **Error Handling**: Comprehensive error catching and reporting
- ✅ **Gas Limits**: Maximum gas price protection

### Best Practices

1. **Use Testnet First**: Always test on testnet before mainnet
2. **Monitor Gas Prices**: Adjust multipliers based on network conditions
3. **Set Appropriate Slippage**: 5% is a good default, adjust for volatility
4. **Use Filters**: Implement token filters to avoid unwanted purchases
5. **Monitor Statistics**: Track bot performance and adjust settings
6. **Backup Private Keys**: Store private keys securely

---

## 📝 Examples

### Example 1: Basic Sniper Bot

```typescript
import { SniperBot } from '@nadfun/sdk'

const sniper = new SniperBot({
  rpcUrl: process.env.RPC_URL!,
  wsUrl: process.env.WS_RPC_URL!,
  privateKey: process.env.PRIVATE_KEY!,
  buyAmount: '0.1',
  slippagePercent: 5.0,
})

sniper.onBuySuccess((event, txHash) => {
  console.log(`✅ Sniped: ${event.name} (${event.symbol})`)
})

await sniper.start()
```

### Example 2: Filtered Sniper Bot

```typescript
const sniper = new SniperBot({
  // ... basic config
  tokenFilter: async (event) => {
    // Only buy tokens with names longer than 5 characters
    return event.name.length > 5
  },
})
```

### Example 3: Portfolio Rebalancing

```typescript
const bundler = new BundlerBot({
  rpcUrl: process.env.RPC_URL!,
  privateKey: process.env.PRIVATE_KEY!,
})

// Rebalance portfolio
await bundler.execute([
  { type: 'sell', token: '0xToken1', amountIn: '1000' },
  { type: 'buy', token: '0xToken2', amountIn: '0.2' },
  { type: 'buy', token: '0xToken3', amountIn: '0.15' },
])
```

---

## 🐛 Troubleshooting

### Common Issues

**Sniper Bot not buying tokens:**
- Check WebSocket connection is active
- Verify `autoBuy` is set to `true`
- Check token filters aren't too restrictive
- Ensure sufficient MON balance

**Bundler Bot operations failing:**
- Verify token balances are sufficient
- Check slippage tolerance is appropriate
- Ensure gas prices aren't too high
- Review error messages in results

**Gas estimation failures:**
- Verify RPC endpoint is accessible
- Check network connectivity
- Try increasing gas multiplier
- Use fixed gas limit as fallback

---

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 🙏 Acknowledgments

- Built using the [Nad.fun SDK](https://github.com/naddotfun/nadfun-sdk-typescript)
- Powered by [Viem](https://viem.sh/) for Ethereum interactions
- Designed for the [Monad](https://monad.xyz/) blockchain

---

## 📞 Contact & Support

- 📖 [Full Documentation](README.md)
- 🐛 [Report Issues](https://github.com/yourusername/nadfun-sdk-typescript/issues)
- 💬 [Discussions](https://github.com/yourusername/nadfun-sdk-typescript/discussions)

---

<div align="center">

**Built with ❤️ for the Monad ecosystem**

⭐ Star this repo if you find it useful!

</div>

