# Glossary

## Token Terms

| Term | Definition |
|------|------------|
| **PP** | The on-chain ERC-20 symbol of the Paimon Prime Vault share token. The underlying contract is named `PPT` (Paimon Prime Token) — `PP` is what wallets and explorers display. ERC-4626 share representing a proportional claim on Vault NAV |
| **PPT** | Source-code / audit name for the contract that issues PP shares (`src/ppt/PPT.sol`). Used interchangeably with PP when discussing the implementation |
| **pSPCX** | Paimon Permissioned SpaceX SPV Token. EIP-3643 security token, KYC-gated transfer. Tokenizes a SpaceX SPV interest |
| **xSPCX** | Paimon Tradable SpaceX SPV Token. Plain ERC-20 shadow of pSPCX, freely transferable, mint/burn restricted to `TokenBridge` |
| **PaimonBadge** | Soulbound (non-transferable) ERC-721 NFT minted on Drop participation. Five badge types: Space Pioneer, AI Believer, Dual Holder, Top Referrer, Quest Master |
| **CASH+** | External RWA token issued by CashPlus (price ~$107). Held by Prime Vault as a Tier-2 asset via `CashPlusAdapter` |

## Phase-2 Token Concepts (Not Deployed)

| Term | Definition |
|------|------------|
| `sPP` / `jPP` | Senior / Junior tranches of a future Tranche Vault. **No contract deployed**, no token issued |
| `PAIMON` | Future governance + utility ERC-20. **Not yet issued**. The mainnet contracts named `PaimonTreasury` and `PaimonBadge` are infrastructure, not the PAIMON token |
| `esPAIMON` | Escrowed PAIMON, design only |
| `vePAIMON` | Vote-escrowed PAIMON, design only |

## Vault and Asset Terms

| Term | Definition |
|------|------------|
| **Prime Vault** | The ERC-4626 vault that issues PP and holds the tiered RWA portfolio |
| **NAV** | Net Asset Value per PP share, computed as `(grossAssets - liabilities - withdrawableFees) / effectiveSupply` |
| **Effective Supply** | `totalSupply(PP) - totalLockedShares` — supply excluding shares pending redemption |
| **Locked Shares** | PP shares marked locked because their owner has filed an active redemption request; excluded from NAV math |
| **Gross Value** | Total fair value of underlying assets before deducting redemption liabilities and fees |
| **L1 (`TIER_1_CASH`)** | Cash / instant-liquidity tier of the Vault portfolio |
| **L2 (`TIER_2_MMF`)** | Money-market-fund tier (e.g. CashPlus) |
| **L3 (`TIER_3_HYD`)** | "High-yield distribution" tier — long-duration RWA. Note: this is a **portfolio category label**, not a token symbol |

## Redemption Terms

| Term | Definition |
|------|------------|
| **Standard Channel** | `RedemptionChannel.STANDARD` — T+7 settlement, 0.5 % fee |
| **Emergency Channel** | `RedemptionChannel.EMERGENCY` — instant settlement, 1.5 % total fee, capped by `emergencyQuota` |
| **Scheduled Channel** | `RedemptionChannel.SCHEDULED` — reserved for windowed batches, not currently routed |
| **Standard Quota** | `totalLiquidity × standardQuotaRatio` (default 7000 bps = 70 %) |
| **Emergency Quota** | `emergencyQuota` storage variable on `PPT`, refreshed by KEEPER based on liquidity conditions |
| **Approval Threshold** | Limit above which redemption requires KEEPER approval. Standard: 50,000 PP or 20 % of standard quota. Emergency: 30,000 PP or 20 % of emergency quota |
| **Redemption Voucher** | ERC-721 issued by `RedemptionVoucher` (`0x73F42b0D…3514`) when a redemption's expected settlement extends beyond the standard window. Transferable claim on the pending redemption |

## Pre-IPO / Compliance Terms

| Term | Definition |
|------|------------|
| **EIP-3643** | Ethereum standard for permissioned (regulated) tokens. Adds KYC-gated transfer, freeze, partial freeze, forced transfer, agent role |
| **Shadow ERC-20** | A plain ERC-20 contract that mirrors a permissioned token; mint/burn restricted to a single Bridge contract; used to expose KYC-restricted assets to DeFi without modifying integrating protocols |
| **TokenBridge** | The hub contract that holds the invariant `totalSupply(shadow) = locked(security) × ratio`. Manages N (security, shadow) pairs |
| **KYCAggregator** | `IIdentityRegistry` implementation that ORs the verification results of one or more underlying providers. Used by both EIP3643Token transfers and TokenBridge redeems |
| **SimpleKYCProvider** | A whitelist-style KYC provider with operator-managed verification status. One of the providers feeding into KYCAggregator |
| **Forced Transfer** | EIP-3643 agent-only function that bypasses normal `_msgSender()` allowance checks. Used by `TokenBridge.deposit` to escrow security tokens, and by ops multisig for compliance-mandated movements |
| **Partial Freeze** | EIP-3643 mechanism to lock a portion of a user's balance (`_frozenTokens[user]`) while leaving the rest transferable. Used for vesting schedules or compliance escrow |

