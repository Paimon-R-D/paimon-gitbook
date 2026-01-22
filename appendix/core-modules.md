# Core Modules

## Module Overview

| Module | Function |
|--------|----------|
| **Prime Vault** | ERC-4626 standard asset aggregation, NAV calculation, redemption state machine |
| **Tranche Vault** | sPPT/jPPT Yield Stratification, Epoch Settlement, Priority Repayment Logic |
| **Gauge System** | jPPT Staking, esPAIMON Emission, Boost Calculation |
| **vePAIMON** | Token Locking, Voting Weight Calculation, Protocol Fee Sharing |
| **Protection Band** | Deviation Monitoring, T+0 Suspension Trigger, Recovery Condition Check |
| **Redemption Queue** | Multi-channel redemptions, priority sequencing, batch settlement |

---

## Prime Vault

### Purpose
Central asset management vault implementing ERC-4626 standard.

### Key Functions

| Function | Description |
|----------|-------------|
| `deposit(assets, receiver)` | Deposit underlying assets, receive PPT shares |
| `withdraw(assets, receiver, owner)` | Withdraw assets by specifying amount |
| `redeem(shares, receiver, owner)` | Redeem PPT shares for underlying assets |
| `updateNAV(newValue)` | Update net asset value (oracle/admin) |

### State Variables

| Variable | Type | Description |
|----------|------|-------------|
| `totalAssets` | uint256 | Total value of underlying assets |
| `currentNAV` | uint256 | Current net asset value per share |
| `budgetL1` | uint256 | Available instant liquidity |
| `budgetL2` | uint256 | Available standard liquidity |

### Events

```solidity
event Deposit(address indexed sender, address indexed owner, uint256 assets, uint256 shares);
event Withdraw(address indexed sender, address indexed receiver, address indexed owner, uint256 assets, uint256 shares);
event NAVUpdated(uint256 oldNAV, uint256 newNAV);
```

---

## Tranche Vault

### Purpose
Split PPT yield into senior (fixed) and junior (variable) tranches.

### Key Functions

| Function | Description |
|----------|-------------|
| `depositSenior(pptAmount)` | Deposit PPT to receive sPPT |
| `depositJunior(pptAmount)` | Deposit PPT to receive jPPT |
| `redeemSenior(shares)` | Redeem sPPT for PPT (priority) |
| `redeemJunior(shares)` | Redeem jPPT for PPT (subordinate) |
| `settleEpoch()` | Distribute yield at epoch end |

### State Variables

| Variable | Type | Description |
|----------|------|-------------|
| `seniorRate` | uint256 | Fixed senior yield rate (e.g., 4%) |
| `currentEpoch` | uint256 | Current epoch number |
| `juniorRatio` | uint256 | Current junior/total ratio |
| `minJuniorRatio` | uint256 | Minimum allowed junior ratio |

### Events

```solidity
event EpochSettled(uint256 indexed epoch, int256 totalYield, uint256 seniorYield, int256 juniorYield);
event SeniorDeposit(address indexed user, uint256 pptAmount, uint256 shares);
event JuniorDeposit(address indexed user, uint256 pptAmount, uint256 shares);
```

---

## Gauge System

### Purpose
Manage staking incentives and emission distribution.

### Key Functions

| Function | Description |
|----------|-------------|
| `stake(amount)` | Stake jPPT into gauge |
| `unstake(amount)` | Remove staked jPPT |
| `claim()` | Claim earned esPAIMON |
| `updateEmission()` | Update emission rate based on votes |

### State Variables

| Variable | Type | Description |
|----------|------|-------------|
| `totalStaked` | uint256 | Total jPPT staked |
| `emissionRate` | uint256 | Current esPAIMON per second |
| `boostMultiplier` | mapping | User boost multipliers |

### Events

```solidity
event Staked(address indexed user, uint256 amount);
event Unstaked(address indexed user, uint256 amount);
event RewardClaimed(address indexed user, uint256 amount);
```

