# Redemption Mechanism: Two-Channel Design with Approval

## Design Objectives

The redemption mechanism must balance three competing objectives:

1. **User Experience**: Provide reasonable exit options
2. **System Security**: Preventing runs that deplete liquidity
3. **Fairness**: Transparent approval process for large redemptions

## Two-Channel System

### Channel One: Emergency Redemption (T+0)

| Parameters | Settings |
|------------|----------|
| Settlement Time | Instant (same block) |
| Base Fee | 0.5% (50 bps) |
| Penalty Fee | 1.0% (100 bps) |
| **Total Fee** | **1.5%** |
| Budget Constraint | Emergency Quota (L1 Liquidity) |
| Approval Threshold | >30K tokens OR >20% of emergency quota |

**Design Logic**: Curb runs by imposing penalties rather than outright bans. Users willing to pay higher fees gain immediate exit rights, subject to budget caps and approval requirements.

### Channel Two: Standard Redemption (T+7)

| Parameters | Settings |
|------------|----------|
| Settlement Time | T+7 (7 calendar days) |
| Fee | 0.5% (50 bps) |
| Budget Constraint | Standard Quota (L1 + L2 liquidity × 70%) |
| Approval Threshold | >50K tokens OR >20% of standard quota |

**Design Logic**: Aligns with the liquidation cycles of most L2 assets, providing ordinary users with reasonable exit time expectations.

## Approval Mechanism

Large redemptions require keeper approval before processing:

### Approval Workflow

```
┌─────────────────┐
│ Redemption      │
│ Request         │
└──────┬──────────┘
       │
       ↓
┌──────────────────────────────┐
│ Check Amount vs Thresholds   │
└──────────────┬───────────────┘
               │
    ┌──────────┴──────────┐
    ↓                     ↓
┌─────────────┐     ┌─────────────┐
│ Below       │     │ Above       │
│ Threshold   │     │ Threshold   │
└──────┬──────┘     └──────┬──────┘
       ↓                   ↓
┌─────────────┐     ┌─────────────┐
│ Auto-Lock   │     │ PENDING     │
│ Shares      │     │ APPROVAL    │
└──────┬──────┘     └──────┬──────┘
       │                   ↓
       │            ┌─────────────┐
       │            │ Keeper      │
       │            │ Review      │
       │            └──────┬──────┘
       │                   │
       │            ┌──────┴──────┐
       │            ↓             ↓
       │      ┌─────────┐   ┌─────────┐
       │      │ Approve │   │ Reject  │
       │      └────┬────┘   └────┬────┘
       │           ↓             ↓
       ↓      Lock Shares   Return Shares
┌─────────────┐    │
│ Wait Period │←───┘
│ (0 or 7d)   │
└──────┬──────┘
       ↓
┌─────────────┐
│ Settlement  │
│ (Claim)     │
└─────────────┘
```

### Approval Thresholds

| Channel | Absolute Threshold | Ratio Threshold |
|---------|-------------------|-----------------|
| Standard (T+7) | 50,000 tokens | 20% of standard quota |
| Emergency (T+0) | 30,000 tokens | 20% of emergency quota |

**Note**: Redemptions exceeding EITHER threshold require approval.

## Redemption Voucher (NFT)

When redemption settlement is delayed beyond 7 days, an ERC-721 voucher is automatically issued:

| Feature | Description |
|---------|-------------|
| **Trigger** | Settlement delay > 7 days |
| **Token Standard** | ERC-721 (transferable) |
| **Contents** | Request ID, net amount, expected settlement time |
| **Purpose** | Tradeable claim on pending redemption |

This allows users to exit their position by selling the voucher on secondary markets if they cannot wait for settlement.

## Redemption Flow

```
┌─────────────────┐
│ Redemption      │
│ Request         │
└──────┬──────────┘
       │
       ↓
┌──────────────────────────────┐
│ Check Channel & Quota        │
└──────────────┬───────────────┘
               │
    ┌──────────┼──────────┐
    ↓          ↓          ↓
┌────────┐  ┌────────┐  ┌────────────────┐
│Emergency│ │Standard│  │ Insufficient   │
│Quota OK │ │Quota OK│  │ Quota          │
└───┬────┘  └───┬────┘  └────┬───────────┘
    ↓           ↓            ↓
┌────────┐  ┌────────┐  ┌──────────────┐
│T+0     │  │T+7     │  │ Rejected     │
│(1.5%)  │  │(0.5%)  │  │ (Try Later)  │
└────────┘  └────────┘  └──────────────┘
```

## Budget Transparency

All liquidity budgets are **publicly disclosed in real-time**. Users can query:

- Emergency quota utilization
- Standard quota utilization
- Pending approval requests
- Estimated settlement times

**Design rationale**: Transparency is the foundation of trust. Concealing quota status would cause the market to price in worst-case assumptions.

## Fee Structure

| Channel | Fee Type | Rate | Purpose |
|---------|----------|------|---------|
| T+0 | Base + Penalty | 1.5% | Discourage panic exits |
| T+7 | Standard Fee | 0.5% | Cover operational costs |

## Anti-Gaming Measures

### Large Redemption Control
The approval mechanism prevents sudden large withdrawals from destabilizing the system.

### Front-Running Protection
- Approval adds a review step for large requests
- Settlement is processed in order of approval time

### Budget Gaming Prevention
- Quotas are refreshed by keepers based on liquidity conditions
- No "reservation" of future quota capacity
- All quota updates are atomic with redemption execution

## State Transitions

| Status | Description | Next States |
|--------|-------------|-------------|
| `PENDING_APPROVAL` | Large request awaiting keeper approval | `PENDING`, `REJECTED` |
| `PENDING` | Approved, shares locked, waiting for settlement time | `CLAIMABLE` |
| `CLAIMABLE` | Settlement time reached, ready to claim | `SETTLED` |
| `SETTLED` | Assets claimed, request complete | (Terminal) |
| `REJECTED` | Keeper rejected the request | (Terminal) |
| `CANCELLED` | User cancelled before approval | (Terminal) |
