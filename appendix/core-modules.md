# Core Modules

This page lists every contract module that is part of the Paimon protocol design, separated by deployment status.

## ✅ Live on BSC Mainnet

### Product Line A — Paimon Prime (PP)

| Module | Mainnet address | Source |
|--------|-----------------|--------|
| `PPT` (Prime Vault, ERC-4626) | `0x8505c32631034A7cE8800239c08547e0434EdaD9` | `src/ppt/PPT.sol` |
| `RedemptionManager` | (operational, not enumerated) | `src/ppt/RedemptionManager.sol` |
| `AssetController` | (operational, not enumerated) | `src/ppt/AssetController.sol` |
| `RedemptionVoucher` (ERC-721) | (operational, not enumerated) | `src/ppt/RedemptionVoucher.sol` |
| `PaimonOracleAdapter` | (operational, not enumerated) | `src/adapters/PaimonOracleAdapter.sol` |
| `CashPlusAdapter` | (operational, not enumerated) | `src/adapters/CashPlusAdapter.sol` |
| External money-market vault | (third-party, not enumerated) | (third party) |

### Product Line B — Pre-IPO SPV Tokens

| Module | Mainnet address | Source |
|--------|-----------------|--------|
| `pSPCX` (EIP3643Token) | `0x6DC9a487bF8Fd047e41AB336003AE6e4FE602646` | `src/eip3643/EIP3643Token.sol` |
| `xSPCX` (ShadowERC20) | `0x05353Dabf163Fb2fec87f9e0f00f94Eae4AC1631` | `src/eip3643/ShadowERC20.sol` |
| `TokenBridge` | (operational, not enumerated) | `src/eip3643/TokenBridge.sol` |

### Compliance Layer

| Module | Mainnet address | Source |
|--------|-----------------|--------|
| `KYCAggregator` | (operational, not enumerated) | `src/eip3643/KYCAggregator.sol` |
| `SimpleKYCProvider` | (operational, not enumerated) | `src/eip3643/SimpleKYCProvider.sol` |

### Shared Issuance + Reputation Layer

| Module | Mainnet address | Source |
|--------|-----------------|--------|
| `LaunchpadDrop` V4.2.2 | `0xea088Af719F3238982823fa5eE1C1FaCb2E0e231` | `src/launchpad/LaunchpadDrop.sol` |
| `LaunchpadSettlement` | (operational, not enumerated) | `src/launchpad/LaunchpadSettlement.sol` |
| `PaimonTreasury` (launchpad) | (operational, not enumerated) | `src/treasury/PaimonTreasury.sol` |
| `PaimonBadge` (Soulbound ERC-721) | `0x48e9a6846D9722599621aF8A6af0F23b0dB8184a` | `src/launchpad/PaimonBadge.sol` |
| `PointsHubV2` | `0x748560eaCcd4C01FC29b3B5b72d3b8C85B2B5017` | `src/point/PointsHubV2.sol` |
| `StakingModule` (PPT staking) | `0x80D9b50f4f1ECdd30CD61E400bf8B9b74eC8795f` | `src/point/StakingModule.sol` |
| `LPStakingModule` | (deployed) | `src/point/LPStakingModule.sol` |
| `PointsRedemption` | (deployed) | `src/point/PointsRedemption.sol` |

> Modules marked **"operational, not enumerated"** are infrastructure / settlement / compliance contracts managed under multisig + timelock. Their addresses are visible on-chain through interactions with the user-facing tokens above; we do not enumerate them in published documentation.

## 🚧 Phase-2 — Designed, Not Deployed

