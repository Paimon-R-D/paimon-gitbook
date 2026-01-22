# Core Modules

## Module Overview

| Module | Function | Status |
|--------|----------|--------|
| **PPT (Prime Vault)** | ERC-4626 standard asset aggregation, NAV calculation, quota management | ✅ Implemented |
| **RedemptionManager** | Two-channel redemptions (T+0, T+7), approval workflow, settlement | ✅ Implemented |
| **AssetController** | Asset value updates, delayed settlement management | ✅ Implemented |
| **RedemptionVoucher** | ERC-721 NFT for delayed redemption claims | ✅ Implemented |
| **Tranche Vault** | sPPT/jPPT Yield Stratification, Epoch Settlement | 🚧 Planned |
| **Gauge System** | jPPT Staking, esPAIMON Emission, Boost Calculation | 🚧 Planned |
| **vePAIMON** | Token Locking, Voting Weight Calculation, Protocol Fee Sharing | 🚧 Planned |
| **Protection Band** | Automated deviation monitoring, T+0 suspension trigger | 🚧 Planned |

---

## PPT (Prime Vault) ✅

### Purpose
Central asset management vault implementing ERC-4626 standard. Issues PP (Paimon Prime) tokens representing proportional claims on vault assets.

### Key Functions

| Function | Description |
|----------|-------------|
| `deposit(assets, receiver)` | Deposit underlying assets, receive PP shares (min 500 tokens) |
| `redeem(shares, receiver, owner)` | Direct redemption (disabled, use RedemptionManager) |
| `sharePrice()` | Get current NAV per share |
| `totalAssets()` | Get net asset value (gross - liabilities - fees) |
| `effectiveSupply()` | Get supply excluding locked shares |
| `getStandardChannelQuota()` | Get available T+7 redemption quota |
| `getEmergencyChannelQuota()` | Get available T+0 redemption quota |

### State Variables

| Variable | Type | Description |
|----------|------|-------------|
| `grossValue` | uint256 | Total gross value of underlying assets |
| `totalRedemptionLiability` | uint256 | Outstanding redemption obligations |
| `totalLockedShares` | uint256 | Shares locked pending settlement |
| `emergencyQuota` | uint256 | Available T+0 redemption budget |
| `lockedMintAssets` | uint256 | Assets locked during accumulation period |

### Events

```solidity
event Deposit(address indexed sender, address indexed owner, uint256 assets, uint256 shares);
event GrossValueUpdated(uint256 oldValue, uint256 newValue);
event EmergencyQuotaUpdated(uint256 oldQuota, uint256 newQuota);
```

---

## RedemptionManager ✅

### Purpose
Manage two-channel redemption requests with approval workflow and settlement.

### Key Functions

| Function | Description |
|----------|-------------|
| `requestStandardRedemption(shares)` | Request T+7 redemption |
| `requestEmergencyRedemption(shares)` | Request T+0 redemption (1.5% fee) |
| `approveRedemption(requestId)` | Keeper approves large redemption |
| `rejectRedemption(requestId)` | Keeper rejects redemption |
| `claimRedemption(requestId)` | Claim settled redemption |
| `cancelRedemption(requestId)` | Cancel pending request |

### Approval Thresholds

| Channel | Absolute | Ratio |
|---------|----------|-------|
| Standard (T+7) | 50,000 tokens | 20% of quota |
| Emergency (T+0) | 30,000 tokens | 20% of quota |

### State Variables

| Variable | Type | Description |
|----------|------|-------------|
| `requests` | mapping | All redemption requests by ID |
| `baseRedemptionFeeBps` | uint256 | Base fee (default 50 bps) |
| `emergencyPenaltyFeeBps` | uint256 | Emergency penalty (default 100 bps) |
| `standardApprovalThreshold` | ApprovalThreshold | T+7 approval limits |
| `emergencyApprovalThreshold` | ApprovalThreshold | T+0 approval limits |

### Events

```solidity
event RedemptionRequested(uint256 indexed requestId, address indexed user, uint256 shares, uint8 channel);
event RedemptionApproved(uint256 indexed requestId, address indexed approver);
event RedemptionRejected(uint256 indexed requestId, address indexed rejecter);
event RedemptionClaimed(uint256 indexed requestId, address indexed user, uint256 assets, uint256 fee);
event RedemptionCancelled(uint256 indexed requestId, address indexed user);
```

### Request States

