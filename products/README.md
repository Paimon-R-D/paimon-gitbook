# Tranche Vault

## Design Motivation

The underlying asset yield in the PPT is approximately 6% APY. This is a real, exogenous yield, yet it lacks appeal in the DeFi market.

**Core challenge**: How to offer higher yields to risk-seeking users without increasing underlying asset risk?

**Solution**: Yield Tranching — Allowing some users to bear greater risk in exchange for leveraged returns.

## Tranche Vault Structure

Tranche Vault is an independent contract layer built on top of PPT, splitting PPT's yield into two tranches:

```
┌─────────────────────────────────────────────────────────────────┐
│                          Tranche Vault                           │
│            (Independent Contract, Holds PPT as Assets)            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Users deposit PPT                                              │
│        │                                                        │
│        ↓                                                        │
│   Choose share class                                            │
│        │                                                        │
│   ┌────┴────┐                                                   │
│   ↓         ↓                                                   │
│  sPPT      jPPT                                                 │
│ Senior    Junior                                                │
│                                                                 │
│ • Fixed 4% yield*    • Floating yield (all residual)             │
│ • Senior repayment   • Subordinated repayment                    │
│ • Lower risk         • Higher risk, higher upside                │
│ • No governance      • Can stake to earn esPAIMON                │
│                                                                 │
│ * The 4% fixed rate is a governance-adjustable parameter.       │
│   Rate changes require Medium Risk governance approval and      │
│   apply only to new deposits after the change takes effect.     │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## Yield Distribution Mechanism

**Allocation Priority**: sPPT receives fixed returns first, while jPPT receives the remaining portion.

### Formula

Let V represent the total value of PPT held by the Vault, r_s denote the proportion of sPPT, r_j = 1 − r_s denote the proportion of jPPT, and Y_total denote the total yield of PPT.

- Y_sPPT = 4% (Fixed)

$$
Y_{jPPT}=\frac{Y_{total}-Y_{sPPT}\cdot r_s}{r_j}
$$

### Scenario Analysis (Assuming sPPT:jPPT = 70:30)

| Total PPT Return | sPPT Yield | jPPT Return | Explanation |
|------------------|------------|-------------|-------------|
| 10% | 4% | 24% | Bull Market, jPPT Gains Significant Leverage |
| 8% | 4% | 17.3% | Normal preference |
| 6% | 4% | 10.7% | Base Scenario |
| 4% | 4% | 4% | Break-even point |
| 2% | 4% | -2.7% | jPPT begins to incur losses |
| 0% | 4% | -9.3% | jPPT bears all downside risk |

**Key Features**: jPPT provides a safety cushion for sPPT, absorbing downside volatility; in return, jPPT gains leveraged returns during uptrends.

## Epoch Settlement Mechanism

PPT's NAV fluctuates continuously, while Tranche Vault employs fixed-cycle settlements:

```
Epoch 1           Epoch 2           Epoch 3
[  7 days ]       [  7 days ]       [  7 days ]
    │                 │                 │
    ↓                 ↓                 ↓
Compute NAV        Compute NAV        Compute NAV
Change             Change             Change
    │                 │                 │
    ↓                 ↓                 ↓
Distribute         Distribute         Distribute
Yield (sPPT first) Yield (sPPT first) Yield (sPPT first)
Residual to jPPT   Residual to jPPT   Residual to jPPT
```

### Settlement Steps

1. At the end of each Epoch, calculate the change in PPT NAV held by the Vault
2. Calculate sPPT's earned yield: 4% ÷ 52 × r_s × V_s
3. The remaining amount (positive or negative) is fully allocated to jPPT
4. If the remainder is negative, deduct it from the net value of jPPT shares

## Entries and Exits

### Deposit

| Operation | Rule |
|-----------|------|
| PPT → sPPT | Anytime, calculated based on current Vault net value |
| PPT → jPPT | At any time, calculate shares based on current Vault NAV |

### Redemption

| Operation | Rules |
|-----------|-------|
| sPPT → PPT | Queued redemption, **with priority over jPPT** |
| jPPT → PPT | Queued redemption, lower priority than sPPT |
| Staked jPPT | Must first unstake |

### Priority Repayment Implementation

```
Redemption Queue Processing Order

1. Process all sPPT redemption requests
2. If liquidity remains, process jPPT redemption requests
3. Any excess enters the waiting queue
```

## Risk Control: Junior Layer Safety Buffer

**Issue**: If the jPPT ratio is too low, it cannot provide sufficient protection for sPPT.

**Mechanism**: Minimum Junior Ratio Constraint

| Parameters | Value | Description |
|------------|-------|-------------|
| Target Junior Ratio | 30% | System Design Target |
| Minimum Junior Ratio | 20% | Mandatory Lower Limit |
| Trigger Action | Pause new sPPT deposits | Prevent excessive leverage |
| Resumption Criteria | Junior ratio returns to 25% | Includes buffer range |

### Extreme Scenario

If PPT NAV continues to decline, causing jPPT NAV to reach zero:

1. jPPT holders lose all principal
2. sPPT begins directly bearing PPT downside risk
3. At this point, sPPT effectively degrades to a standard PPT exposure
4. Emergency state is triggered, suspending all new deposits

## Integration with Governance System

**Incentive acquisition path (non-governance)**: jPPT staking incentives → esPAIMON (vesting) → PAIMON. 

Governance participation requires **PAIMON locked into vePAIMON**.

```
┌─────────────────────────────────────────────────────────────────┐
│                 Full Path from jPPT to Governance                │
└─────────────────────────────────────────────────────────────────┘

jPPT Holder
      │
      ↓ Stake into jPPT Gauge
      │
      ↓
Earn esPAIMON (mining rewards, non-transferable)
      │
      ↓
Vest (90-day linear)
(to unlock as PAIMON)
      │
      ↓
PAIMON (transferable)
      │
      ↓ Lock
      │
vePAIMON
      │
      ├──────────────────────┐
      ↓                      ↓
Gauge Voting              Protocol Fee Share
(determines emissions)
```

## Comparison with Failure Modes

| Dimension | Terra/Anchor | Paimon Tranche |
|-----------|--------------|----------------|
| Revenue Sources | Token Subsidies (Endogenous) | Underlying Assets + Tiered Amplification (Exogenous) |
| High-Yield Mechanism | Money Printing | Risk redistribution |
| Downside Protection | None | jPPT as a Safety Net |
| Liquidation risk | Death spiral | No Lending, No Liquidation |
| Worst-case scenario | Systemic Collapse | jPPT becomes worthless, sPPT degrades to PPT |
