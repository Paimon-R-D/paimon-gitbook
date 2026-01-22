# System Architecture: Three-Layer Closed Loop

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                   Governance Layer (Vote/Gauge)                  │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐         │
│   │ Emissions    │ ←→ │ Parameters  │ ←→ │ Access       │         │
│   │ Allocation   │     │ Governance  │     │ Approval     │         │
│   └─────────────┘    └─────────────┘    └─────────────┘         │
└────────────────────────────┬────────────────────────────────────┘
                             │ Incentives + Resources
                             ↓
┌─────────────────────────────────────────────────────────────────┐
│                    Liquidity Layer (DEX Pools)                   │
│                                                                 │
│   PP/USDC Pool ←──────→ Arbitrageurs ←──────→ NAV Anchoring     │
│                                                                 │
└────────────────────────────┬────────────────────────────────────┘
                             │ Secondary Trading
                             ↓
┌─────────────────────────────────────────────────────────────────┐
│                    Aggregation Layer (Prime Vault)               │
│                                                                 │
│   User Assets ──→ ERC-4626 Vault ──→ PP Share Token             │
│                     │                                           │
│              ┌──────┴──────┐                                   │
│              ↓             ↓                                   │
│          NAV Calculation   Redemption State Machine             │
│                                                                 │
└────────────────────────────┬────────────────────────────────────┘
                             │ Asset Inflow
                             ↓
┌─────────────────────────────────────────────────────────────────┐
│                    Issuance Layer (Launchpad)                   │
│                                                                 │
│   RWA ──┐                                                       │
│   PE  ─┼──→ Hard Gate Review ──→ Soft Scoring ──→ Prime Candidate│
│   Funds─┘     (Custody/Audit/Disclosure)                        │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Participant Roles

| Role | Function | Incentive Source |
|------|----------|------------------|
| **Issuer** | Submit assets, fulfill disclosure obligations | Financing channels, liquidity premium |
| **Prime Users** | Subscription/Redemption Process | NAV growth, underlying asset returns |
| **LP/Market Maker** | Providing Secondary Market Liquidity | Transaction Fees, Mining Incentives |
| **Governance participants** | Vote on emission and parameters | Protocol fee sharing, governance rights |
| **Authorized Market Makers (AMP)** | Professional Arbitrage & Liquidity Provision | Priority Settlement, Fee Discounts |

## Layer Descriptions

### Issuance Layer (Launchpad)
The entry point for alternative assets into the Paimon ecosystem. Assets must pass hard threshold checks (custody, audit, disclosure) before being eligible for inclusion in Prime Vault.

→ [Learn more about Launchpad](launchpad.md)

### Aggregation Layer (Prime Vault)
The core asset management layer built on ERC-4626 standard. Manages NAV calculation, PP issuance, and the redemption state machine.

→ [Learn more about Prime Vault](prime-vault.md)

### Liquidity Layer (DEX)
Secondary market for PP trading. Provides price discovery mechanism and allows market-based liquidity through arbitrage.

→ [Learn more about DEX Strategy](../products/dex-strategy.md)

### Governance Layer
Controls protocol parameters, emission allocation, and access approvals through vePAIMON voting mechanism.

→ [Learn more about Governance](../governance/README.md)