## Launchpad Terms

| Term | Definition |
|------|------------|
| **Drop** | A single token-issuance event managed by `LaunchpadDrop`. Has an asset (`rwaToken`), a target raise (`totalUsdtTarget`), 4 layers, a commitment window, a sale window, and a refund window |
| **DropPhase** | Drop lifecycle state: `Created → QuestActive → Snapshot → Commitment → LayerSale → Settled → Finalized` (or `Cancelled`) |
| **Layer** | One of four pricing tiers within a drop. Each has independent `priceDiscount`, `pointsPerUsd`, `allocationBps`, `minUsdtRequired`, `perWalletCap`, sale window, and `SettlementMode` |
| **SettlementMode** | `Batch` (keeper allocates on-chain in batches up to `MAX_BATCH_SIZE = 200`) or `MerkleClaim` (off-chain merkle root, user pulls via proof) |
| **Snapshot** | Merkle-rooted commitment of points balances at the start of a drop, defining who qualifies for which layer |
| **Refund Window** | Period during `Settled` phase when users with un-allocated commitments can `requestRefund` |
| **PointsPerUsd** | The points-to-USDT exchange rate for a given layer. Higher value = stricter points gate at that layer |
| **AllocationBps** | The layer's share of the total raise, in basis points. The four layers must sum to `BPS_BASE = 10000` |

## Points and Reputation

| Term | Definition |
|------|------------|
| **PointsHubV2** | Aggregator that sums points balances across all registered modules. Holds the live total and tracks per-user already-claimed amounts |
| **REWARD_ROLE** | Permission on `PointsHubV2` allowing a contract to credit a user's points balance. Held by source modules (StakingModule, etc.) |
| **DEDUCTOR_ROLE** | Permission on `PointsHubV2` allowing a contract to debit a user's points balance. Held by sink contracts (LaunchpadDrop, PointsRedemption) |
| **Boost** | Multiplier applied to points accrual for locked stakes. Linear from 1.02× (7-day lock) to 2.00× (365-day lock) in `StakingModule`. Subject to `MAX_EXTRA_BOOST = 10000` (= +1.00×) |
| **Soulbound** | NFT property: non-transferable. Applied to `PaimonBadge` — once minted to a wallet, it stays with that wallet permanently |

## Operational Terms

| Term | Definition |
|------|------------|
| **KEEPER** | Service-account role (`KEEPER_ROLE`) that signs on-chain operational transactions: approve/reject redemption, refresh emergency quota, advance drop phases, settle layers, configure assets |
| **REBALANCER** | Service-account role for asset purchase / redeem / liquidation operations. Distinct from KEEPER per multisig-key separation |
| **RBAC** | Off-chain role-based access control (SUPER_ADMIN / FUND_MANAGER / OPERATOR / AUDITOR / ANALYST) gating which humans can trigger which on-chain operations |
| **Multisig** | Multiple-signer wallet holding `ADMIN_ROLE` / `DEFAULT_ADMIN_ROLE` on critical contracts |
| **Timelock** | Delay-imposed wallet holding `UPGRADER_ROLE` on UUPS proxies |
| **UUPS** | Universal Upgradeable Proxy Standard (EIP-1822). Used for upgrading PPT, RedemptionManager, AssetController, EIP3643Token, ShadowERC20, TokenBridge, LaunchpadDrop, etc. |

## Asset Terms

| Term | Definition |
|------|------------|
| **RWA** | Real World Assets — tokenized physical or financial assets backed by off-chain custody / legal structure |
| **SPV** | Special Purpose Vehicle — a legal entity that holds a single asset (e.g. SpaceX equity) on behalf of its investors. Tokenization wraps an SPV interest into pSPCX |
| **Pre-IPO** | Equity in a private company before public listing (e.g. SpaceX). Distinct from RWA in that it is single-name rather than diversified |

## Abbreviations

| Abbreviation | Full Form |
|--------------|-----------|
| AMM | Automated Market Maker |
| APY | Annual Percentage Yield |
| BPS | Basis Points (1 bps = 0.01 %) |
| DEX | Decentralized Exchange |
| ERC-4626 | Tokenized Vault Standard |
| EIP-3643 | Permissioned Token Standard |
| KYC | Know Your Customer |
| LP | Liquidity Provider |
| NAV | Net Asset Value |
| RBAC | Role-Based Access Control |
| RWA | Real World Asset |
| SPV | Special Purpose Vehicle |
| TVL | Total Value Locked |
| UUPS | Universal Upgradeable Proxy Standard |
