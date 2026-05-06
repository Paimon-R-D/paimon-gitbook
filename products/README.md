# Tranche Vault

{% hint style="warning" %}
**Phase 2 Concept — Not Deployed**

This document describes a **future design**. There is **no Tranche Vault contract on BSC mainnet today** and no `sPP` / `jPP` / `esPAIMON` token has been issued. The entire mechanism below (Senior/Junior split, Epoch settlement, esPAIMON staking incentive) is a design proposal subject to change.

For products that are **live on mainnet today**, see:
- [Paimon Prime Vault (PP)](../architecture/prime-vault.md) — ERC-4626 RWA fund
- [Pre-IPO SPV Tokens (pSPCX / xSPCX)](pre-ipo-spv.md) — SpaceX SPV tokenization
{% endhint %}

## Design Motivation

The underlying asset yield in the PP is approximately 6% APY. This is a real, exogenous yield, yet it lacks appeal in the DeFi market.

**Core challenge**: How to offer higher yields to risk-seeking users without increasing underlying asset risk?

**Solution**: Yield Tranching — Allowing some users to bear greater risk in exchange for leveraged returns.

## Tranche Vault Structure

Tranche Vault is an independent contract layer built on top of PP, splitting PP's yield into two tranches:

```
┌─────────────────────────────────────────────────────────────────┐
│                          Tranche Vault                           │
│            (Independent Contract, Holds PP as Assets)            │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│   Users deposit PP                                              │
│        │                                                        │
│        ↓                                                        │
│   Choose share class                                            │
│        │                                                        │
│   ┌────┴────┐                                                   │
│   ↓         ↓                                                   │
│  sPP      jPP                                                 │
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

**Allocation Priority**: sPP receives fixed returns first, while jPP receives the remaining portion.

### Formula

Let V represent the total value of PP held by the Vault, r_s denote the proportion of sPP, r_j = 1 − r_s denote the proportion of jPP, and Y_total denote the total yield of PP.

- Y_sPP = 4% (Fixed)

$$
Y_{jPP}=\frac{Y_{total}-Y_{sPP}\cdot r_s}{r_j}
$$

### Scenario Analysis (Assuming sPP:jPP = 70:30)

| Total PP Return | sPP Yield | jPP Return | Explanation |
|------------------|------------|-------------|-------------|
| 10% | 4% | 24% | Bull Market, jPP Gains Significant Leverage |
| 8% | 4% | 17.3% | Normal preference |
| 6% | 4% | 10.7% | Base Scenario |
| 4% | 4% | 4% | Break-even point |
| 2% | 4% | -2.7% | jPP begins to incur losses |
| 0% | 4% | -9.3% | jPP bears all downside risk |

**Key Features**: jPP provides a safety cushion for sPP, absorbing downside volatility; in return, jPP gains leveraged returns during uptrends.

## Epoch Settlement Mechanism

PP's NAV fluctuates continuously, while Tranche Vault employs fixed-cycle settlements:

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
Yield (sPP first) Yield (sPP first) Yield (sPP first)
Residual to jPP   Residual to jPP   Residual to jPP
```

### Settlement Steps

1. At the end of each Epoch, calculate the change in PP NAV held by the Vault
2. Calculate sPP's earned yield: 4% ÷ 52 × r_s × V_s
3. The remaining amount (positive or negative) is fully allocated to jPP
4. If the remainder is negative, deduct it from the net value of jPP shares

## Entries and Exits

### Deposit

| Operation | Rule |
|-----------|------|
| PP → sPP | Anytime, calculated based on current Vault net value |
| PP → jPP | At any time, calculate shares based on current Vault NAV |

### Redemption

| Operation | Rules |
|-----------|-------|
| sPP → PP | Queued redemption, **with priority over jPP** |
| jPP → PP | Queued redemption, lower priority than sPP |
| Staked jPP | Must first unstake |

### Priority Repayment Implementation

```
Redemption Queue Processing Order

1. Process all sPP redemption requests
2. If liquidity remains, process jPP redemption requests
3. Any excess enters the waiting queue
```

## Risk Control: Junior Layer Safety Buffer

**Issue**: If the jPP ratio is too low, it cannot provide sufficient protection for sPP.

**Mechanism**: Minimum Junior Ratio Constraint

| Parameters | Value | Description |
|------------|-------|-------------|
| Target Junior Ratio | 30% | System Design Target |
| Minimum Junior Ratio | 20% | Mandatory Lower Limit |
| Trigger Action | Pause new sPP deposits | Prevent excessive leverage |
| Resumption Criteria | Junior ratio returns to 25% | Includes buffer range |

### Extreme Scenario

If PP NAV continues to decline, causing jPP NAV to reach zero:

1. jPP holders lose all principal
2. sPP begins directly bearing PP downside risk
3. At this point, sPP effectively degrades to a standard PP exposure
4. Emergency state is triggered, suspending all new deposits

## Integration with Governance System

**Incentive acquisition path (non-governance)**: jPP staking incentives → esPAIMON (vesting) → PAIMON. 

Governance participation requires **PAIMON locked into vePAIMON**.

```
┌─────────────────────────────────────────────────────────────────┐
│                 Full Path from jPP to Governance                │
└─────────────────────────────────────────────────────────────────┘

jPP Holder
      │
      ↓ Stake into jPP Gauge
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
| Downside Protection | None | jPP as a Safety Net |
| Liquidation risk | Death spiral | No Lending, No Liquidation |
| Worst-case scenario | Systemic Collapse | jPP becomes worthless, sPP degrades to PP |
