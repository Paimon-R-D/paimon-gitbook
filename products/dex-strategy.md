# DEX Strategy: v2 Launch, v4 Evolution

## Phase 1: v2-like AMM

### Reasons for Selection

- Simple implementation, strong ecosystem compatibility
- Low deployment and integration costs
- Sufficient to meet early liquidity bootstrapping needs

### Core Configuration

| Parameter | Setting |
|-----------|---------|
| Primary Pair | PP/USDC |
| Market Making | Constant product (x·y=k) |
| Oracle | Standard TWAP |

## Phase 2: v4 Hooks

Uniswap v4's hooks system enables custom logic execution before/after swap/liquidity operations, enabling Paimon to:

| Hook Type | Use Cases |
|-----------|-----------|
| `beforeSwap` | Compliance checks, restricted trade gating |
| `afterSwap` | Dynamic fee rate adjustment, risk control signal updates |
| `beforeAddLiquidity` | LP Eligibility Verification |
| `afterRemoveLiquidity` | Liquidity Withdrawal Restrictions |

## Evolution Path

```
v2-like AMM            v4 Hooks              Advanced Features
(fast launch)   ──→   (base integration) ──→ (risk/compliance at pool level)
   │                     │                      │
   ↓                     ↓                      ↓
Build trading data     Pool-level programmability  Dynamic fees, restricted trading
Depth/fee baseline     Risk signal integration     Compliance gating, custom oracles
```

## Liquidity Incentives

### LP Rewards

| Source | Description |
|--------|-------------|
| Trading Fees | Standard AMM fees from swap volume |
| Mining Incentives | PAIMON emissions via Gauge voting |
| External Incentives | Third-party rewards through Nitro system |

### Gauge Integration

The PP/USDC pool has its own Gauge that receives emissions based on vePAIMON voting allocation. This creates a direct link between governance participation and liquidity depth.

## Arbitrage Mechanics

### Market Arbitrage

| Scenario | Action | Effect |
|----------|--------|--------|
| P_mkt < NAV (discount) | Buy PP → Redemption → Acquire underlying assets | Pushes up P_mkt, narrows discount |
| P_mkt > NAV (premium) | Subscribe PP → Sell | Pressure down P_mkt, narrow premium |

### Constraints

Arbitrage scale is constrained by:
- Redemption budget limits
- Subscription caps
- Protection band triggers

This prevents unlimited arbitrage from depleting liquidity while still enabling price convergence.

## Price Discovery

### NAV vs Market Price

| Price | Source | Update Frequency |
|-------|--------|------------------|
| NAV | Audits, reconciliation, disclosure | Weekly minimum |
| P_mkt | DEX trading | Real-time |

### Convergence Mechanism

1. **Long-term**: Arbitrageurs drive P_mkt toward NAV
2. **Short-term**: Premium/discount reflects liquidity premium
3. **Protection**: Band limits prevent extreme divergence

## Integration Points

### With Prime Vault
- Subscription creates PP supply
- Redemption removes PP supply
- Both affect DEX liquidity

### With Protection Band
- DEX price feeds into deviation calculation
- Band trigger suspends T+0 channel
- TWAP smooths price signals

### With Governance
- Gauge voting directs emissions
- Parameter governance (fees, caps)

## Future Considerations

### v4 Hook Possibilities

| Feature | Hook Implementation |
|---------|---------------------|
| KYC/AML gating | `beforeSwap` checks whitelist |
| Dynamic fees | `afterSwap` adjusts based on volatility |
| Circuit breakers | `beforeSwap` checks price deviation |
| Compliance reporting | `afterSwap` emits events for tracking |

### Cross-Chain Expansion

- Bridge integration for multi-chain liquidity
- Unified PP across chains
- Cross-chain arbitrage opportunities
