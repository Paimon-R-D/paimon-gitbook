# Problem Definition: Why Tokenization ≠ Liquidity

## 1.1 The Liquidity Paradox of Alternative Assets

The alternative asset market exceeds $13 trillion, yet its on-chain adoption faces fundamental barriers.

### Limitations of Traditional Frameworks

- **Ambiguous Rights Chain**: Unclear mapping between on-chain tokens and off-chain legal entitlements
- **Valuation Black Box**: NAV relies on infrequent external audits, causing severe information asymmetry
- **Exit Friction**: Quarterly/annual redemption windows conflict with DeFi's 24/7 trading expectations

### Challenges of Existing RWA Protocols

| Solution Type | Representative Projects | Issue |
|--------------|------------------------|-------|
| Pure compliance approach | Securitize | Extremely poor liquidity; secondary market exists in name only |
| Pure DeFi approach | Early RWA pools | Lacks real asset backing, unsustainable returns |
| Hybrid approach | Most Projects | Neither here nor there—neither compliant nor liquid |

## 1.2 Paimon's Design Trade-offs

Paimon does not pursue the goal of enabling all assets to "exit instantly like ETH"—the nature of alternative assets makes this objective unfeasible.

### Core Principles

1. **Liquidity is a budget, not a right**: Instant exit is a limited resource that must be rationally allocated through cost mechanisms

2. **Weakly Coupled Pricing**: The intrinsic value (NAV) of PPs and market price (P_mkt) are permitted to diverge, converging under controlled rules

3. **Transparency over promises**: Publicly disclose all liquidity budgets and queue statuses, allowing the market to price risk premiums autonomously

### Design Philosophy

```
Traditional Approach:
Asset → Token → "Instant Liquidity" (Promise)
         ↓
      Failure when stressed

Paimon Approach:
Asset → Token → Tiered Liquidity (Budget-based)
         ↓
      Sustainable under stress
```

The key insight is that **alternative assets cannot and should not promise instant liquidity**. Instead, Paimon creates a transparent, rules-based system where:

- Users understand the liquidity constraints upfront
- The cost of liquidity reflects its true scarcity
- Market participants can price risk appropriately
- The system remains stable under redemption pressure
