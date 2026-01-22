# Premium/Discount Protection Band and Emergency Mechanism

> **⚠️ Implementation Status: PLANNED**
>
> The Protection Band mechanism described in this document is part of the Phase 2 roadmap. Current contract implementation does not include automated TWAP monitoring or T+0 suspension triggers. Manual risk control via keeper roles is available in the current release.

## Weakly Coupled Pricing Model

PP maintains two concurrent prices:

| Price Type | Description | Determination |
|------------|-------------|---------------|
| **NAV** | Net asset value per share within Prime Vault | Audit/reconciliation inputs |
| **P_mkt** | Market trading price in the DEX pool | Supply and demand |

### Objectives

- **Long-term**: P_mkt fluctuates around NAV
- **Short-term**: Allow premiums/discounts to exist, reflecting liquidity premiums and risk pricing

### Why Not Enforce 1:1 Pegging?

1. Forced pegging requires infinite liquidity support—unrealistic for alternative assets
2. Premiums/discounts themselves serve as useful risk signals
3. Arbitrageurs converge spreads within rules, naturally building liquidity depth

## Protection Band Parameters

**Initial protection band**: ±15%

### Defining Deviation

$$
D_t=\left|\frac{P_{\text{mkt}}}{\mathrm{NAV}_t}-1\right|
$$

Price sampling uses **TWAP (Time-Weighted Average Price)** to prevent single-point manipulation.

## Trigger Actions

When D_t ≥ 15%:

| Action | Explanation |
|--------|-------------|
| **Automatically suspend T+0** | Emergency Redemption Channel Closed |
| T+7 remains open | Standard redemptions remain unaffected |
| Queued redemptions maintained | Queue processing continues |

### Resumption Conditions

1. Deviation returns within protection band (D_t < 15%)
2. Multi-signature risk control confirms no ongoing manipulation

## State Diagram

```
┌─────────────────────────────────────┐
│             Normal State            │
│          D < 15%, T+0 Open          │
└──────────────────┬──────────────────┘
                   │ D ≥ 15%
                   ↓
┌─────────────────────────────────────┐
│           Protected State           │
│         D ≥ 15%, T+0 Paused         │
└──────────────────┬──────────────────┘
                   │ D < 15% AND Risk Check OK
                   ↓
┌─────────────────────────────────────┐
│        Return to Normal State       │
└─────────────────────────────────────┘
```

## Design Logic

The protection band interrupts the death spiral of:

```
Discount → Panic → Run → Greater Discount → ...
```

By suspending T+0 rather than full redemptions:
- User exit paths are preserved (T+7 and Queue remain open)
- Instant arbitrage opportunities are eliminated
- Time is created for market to absorb information
- System stability is maintained without total lockup

## TWAP Implementation

### Why TWAP?

Single-point price readings are vulnerable to manipulation:
- Flash loan attacks
- Sandwich attacks
- Low-liquidity periods

### TWAP Parameters

| Parameter | Value | Rationale |
|-----------|-------|-----------|
| Window | 1 hour | Balance between responsiveness and manipulation resistance |
| Sample frequency | Every block | Capture all price movements |
| Outlier exclusion | Top/bottom 5% | Remove manipulation attempts |

## Emergency Scenarios

### Scenario: Sudden Large Discount

1. Market price drops sharply (e.g., -20%)
2. D_t exceeds 15% threshold
3. T+0 channel automatically suspends
4. Users can still exit via T+7 or Queue
5. Arbitrageurs buy cheap PP
6. Price gradually recovers
7. When D_t < 15%, T+0 resumes

### Scenario: Manipulation Attempt

1. Attacker tries to crash price
2. TWAP smooths the attack signal
3. If TWAP still triggers protection band:
   - T+0 suspends (blocking instant arbitrage)
   - Risk control team investigates
   - Resume only after confirmation of no manipulation

## Governance Parameters

| Parameter | Default | Governance Adjustable |
|-----------|---------|----------------------|
| Protection band width | ±15% | Yes |
| TWAP window | 1 hour | Yes |
| Resume threshold | D < 15% | Yes |
| Multi-sig requirement | 3/5 | Yes (for emergency override) |
