# Redemption Mechanism: Three-Channel Design

## Design Objectives

The redemption mechanism must balance three competing objectives:

1. **User Experience**: Provide reasonable exit options
2. **System Security**: Preventing runs that deplete liquidity
3. **Fairness**: Preventing unfair first-come-first-served dynamics

## Three-Channel Rule

### Channel One: Emergency Redemption (T+0)

| Parameters | Settings |
|------------|----------|
| Settlement Time | Instant (same block) |
| Fees | Penalty Rate (Configured by Governance) |
| Budget Constraint | Utilizes Budget-0 Only (L1 Liquidity) |
| Suspension Conditions | Automatically pauses when premium/discount exceeds protection band |

**Design Logic**: Curb runs by imposing penalties rather than outright bans. Users willing to pay high fees gain immediate exit rights, subject to budget caps.

### Channel Two: Standard Redemption (T+7)

| Parameters | Settings |
|------------|----------|
| Settlement Time | T+7 (7 calendar days) |
| Fees | Standard Fee Rate (Configured by Governance) |
| Budget Constraint | Utilization of Budget-7 (L1 + L2 liquidity) |

**Design Logic**: Aligns with the liquidation cycles of most L2 assets, providing ordinary users with reasonable exit time expectations.

### Channel Three: Queued Redemption

When redemption demand exceeds the safety threshold, new redemptions enter the queue:

| Parameters | Settings |
|------------|----------|
| Trigger Condition | Cumulative pending redemptions > Safety threshold |
| Settlement Method | Batch Settlement |
| Priority | Time + Amount Weighted |

### Queue Priority Formula

$$
\text{Priority}_i = w_T \cdot f(\tau_i) + w_A \cdot g(a_i)
$$

Where:
- **τ_i**: Queue waiting time for request i
- **a_i**: Redemption amount for request i
- **f, g**: Normalization functions (log or capped scaling to prevent order splitting attacks)
- **w_T, w_A**: Weight coefficients (configured by governance)

## Redemption Flow

```
┌─────────────────┐
│ Redemption      │
│ Request         │
└──────┬──────────┘
       │
       ↓
┌──────────────────────────────┐
│ Check Safety Threshold & Budgets│
└──────────────┬───────────────┘
               │
    ┌──────────┼──────────┬───────────────┐
    ↓          ↓          ↓               ↓
┌────────┐  ┌────────┐  ┌───────────┐  ┌────────────────┐
│Budget-0│  │Budget-7│  │ Over-thresh│  │ Protection Band │
│Available│ │Available│ │           │  │ Triggered       │
└───┬────┘  └───┬────┘  └───┬───────┘  └────┬────────────┘
    ↓           ↓           ↓                ↓
┌────────┐  ┌────────┐  ┌──────────┐     ┌──────────────┐
│T+0     │  │T+7     │  │Queued    │     │Pause T+0      │
│(Penalty)│ │(Standard)│ │Redemption│     │(Emergency)    │
└────────┘  └────────┘  └──────────┘     └──────────────┘
```

## Budget Transparency

All liquidity budgets and queue statuses **are publicly disclosed in real-time**. Users can query:

- Budget utilization rates for each layer
- Queue depths and positions
- Estimated settlement times

**Design rationale**: Transparency is the foundation of trust. Concealing queue status would cause the market to price in worst-case assumptions.

## Fee Structure

| Channel | Fee Type | Typical Range | Purpose |
|---------|----------|---------------|---------|
| T+0 | Penalty Fee | 1-5% | Discourage panic exits |
| T+7 | Standard Fee | 0.1-0.5% | Cover operational costs |
| Queue | No additional fee | 0% | Fair access for patient users |

## Anti-Gaming Measures

### Order Splitting Prevention
The priority formula uses logarithmic scaling for amounts, reducing the incentive to split large orders into many small ones.

### Front-Running Protection
- Queue position is based on block timestamp, not transaction order within a block
- Batch settlement randomizes execution order within priority tiers

### Budget Gaming Prevention
- Budgets are calculated on a rolling basis
- No "reservation" of future budget capacity
- All budget updates are atomic with redemption execution