```solidity
enum RedemptionStatus {
    NONE,
    PENDING_APPROVAL,  // Awaiting keeper approval
    PENDING,           // Approved, waiting for settlement time
    CLAIMABLE,         // Ready to claim
    SETTLED,           // Claimed
    REJECTED,          // Rejected by keeper
    CANCELLED          // Cancelled by user
}
```

---

## AssetController ✅

### Purpose
Manage asset value updates and delayed settlement for sales/purchases.

### Key Functions

| Function | Description |
|----------|-------------|
| `updateGrossValue(newValue)` | Update total asset value |
| `createDelayedSettlement(type, amount, deadline)` | Create pending asset settlement |
| `executeSettlement(settlementId)` | Execute pending settlement |

### Events

```solidity
event GrossValueUpdated(uint256 oldValue, uint256 newValue);
event DelayedSettlementCreated(uint256 indexed id, uint8 settlementType, uint256 amount);
event DelayedSettlementExecuted(uint256 indexed id);
```

---

## RedemptionVoucher ✅

### Purpose
ERC-721 NFT representing claims on delayed redemptions (issued when settlement exceeds 7 days).

### Key Functions

| Function | Description |
|----------|-------------|
| `mint(to, requestId, netAmount, settlementTime)` | Issue voucher for delayed redemption |
| `burn(tokenId)` | Burn voucher after redemption claimed |
| `getVoucherInfo(tokenId)` | Get voucher details |

### Events

```solidity
event VoucherMinted(uint256 indexed tokenId, address indexed to, uint256 requestId, uint256 netAmount);
event VoucherBurned(uint256 indexed tokenId);
```

---

## Tranche Vault 🚧 (Planned)

### Purpose
Split PPT yield into senior (fixed) and junior (variable) tranches.

> This module is planned for Phase 2 deployment.

### Planned Functions

| Function | Description |
|----------|-------------|
| `depositSenior(pptAmount)` | Deposit PPT to receive sPPT |
| `depositJunior(pptAmount)` | Deposit PPT to receive jPPT |
| `redeemSenior(shares)` | Redeem sPPT for PPT (priority) |
| `redeemJunior(shares)` | Redeem jPPT for PPT (subordinate) |
| `settleEpoch()` | Distribute yield at epoch end |

---

## Gauge System 🚧 (Planned)

### Purpose
Manage staking incentives and emission distribution.

> This module is planned for Phase 2 deployment.

### Planned Functions

| Function | Description |
|----------|-------------|
| `stake(amount)` | Stake jPPT into gauge |
| `unstake(amount)` | Remove staked jPPT |
| `claim()` | Claim earned esPAIMON |
| `updateEmission()` | Update emission rate based on votes |

---

## vePAIMON 🚧 (Planned)

### Purpose
Vote-escrow mechanism for governance participation.

> This module is planned for Phase 2 deployment.

### Planned Functions

| Function | Description |
|----------|-------------|
| `lock(amount, duration)` | Lock PAIMON for voting power |
| `increase(amount)` | Add more PAIMON to existing lock |
| `extend(newDuration)` | Extend lock duration |
| `withdraw()` | Withdraw after lock expires |
| `vote(gaugeIds, weights)` | Allocate voting power to gauges |

---

## Protection Band 🚧 (Planned)

### Purpose
Monitor price deviation and trigger protective actions.

> This module is planned for Phase 2 deployment. Current implementation uses manual keeper controls.

### Planned Functions

| Function | Description |
|----------|-------------|
| `checkDeviation()` | Calculate current deviation D_t |
| `triggerProtection()` | Activate protection mode |
| `checkRecovery()` | Check if conditions allow recovery |
| `resumeNormal()` | Return to normal operations |

---

## Module Interactions (Current Implementation)

```
┌─────────────────────────────────────────────────────────────────┐
│                         User Actions                             │
└────────────────────────────┬────────────────────────────────────┘
                             │
         ┌───────────────────┼───────────────────┐
         ↓                   ↓                   ↓
   ┌──────────┐       ┌──────────────┐    ┌──────────────┐
   │   PPT    │       │ Redemption   │    │    Asset     │
   │ (Vault)  │←─────→│   Manager    │←──→│  Controller  │
   └────┬─────┘       └──────┬───────┘    └──────────────┘
        │                    │
        │                    ↓
        │             ┌──────────────┐
        └────────────→│  Redemption  │
                      │   Voucher    │
                      └──────────────┘
```

## Contract Addresses

> **Note**: Contract addresses will be published upon mainnet deployment. All deployed contracts will undergo security audits and be verified on block explorers. Check our [official documentation](https://docs.paimon.finance) for the latest deployment information.
