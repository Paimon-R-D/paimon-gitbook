# Authorized Market Maker (AMP) System

> **⚠️ Implementation Status: PLANNED**
>
> The AMP (Authorized Market Maker) system described in this document is part of the Phase 2 roadmap. Current release does not include on-chain AMP qualification, tiering, or performance monitoring. Market making activities are managed through off-chain agreements.

## Design Purpose

Enables professional market makers to arbitrage within rules, enhancing market depth and price efficiency, but must be institutionalized to prevent it from becoming a run accelerator.

## Access and Obligations

| Category | Requirements |
|----------|--------------|
| **Qualification** | Completion of KYC, margin deposit, and meeting risk control score requirements |
| **Benefits** | Higher redemption limits, lower fees, priority settlement |
| **Obligations** | Minimum depth commitment, maximum spread limit, expanded quotes upon deviation band trigger |
| **Penalties** | Margin Deductions for Violations, Downgrade/Blacklisting for Repeated Violations |

## Arbitrage Mechanism

AMP executes arbitrage in the following scenarios:

| Scenario | Action | Effect |
|----------|--------|--------|
| P_mkt < NAV (discount) | Buy PP → Redemption → Acquire underlying assets | Pushes up P_mkt, narrows discount |
| P_mkt > NAV (premium) | Subscribe PP → Sell | Pressure down P_mkt, narrow premium |

### Constraints

Arbitrage scale is constrained by redemption budget and subscription limits to prevent unlimited arbitrage from depleting liquidity.

## AMP Tiers

| Tier | Margin Requirement | Redemption Limit | Fee Discount | Obligations |
|------|-------------------|------------------|--------------|-------------|
| Bronze | 100K USDC | 2x standard | 10% | Basic depth commitment |
| Silver | 500K USDC | 5x standard | 25% | Higher depth, tighter spread |
| Gold | 2M USDC | 10x standard | 50% | Priority settlement, expanded quotes during stress |

## Operational Requirements

### Depth Commitments

AMPs must maintain minimum liquidity within defined price ranges:

| Tier | Depth (each side) | Price Range |
|------|------------------|-------------|
| Bronze | 50K USDC | ±2% of NAV |
| Silver | 200K USDC | ±3% of NAV |
| Gold | 1M USDC | ±5% of NAV |

### Spread Limits

| Market Condition | Maximum Spread |
|------------------|----------------|
| Normal (D < 5%) | 0.5% |
| Elevated (5% ≤ D < 10%) | 1.0% |
| Stressed (D ≥ 10%) | 2.0% |

### Protection Band Response

When the protection band is triggered (D ≥ 15%):

1. AMPs must **expand** quotes (not withdraw)
2. Wider spread permitted (up to 5%)
3. Additional depth commitment may be required
4. Priority settlement rights activated

## Performance Monitoring

### Scoring Factors

| Factor | Weight | Description |
|--------|--------|-------------|
| Uptime | 30% | Time with active quotes |
| Depth | 30% | Average liquidity provided |
| Spread | 20% | Average spread maintained |
| Stress Response | 20% | Performance during volatile periods |

### Score Thresholds

| Score | Status | Consequence |
|-------|--------|-------------|
| ≥ 80 | Good Standing | Full benefits |
| 60-79 | Warning | Benefits reduced by 25% |
| 40-59 | Probation | Benefits reduced by 50% |
| < 40 | Suspension | AMP status revoked |

## Penalty System

### Violation Categories

| Category | Example | Penalty |
|----------|---------|---------|
| Minor | Brief quote withdrawal | Warning |
| Moderate | Spread violation during normal conditions | 5% margin deduction |
| Major | Depth violation during stress | 20% margin deduction |
| Severe | Repeated major violations | Blacklist |

### Appeal Process

1. AMP submits appeal within 48 hours
2. Risk committee reviews within 7 days
3. Decision is final (governance can override in extreme cases)

## Integration Points

### With Prime Vault

- AMPs have priority access to redemption budget
- Subscription limits may be higher for AMPs
- Settlement is processed before general queue

### With Protection Band

- AMPs are notified immediately of band triggers
- Expanded quote obligations activated
- Priority settlement ensures arbitrage can occur

### With Governance

- AMP qualification standards set by governance
- Margin requirements adjustable
- Violation penalties configurable

## Benefits Summary

| Benefit | Bronze | Silver | Gold |
|---------|--------|--------|------|
| Redemption Limit | 2x | 5x | 10x |
| Fee Discount | 10% | 25% | 50% |
| Priority Settlement | No | Partial | Yes |
| Governance Weight Bonus | No | No | Yes |

## Becoming an AMP

1. **Application**: Submit KYC documentation and operational track record
2. **Due Diligence**: Risk committee reviews within 14 days
3. **Margin Deposit**: Transfer required collateral to escrow
4. **Onboarding**: Technical integration and testing
5. **Activation**: Begin market making operations
