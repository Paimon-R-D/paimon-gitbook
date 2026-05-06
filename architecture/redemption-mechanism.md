# Redemption Mechanism: Two-Channel Design with Approval

{% hint style="info" %}
**Source of truth**: `src/ppt/RedemptionManager.sol`, `src/ppt/PPTTypes.sol` — every parameter in this document is verifiable against these contracts.
{% endhint %}

## Design Objectives

The redemption mechanism must balance three competing objectives:

1. **User Experience**: Provide reasonable exit options
2. **System Security**: Preventing runs that deplete liquidity
3. **Fairness**: Transparent approval process for large redemptions

## Two-Channel System

### Channel One: Emergency Redemption (T+0)

| Parameter | Value | Source |
|-----------|-------|--------|
| Channel enum | `RedemptionChannel.EMERGENCY` | `PPTTypes.sol:57-61` |
| Settlement Time | Instant (`EMERGENCY_REDEMPTION_DELAY = 0`) | `PPTTypes.sol:18` |
| Base Fee | 0.5 % (`baseRedemptionFeeBps = 50`) | `RedemptionManager.sol:265` |
| Penalty Fee | 1.0 % (`emergencyPenaltyFeeBps = 100`) | `RedemptionManager.sol:266` |
| **Total Fee** | **1.5 %** | base + penalty |
| Budget Constraint | `emergencyQuota` (KEEPER-refreshed) | `PPT.sol:77` |
| Approval Threshold | `emergencyApprovalAmount = 30,000` PP **OR** `emergencyApprovalQuotaRatio = 2000` bps (20 %) of current `emergencyQuota` | `RedemptionManager.sol:77,80,271-272,854-859` |

**Design Logic**: Curb runs by imposing penalties rather than outright bans. Users willing to pay the 1.5 % fee gain immediate exit rights, subject to the dynamic emergency quota and the approval threshold.

### Channel Two: Standard Redemption (T+7)

| Parameter | Value | Source |
|-----------|-------|--------|
| Channel enum | `RedemptionChannel.STANDARD` | `PPTTypes.sol:57-61` |
| Settlement Time | 7 days (`STANDARD_REDEMPTION_DELAY = 7 days`) | `PPTTypes.sol:17` |
| Fee | 0.5 % (`baseRedemptionFeeBps = 50`) | `RedemptionManager.sol:265` |
| Budget Constraint | `totalLiquidity × standardQuotaRatio` (default 7000 bps = 70 %) | `PPT.sol:39,163,370` |
| Approval Threshold | `standardApprovalAmount = 50,000` PP **OR** `standardApprovalQuotaRatio = 2000` bps (20 %) of dynamic standard quota | `RedemptionManager.sol:71,74,269-270,841-846` |

**Design Logic**: Aligns with the liquidation cycles of L2 (money-market) assets, giving ordinary users a predictable 7-day exit while preserving runway for the L3 sleeve.

### Channel Three: Scheduled (Reserved)

`RedemptionChannel.SCHEDULED` is defined in `PPTTypes.sol` for large windowed batches but is **not currently routed** — `windowId` is fixed at `0` in active code paths. Reserved for future quarterly window functionality.

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

When a redemption request's expected settlement extends beyond the standard window, the request flips its `hasVoucher` flag and an ERC-721 voucher is minted by `RedemptionVoucher` (`0x73F42b0D657785fE844e3BF486Fe1e15fFE13514`):

| Feature | Description |
|---------|-------------|
| **Trigger** | Expected settlement time extends past the configured window. The hard ceiling is `DEFAULT_MAX_SETTLEMENT_DELAY = 30 days` (`PPTTypes.sol:19`); voucher issuance kicks in earlier when the underlying asset settlement is known to require waiting |
| **Token Standard** | ERC-721 (transferable) |
| **Contents** | `requestId`, net redeem amount, expected settlement timestamp |
| **Purpose** | Lets a user transfer or sell their pending claim if they cannot wait for settlement |
| **Settle path** | `POST /api/v1/redemptions/{id}/settle-with-voucher` — voucher holder claims the underlying once settlement actually occurs |

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

The on-chain state machine has **five** states, defined in the `RedemptionStatus` enum at `PPTTypes.sol:49-55`:

| Status | Description | Reachable from | Next states |
|--------|-------------|----------------|-------------|
| `PENDING_APPROVAL` | Request exceeded an approval threshold; awaiting KEEPER decision. Shares marked as `pendingApprovalShares` (do **not** affect NAV yet) | created via `requestStandardRedemption` / `requestEmergencyRedemption` when above threshold | `APPROVED`, `CANCELLED` (if KEEPER rejects) |
| `PENDING` | Below threshold OR auto-approved → shares locked, redemption liability added to vault, waiting for settlement time | direct creation (small) | `SETTLED` |
| `APPROVED` | KEEPER approved a previously `PENDING_APPROVAL` request → shares locked, settlement timer started | `PENDING_APPROVAL` via `approveRedemption()` | `SETTLED`, `CANCELLED` (admin cancellation path) |
| `SETTLED` | Underlying transferred to receiver, fee booked, request complete | `PENDING`, `APPROVED` | (terminal) |
| `CANCELLED` | Either user cancellation before approval, or KEEPER rejection of a `PENDING_APPROVAL` request | `PENDING_APPROVAL`, `APPROVED` | (terminal) |

{% hint style="warning" %}
**No separate `CLAIMABLE` or `REJECTED` state in the contract.**
- "Ready to claim" is implied by `(status == PENDING \|\| status == APPROVED) && block.timestamp >= settlementTime` — settlement transitions the status directly to `SETTLED`.
- Rejection of a large request sets status to `CANCELLED` (see `rejectRedemption()` at `RedemptionManager.sol:596-622`); the distinction "user cancel vs keeper reject" is preserved via the rejection event + reason string, not via a separate status value.
{% endhint %}
