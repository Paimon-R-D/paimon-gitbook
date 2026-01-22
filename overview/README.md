# Executive Summary

## Core Problem

Tokenized alternative assets (RWAs, private equity, private funds) face a structural dilemma—a fundamental mismatch between on-chain liquidity and the off-chain asset realization cycle. Existing solutions either sacrifice liquidity for compliance or security for instant exit, offering no compromise.

## Paimon's Solution

Constructing a three-tier closed-loop architecture that deeply integrates real returns from alternative assets with native DeFi liquidity mechanisms:

| Tier | Features | Core Outputs |
|------|----------|--------------|
| **Launchpad** | Compliance-driven issuance and access screening | Audited On-Chain Assets |
| **Prime Vault** | ERC-4626 Asset Aggregation and NAV Management | PPT Share Token |
| **DEX + Vote** | Secondary Market Liquidity and Governance Incentives | Price Discovery + Yield Distribution |

## Key Innovations

### Yield Tiering (Tranche)
PPT holders may choose Senior (sPPT, fixed 4% yield) or Junior (jPPT, risk-bearing for leveraged returns), catering to diverse risk preferences.

### vePAIMON Governance Model
PAIMON may be locked into vePAIMON to obtain time-weighted governance power for gauge voting, protocol parameter signaling, and value-return mechanisms. PAIMON can be acquired through multiple paths, including ecosystem allocations, market activity, and protocol incentives (e.g., jPPT staking rewards).

### Tiered Liquidity Budget
- **L1 (Instant)**: T+0 emergency liquidity
- **L2 (T+7)**: Standard redemption liquidity
- **L3 (Quarterly)**: Long-term allocation liquidity

This tiered approach aligns with real asset liquidation cycles.

### Three-Channel Redemption Mechanism
- **T+0 (Penalized)**: Emergency exit with penalty fees
- **T+7 (Standard)**: Normal redemption timeline
- **Queued Redemption**: For excess demand scenarios

This design balances liquidity demand and system security.

### Discount/Premium Protection Band
Automatic suspension of emergency redemptions at ±15% deviation to prevent run feedback loops.

## Target Users

| User Type | Value Proposition |
|-----------|-------------------|
| DeFi-native institutions | Alternative asset exposure with on-chain liquidity |
| Liquidity providers | Real yields from underlying assets |
| Asset issuers | Tokenization and secondary market access |
