# PAIMON Token Economics

## Table of Contents

- [1. Token System Overview](#1-token-system-overview)
- [2. Emission and Budget Allocation](#2-emission-and-budget-allocation)
- [3. Reward Distribution and Vesting](#3-reward-distribution-and-vesting)
- [4. Governance and Incentives](#4-governance-and-incentives)
- [5. Distribution Security](#5-distribution-security)
- [6. Treasury and Capital Circulation](#6-treasury-and-capital-circulation)
- [7. Metrics and Risk Control](#7-metrics-and-risk-control)

---

## 1. Token System Overview

> **Brief Description**: The governance model is centered on **PAIMON locked into vePAIMON**. Participation incentives (e.g., jPPT staking) may distribute **esPAIMON**, which can vest into **PAIMON** and optionally be locked into **vePAIMON** to join governance.

### Token Stack

| Token/Certificate | Type | Total Supply/Cap | Core Function | Issuing Entity |
|-------------------|------|------------------|---------------|----------------|
| **PPT** | ERC-20 (ERC-4626 Vault Shares) | Varies with Prime Vault size | Prime Vault Shares, NAV Pricing, Redemption Three-Channel Access, DEX Trading Assets | Prime Vault (ERC-4626) |
| **sPPT** | Tranche Shares | Varies with Tranche Vault size | Fixed-income tier (e.g., 4%), priority repayment, no governance participation (see [Tranche Vault](../products/README.md)) | Tranche Vault |
| **jPPT** | Tranche Share | Varies with Tranche Vault size | Exposure to downside risk for leveraged returns; can be staked to earn esPAIMON (see [Tranche Vault](../products/README.md)) | Tranche Vault |
| **PAIMON** | ERC-20 | 10,000,000,000 (Hard Cap) | Governance, staking, liquidity incentives, external incentive assets, protocol value capture vehicle | EmissionManager + Treasury |
| **esPAIMON** | ERC-20 Vesting | Subject to emission budget/quota constraints | jPPT Staking Mining Rewards; Can be vested to unlock as PAIMON; Also usable for Boost | Reward Distributor / Treasury |
| **vePAIMON** | ve (Voting Escrow) | No cap | Gauge Voting, Emission Allocation, Protocol Fee Sharing (per governance charter) | Voting Escrow (vePAIMON) |

### Token Allocation

| Party | Percentage | Liquidity / Vesting Term |
|-------|------------|--------------------------|
| Strategic Round | 10% | 6M cliff + 18M linear |
| Private Round | 10% | 6M cliff + 18M linear |
| Team | 15% | 12M cliff + 36M linear |
| Advisors | 5% | 6M cliff + 24M linear |
| Ecosystem | 5% | 0–3% at TGE + 12–24M milestone-based release |
| Community | 30% | Long-term emission; primarily via esPAIMON / delayed release |
| Liquidity & Market Making | 5% | Available at TGE (multisig custody) |
| Foundation / DAO Reserve | 10% | Multisig custody |
| Marketing & Partnerships | 5% | 0–3% at TGE + 9–12M linear |
| Voting / Gauge Incentives | 5% | 36M linear |

### Supply Conservation Rule

- **Emission Ledger**: EmissionManager maintains the "Community Emission Reserve." Each time RewardDistributor mints esPaimon, the reserve is simultaneously deducted; if the reserve reaches zero, minting fails immediately.
- **One-for-One Mechanism**: When users claim, an equivalent amount of esPaimon is first burned, then an equal amount of PAIMON is released from the Emission/Treasury inventory. This ensures the actual circulating supply remains ≤ 10B.
- **Boost Restriction**: Only "vested and converted to PAIMON/locked into veNFT shares" contribute multipliers; es still in vesting are excluded from any Boost calculations.
- **Governance Position Settlement**: Governance positions are created only by **locking PAIMON into vePAIMON**. esPAIMON must **vest into PAIMON first**. Supply accounting remains one-for-one and prevents double counting.

---

## 2. Emission and Budget Allocation

### Three-Phase Emission Curve

| Phase | Cycle | Weekly Base Emission | Decline | Stage Total |
|-------|-------|---------------------|---------|-------------|
| Phase A: Launch | Week 1-12 | 37.5M | 0% | 450M |
| Phase B: Growth | Week 13-248 | First Week 55.584M | 1.5%/week exponential decay to 4.327M | ≈ 8.55B |
| Phase C: Tail Phase | Week 249-352 | 4.327M | 0% | 450M |

Phase B decay is solidified on-chain with 236 lookup tables, `EmissionManager.getBaseBudget(week)` O(1).

### Demand-Linked Adjustment

Metric Recommendations:
- **Prime TVL Increment** (Week-over-Week)
- **DEX Depth/Volume** (TWAP volume, depth, and slippage for PPT/USDC pools)
- **Premium/Discount Deviation** D_t = |P_mkt/NAV - 1| (for risk control reduction, not boosting)

Adjustment Strategy:
- Growth/Cold Start Phase: Benchmark Emission × 1.00
- Insufficient Liquidity (High Slippage/Low Depth): Allocate more budget toward DEX/LP incentives
- Deviation persistently nears protection band: Reduce incentive release speed

### Three-Channel Budget Allocation

| Channel | Default Allocation | Sink | Description |
|---------|-------------------|------|-------------|
| Prime Growth Incentive | 40% | Prime/Tranche Related Gauge | Skewed toward risk-taking activities (e.g., jPPT staking) |
| DEX/LP Incentives | 40% | Gauge for PPT/USDC Pools | Guide secondary liquidity depth and price discovery |
| Ecosystem / Gauge Incentives | 20% | Treasury / Incentive Budget | Launchpad partnerships, external incentives |

Unused budget is returned to the Treasury via `reclaimUnused(week)` after the cooling-off period.

---

## 3. Reward Distribution and Vesting

### Multi-Tier Vesting (User Selected)

| Tier | Vesting Period | Reward Coefficient | Early Exit Penalty |
|------|----------------|-------------------|-------------------|
| Fast | 90 days | 0.6× | 50% unvested portion forfeited |
| Standard | 365 days | 1.0× | 50% unvested portion forfeited |
| Loyal | 540 days | 1.2× | 60% unvested portion forfeited |

### Example Calculation (baseReward = 100)

| Tier | Community Emission Quota | User's Actual Total Claim | Remarks |
|------|-------------------------|--------------------------|---------|
| Fast | 100 × 0.6 = 60 | 60 | Short-term release, discount factor |
| Standard | 100 × 1.0 = 100 | 100 | Benchmark Tier |
| Loyal | 100 × 1.2 = 120 | 120 | Long-term incentive, multiplier |

### Boost Mechanism

#### Boost Inputs
- **Base**: baseline multiplier granted to all eligible positions
- **Activity / Stake Boost**: derived from qualifying activity (LP, jPPT staking, etc.)
- **Governance / ve Boost**: derived from vePAIMON voting power

#### Eligibility Rules
- User holds an active qualifying position
- User's vePAIMON voting power is non-zero
- User complies with transfer cool-down / anti-flipping rules

#### Key Points
- **esPAIMON does not carry governance rights** - it cannot vote
- Governance participation requires **PAIMON locked into vePAIMON**
- Boost multiplier has a soft cap (e.g., 1.5×)

---

## 4. Governance and Incentives

### vePAIMON Rules

- Lock-up periods: 1 week to 4 years
- Voting power decays linearly toward unlock
- NFTs are transferable with 48-hour cooling-off for voting/claiming

### Gauge KPI Constraints

| Condition | Action |
|-----------|--------|
| Score < 60 for 1 week | maxWeight reduced by 30% |
| Score < 60 for 2 weeks | maxWeight further reduced to 40% |
| Score ≥ 70 for 2 weeks | Weight recovery |

### Nitro External Incentives

- External projects `propose()` and require VE voting approval
- Incentive assets locked for 4 weeks
- Distributed to eligible voters/LPs from preceding two weeks
- Early termination requires governance approval

---

## 5. Distribution Security

### Security Measures

1. **Multi-signature**: `submitRoot(newRoot, signatures)` requires 3/5 EIP-712 signatures
2. **Cooling-off period**: Root enters `pending` status for 6 hours
3. **Rollback Window**: Old root claimable for 24 hours, then frozen
4. **Distribution Service**: Generates merkle.json, root.json, and summary.txt

### Flow

```
NewRoot → Pending (6h) → Cancel OR Activate
                              ↓
                        Old root claimable 24h
                              ↓
                        Old root frozen
```

---

## 6. Treasury and Capital Circulation

### Income Recirculation Formula

Monthly `reportIncome(income)`:
- **40%** → Governance Value Capture (Repurchase PAIMON + lock as vePAIMON)
- **40%** → Ecosystem Incentives (external incentive matching)
- **20%** → Operational Reserve (audits, operations, compliance)

### Adaptive Liquidity Weighting

| Condition | Action |
|-----------|--------|
| Insufficient DEX depth | Increase DEX/LP incentive weight |
| Deviation nearing protection band | Reduce short-term release, shift to vesting |
| Normal | Revert to default weights |

### Data Transparency

Weekly automated report publishing:
- Circulating supply (including unvested amounts)
- PPT/NAV/Deviation and DEX Depth/Volume
- Emission Utilization Rate
- Gauge weightings and KPIs
- Treasury asset table

---

## 7. Metrics and Risk Control

### Operational Metrics

| Metric | Target/Threshold | Trigger Action |
|--------|------------------|----------------|
| Prime/DEX Health | 70%–90% | Below 60% → Review incentives; Above 95% → Adjust weighting |
| Unused Budget Return Rate | < 10%/quarter | Review claim processes |
| Loyal Tier Proportion | 20%–35% | < 15% → Discuss adjustment |
| Gauge Performance Score | ≥ 70 | Automatic weight restriction for consecutive low scores |

### Risks and Mitigation

| Risk | Explanation | Mitigation |
|------|-------------|------------|
| Oracle Data Manipulation | Keeper Malicious or Data Delay | Multi-source validation, threshold limiting, emergency pause |
| Spread Arbitrage | Rapid Tier Switching | Weekly Limits, Behavior Monitoring |
| Guard process failure | KPI Not Updated | Backup daemon + manual override + alerts |
| Multi-signature congestion | Delayed Root Activation | Schedule adjustments |
| Treasury repurchase impacts | AMM Liquidity Shortage | TWAP/Batch Execution, Slippage Cap |