| Module | Purpose | Status |
|--------|---------|--------|
| `PAIMON` | Governance + utility ERC-20 | Token not yet issued |
| `VotingEscrow` (veNFT) | Lock-up + voting power NFT | Not deployed; design currently locks HYD, intended to lock PAIMON |
| `GaugeController` | Emission allocation voting (7-day epochs, batch voting) | Not deployed |
| `BribeMarketplace` | External incentives on Gauges | Not deployed |
| `RewardDistributor` | Merkle-root reward distribution | Not deployed |
| `DEXFactory` / `DEXPair` | Paimon's own AMM (V2-style with 0.25 % fee, 70 / 30 voter / treasury split) | Not deployed |
| `Treasury.sol` (RWA-backed CDP) | RWA collateral → mint `HYD` synthetic | Not deployed; experimental design |
| `HYD` synthetic asset | Low-volatility synthetic backed by RWA | Not deployed |
| `PSM` (Peg Stability Module) | 1:1 USDC ↔ HYD swap with 0.1 % fee | Not deployed |
| Tranche Vault (`sPP` / `jPP`) | Senior / Junior yield stratification on top of PP | **No contract written** — design only |
| Protection Band automation | TWAP-based deviation monitoring + auto T+0 suspension | Not deployed; manual KEEPER intervention only |

## Module Reference

### `PPT` (Prime Vault) — ERC-4626 Live Vault

ERC-4626 + UUPS upgradeable + AccessControl + Pausable + ReentrancyGuard.

#### Key state

| Variable | Type | Meaning |
|----------|------|---------|
| `assetController` | address | Source of `calculateAssetValue()` for off-vault held assets |
| `redemptionManager` | address | Source of liability data |
| `totalRedemptionLiability` | uint256 | Outstanding pending-redemption obligations (deducted from NAV) |
| `totalLockedShares` | uint256 | Shares locked while pending settlement (excluded from `effectiveSupply`) |
| `withdrawableRedemptionFees` | uint256 | Accrued fees not yet withdrawn (deducted from NAV) |
| `emergencyQuota` | uint256 | KEEPER-managed budget for T+0 redemptions |
| `lockedMintAssets` | uint256 | Recently minted underlying not yet released for redemption |
| `standardQuotaRatio` | uint256 | bps applied to `totalLiquidity` to derive Standard channel quota (default `7000`) |
| `lockedSharesOf[user]` | mapping | Per-user locked-share count |

#### Key functions

| Function | Meaning |
|----------|---------|
| `deposit(assets, receiver)` | ERC-4626 deposit; reverts if `assets < MIN_DEPOSIT (10e18)` |
| `mint(shares, receiver)` | ERC-4626 mint; same minimum |
| `sharePrice()` | NAV per share with virtual offset for inflation-attack protection |
| `totalAssets()` | Gross assets minus liabilities minus fees (ERC-4626 override) |
| `effectiveSupply()` | `totalSupply - totalLockedShares` |
| `grossAssets()` | Cash balance + `assetController.calculateAssetValue()` |
| `refreshEmergencyQuota`, `restoreEmergencyQuota` | KEEPER-only quota management |
| `addRedemptionLiability` / `removeRedemptionLiability` | OPERATOR-only (called by RedemptionManager) |

### `RedemptionManager`

Two-channel redemption with approval workflow and settlement.

| Function | Meaning |
|----------|---------|
| `requestStandardRedemption(shares)` | Open T+7 redemption (auto-approved if below threshold) |
| `requestEmergencyRedemption(shares)` | Open T+0 redemption (1.5 % fee, capped by emergency quota) |
| `approveRedemption(requestId)` | KEEPER approves a `PENDING_APPROVAL` request |
| `rejectRedemption(requestId, reason)` | KEEPER rejects → status becomes `CANCELLED`, emergency quota refunded |
| `claimRedemption(requestId)` | User claims after settlement time |
| `cancelRedemption(requestId)` | User cancels before approval |

| Threshold parameter | Default | Purpose |
|---------------------|---------|---------|
| `baseRedemptionFeeBps` | 50 (0.5 %) | Standard fee |
| `emergencyPenaltyFeeBps` | 100 (1 %) | Added on top of base for emergency channel |
| `standardApprovalAmount` | 50,000 PP | Above this → requires approval |
| `standardApprovalQuotaRatio` | 2000 bps (20 %) | Or above 20 % of dynamic standard quota → requires approval |
| `emergencyApprovalAmount` | 30,000 PP | Above this → requires approval |
| `emergencyApprovalQuotaRatio` | 2000 bps (20 %) | Or above 20 % of emergency quota → requires approval |

### `AssetController`

Manages asset value updates and delayed settlement (e.g. CashPlus subscription).