---

## vePAIMON

### Purpose
Vote-escrow mechanism for governance participation.

### Key Functions

| Function | Description |
|----------|-------------|
| `lock(amount, duration)` | Lock PAIMON for voting power |
| `increase(amount)` | Add more PAIMON to existing lock |
| `extend(newDuration)` | Extend lock duration |
| `withdraw()` | Withdraw after lock expires |
| `vote(gaugeIds, weights)` | Allocate voting power to gauges |

### State Variables

| Variable | Type | Description |
|----------|------|-------------|
| `locked` | mapping | User lock amounts and end times |
| `totalVotingPower` | uint256 | Total system voting power |
| `gaugeVotes` | mapping | Votes allocated to each gauge |

### Events

```solidity
event Locked(address indexed user, uint256 amount, uint256 duration, uint256 tokenId);
event VoteCast(address indexed user, uint256[] gaugeIds, uint256[] weights);
event Withdrawn(address indexed user, uint256 amount);
```

---

## Protection Band

### Purpose
Monitor price deviation and trigger protective actions.

### Key Functions

| Function | Description |
|----------|-------------|
| `checkDeviation()` | Calculate current deviation D_t |
| `triggerProtection()` | Activate protection mode |
| `checkRecovery()` | Check if conditions allow recovery |
| `resumeNormal()` | Return to normal operations |

### State Variables

| Variable | Type | Description |
|----------|------|-------------|
| `bandWidth` | uint256 | Protection band width (default 15%) |
| `isProtected` | bool | Current protection status |
| `twapWindow` | uint256 | TWAP calculation window |

### Events

```solidity
event ProtectionTriggered(uint256 deviation, uint256 timestamp);
event ProtectionRecovered(uint256 deviation, uint256 timestamp);
event BandWidthUpdated(uint256 oldWidth, uint256 newWidth);
```

---

## Redemption Queue

### Purpose
Manage multi-channel redemption requests and settlements.

### Key Functions

| Function | Description |
|----------|-------------|
| `requestRedemption(shares, channel)` | Submit redemption request |
| `processQueue()` | Process pending redemptions |
| `getPosition(user)` | Get user's queue position |
| `estimateWait(shares)` | Estimate wait time |

### State Variables

| Variable | Type | Description |
|----------|------|-------------|
| `queue` | Request[] | Pending redemption requests |
| `safetyThreshold` | uint256 | Threshold for queue activation |
| `batchSize` | uint256 | Maximum batch settlement size |

### Events

```solidity
event RedemptionRequested(address indexed user, uint256 shares, uint8 channel, uint256 queuePosition);
event RedemptionSettled(address indexed user, uint256 shares, uint256 assets, uint256 fee);
event QueueProcessed(uint256 requestsProcessed, uint256 assetsDistributed);
```

---

## Module Interactions

```
┌─────────────────────────────────────────────────────────────────┐
│                         User Actions                             │
└────────────────────────────┬────────────────────────────────────┘
                             │
         ┌───────────────────┼───────────────────┐
         ↓                   ↓                   ↓
   ┌──────────┐       ┌──────────┐       ┌──────────┐
   │  Prime   │       │ Tranche  │       │  Gauge   │
   │  Vault   │←─────→│  Vault   │←─────→│  System  │
   └────┬─────┘       └────┬─────┘       └────┬─────┘
        │                  │                  │
        ↓                  ↓                  ↓
   ┌──────────┐       ┌──────────┐       ┌──────────┐
   │Redemption│       │Protection│       │ vePAIMON │
   │  Queue   │←─────→│   Band   │←─────→│          │
   └──────────┘       └──────────┘       └──────────┘
```

## Contract Addresses

> **Note**: Contract addresses will be published upon mainnet deployment. All deployed contracts will undergo security audits and be verified on block explorers. Check our [official documentation](https://docs.paimon.finance) for the latest deployment information.