### `RedemptionVoucher` (ERC-721)

Issued when a redemption's expected settlement extends past the standard window. Contains `requestId`, net amount, expected settlement time. Transferable; the voucher holder calls a public settle endpoint to claim the underlying once settlement actually occurs.

### `EIP3643Token` (pSPCX)

UUPS-upgradeable EIP-3643 implementation. Roles: `DEFAULT_ADMIN_ROLE` / `ADMIN_ROLE` / `UPGRADER_ROLE` / `AGENT_ROLE`. Functions: standard ERC-20 + `freezeAddress`, `freezePartialTokens`, `unfreeze`, `forcedTransfer`, `pause`, `setKYCProvider`. See [EIP-3643 Compliance](../architecture/eip3643-compliance.md).

### `ShadowERC20` (xSPCX)

UUPS-upgradeable plain ERC-20 with single special rule: only `bridge` address can `mint` / `burn`. All other functions standard.

### `TokenBridge`

Hub managing N (security, shadow) pairs. Maintains invariant `totalSupply(shadow) = lockedAmount(security) × ratio` per pair. Functions: `createPair`, `deposit` (security → shadow), `redeem` (shadow → security, KYC-gated).

### `KYCAggregator` + `SimpleKYCProvider`

`KYCAggregator.isVerified(addr)` returns OR of all registered providers. `SimpleKYCProvider` is a whitelist managed by operator-mediated `addToWhitelist` / `removeFromWhitelist` calls.

### `LaunchpadDrop` V4.2.2

4-Layer drop sale with points-and-USDT double gate. Phases: `Created → QuestActive → Snapshot → Commitment → LayerSale → Settled → Finalized` (or `Cancelled`). See [Launchpad](../architecture/launchpad.md).

### `PaimonBadge`

Soulbound ERC-721. Five badge types (`SpacePioneer`, `AIBeliever`, `DualHolder`, `TopReferrer`, `QuestMaster`). Each `(user, type)` mintable once. `batchMint` for drop finalization.

### `PointsHubV2`

Aggregator implementing reward credit (`addReward`, `REWARD_ROLE`) and deduction (`deductPoints`, `DEDUCTOR_ROLE`). Backward-compatible with v1 ABI (`getTotalPoints`, `getClaimablePoints`).

### `StakingModule` v2.3

PPT staking → points. Two stake types (`Flexible` 1.0×, `Locked` 1.02×–2.00×). Constants: `MIN_STAKE_AMOUNT = 10e18`, `MIN_LOCK_DURATION = 7 days`, `MAX_LOCK_DURATION = 365 days`, `EARLY_UNLOCK_PENALTY_BPS = 5000`, `MAX_STAKES_PER_USER = 100`.

## Inter-Module Interactions

```
                  ┌─────────────────────────────────────┐
                  │              User Action            │
                  └──────────────────┬──────────────────┘
                                     │
       ┌─────────────────┬───────────┴───────────┬───────────────────┐
       ▼                 ▼                       ▼                   ▼
┌─────────────┐    ┌──────────────┐      ┌──────────────┐   ┌─────────────┐
│  PPT Vault  │    │   Launchpad  │      │  TokenBridge │   │  Staking    │
│             │    │     Drop     │      │              │   │   Module    │
└──────┬──────┘    └──────┬───────┘      └──────┬───────┘   └──────┬──────┘
       │                  │                     │                  │
       │  redemption      │  deduct points      │  mint/burn       │  add reward
       ▼                  ▼                     ▼                  ▼
┌─────────────┐    ┌──────────────┐      ┌──────────────┐   ┌─────────────┐
│ Redemption  │    │ PointsHubV2  │      │  pSPCX  ◄──► │   │ PointsHubV2 │
│  Manager    │    │              │      │  xSPCX       │   │             │
└──────┬──────┘    └──────────────┘      └──────────────┘   └─────────────┘
       │                  ▲
       │                  │  mint badge
       ▼                  │
┌─────────────┐    ┌──────┴───────┐
│  Asset      │    │ PaimonBadge  │
│  Controller │    │  (Soulbound) │
└─────────────┘    └──────────────┘
```
