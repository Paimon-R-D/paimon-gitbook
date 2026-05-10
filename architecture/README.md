# System Architecture

Paimon Finance is composed of **two independent product lines** sitting on a **shared issuance and reputation layer**. The two product lines do not interact at the asset-accounting level — PP NAV does not include xSPCX, and xSPCX redemption does not touch PP — but they share the Launchpad, Points, Badge, Treasury and Compliance infrastructure.

## High-Level Diagram

```
┌──────────────────────────────────────────────────────────────────────────────┐
│                              User-facing Surface                              │
│           Wallet, dApp, Admin Dashboard, External AMM Integrations            │
└──────────────────────────────────────────────────────────────────────────────┘
       │                                          │
       │  Holds PP / claims redemption            │  Holds xSPCX / pSPCX
       │                                          │  KYC users redeem via Bridge
       ▼                                          ▼
┌─────────────────────────────────┐   ┌────────────────────────────────────────┐
│      Product Line A             │   │           Product Line B                │
│      Paimon Prime (PP)          │   │      Pre-IPO SPV Tokens (pSPCX/xSPCX)   │
│                                 │   │                                         │
│  ┌──────────────────────────┐   │   │  ┌─────────────────────────────────┐    │
│  │ PPT Vault (ERC-4626)     │   │   │  │ EIP3643Token (pSPCX)            │    │
│  │  • Deposit / Mint        │   │   │  │  • KYC-gated transfer           │    │
│  │  • NAV per share         │   │   │  │  • Agent: freeze / forced       │    │
│  │  • Locked-share ledger   │   │   │  └────────────┬────────────────────┘    │
│  └──────┬───────────────────┘   │   │               │                         │
│         │                       │   │       TokenBridge                       │
│  ┌──────┴───────────────────┐   │   │       (1:N pSPCX↔xSPCX,                 │
│  │ RedemptionManager        │   │   │        KYC-gated redeem)                │
│  │  • Standard T+7 / 0.5 %  │   │   │               │                         │
│  │  • Emergency T+0 / 1.5 % │   │   │  ┌────────────┴────────────────────┐    │
│  │  • Approval workflow     │   │   │  │ ShadowERC20 (xSPCX)             │    │
│  │  • RedemptionVoucher     │   │   │  │  • Plain ERC-20, free transfer  │    │
│  └──────┬───────────────────┘   │   │  │  • Mint/burn only by Bridge     │    │
│         │                       │   │  └─────────────────────────────────┘    │
│  ┌──────┴───────────────────┐   │   │                                         │
│  │ AssetController          │   │   └─────────────────────────────────────────┘
│  │  • L1 / L2 / L3 tiers    │
│  │  • Adapter integrations  │
│  └──────┬───────────────────┘
│         │
│  ┌──────┴───────────────────┐
│  │ External Adapters        │
│  │  • CashPlusAdapter       │
│  │  • Aave (planned)        │
│  └──────────────────────────┘
└─────────────────────────────────┘
       │                                          │
       │                                          │
       ▼                                          ▼
┌──────────────────────────────────────────────────────────────────────────────┐
│                        Shared Issuance + Reputation Layer                     │
│                                                                              │
│   LaunchpadDrop V4 (4-Layer Drop, points-gated)  ──►  PaimonTreasury          │
│                  │                                                            │
│                  ├─ deducts ─►  PointsHubV2  ◄─── credits ─── StakingModule   │
│                  │                  │                          LPStakingModule │
│                  └─ mints ─────►  PaimonBadge (Soulbound)                     │
│                                                                              │
│              KYCAggregator  ◄──  SimpleKYCProvider (institutional whitelist)  │
└──────────────────────────────────────────────────────────────────────────────┘
       ▲
       │
       │  Off-chain operations
       │
┌──────────────────────────────────────────────────────────────────────────────┐
│   Off-chain operational backend (event-driven)                                │
│   • On-chain event ingestion with redundant transports + gap recovery         │
│   • KEEPER / REBALANCER service accounts for chain ops                        │
│   • Off-chain RBAC for human-triggered operations                             │
│   • Rebalance engine, approval workflow, settlement automation                │
└──────────────────────────────────────────────────────────────────────────────┘
```

## Layer Descriptions

### Product Line A — Paimon Prime (PP)

The original product: an ERC-4626 vault holding a tiered RWA portfolio. Users deposit USDT, receive PP shares, and exit through a two-channel redemption mechanism (Standard T+7 / Emergency T+0 with quota and approval gates). Asset rebalancing across tiers is run by REBALANCER service accounts under a single-active-strategy pattern.

→ [Prime Vault details](prime-vault.md), [Redemption mechanism](redemption-mechanism.md)

### Product Line B — Pre-IPO SPV Tokens

A two-token design: `pSPCX` is the legal/regulated security token (EIP-3643, KYC-gated); `xSPCX` is its DeFi-tradable shadow ERC-20. They are linked exclusively through `TokenBridge` at a fixed 1:10 ratio. The architecture is reusable: adding a new pre-IPO asset (e.g. an Anthropic SPV) only requires deploying a new EIP-3643 + Shadow pair and registering with the existing bridge.

→ [Pre-IPO SPV product](../products/pre-ipo-spv.md), [EIP-3643 architecture](eip3643-compliance.md)

### Shared — Launchpad + Points + Badges

Both product lines distribute their tokens through `LaunchpadDrop V4`, a 4-Layer points-gated sale. Pre-drop activity (PPT staking, LP staking) earns non-transferable points in `PointsHubV2`; points are spent at `commit()` time to unlock cheaper layers. Successful drop participation mints a `PaimonBadge` Soulbound NFT.

→ [Launchpad](launchpad.md), [Points and Badges](../products/points-and-badge.md)

### Compliance Layer

`KYCAggregator` is the identity-registry hub used by `EIP3643Token.transfer` (receiver check) and `TokenBridge.redeem` (sender check). It currently aggregates a single `SimpleKYCProvider` whitelist; the design supports adding additional providers (Sumsub, Onfido, etc.) without contract redeploy.

KYC onboarding is currently **B2B / institutional only**. Retail wallets can hold and trade `xSPCX` freely without going through KYC.

→ [EIP-3643 Compliance](eip3643-compliance.md)

### Off-Chain Backend

An event-driven operational backend runs the off-chain layer:

- On-chain event ingestion with redundant transports + gap recovery
- KEEPER / REBALANCER service accounts that sign on-chain transactions for redemption approvals, asset purchases, drop phase advances, layer settlement, etc.
- Off-chain RBAC gating which human users can trigger which on-chain operations
- Rebalance strategy engine (single active strategy, mutual-exclusion with manual rebalance)
- External-vault settlement automation (status probes + auto-advance state machine)

Critical design rule: **the blockchain is the source of truth**. The backend never holds business state that is not derivable from on-chain events.

## What's Not in This Architecture (Yet)

The following modules are present in the source tree but **not deployed** to mainnet, and therefore not part of the live system:

- `PAIMON` ERC-20 token (governance utility token, not yet minted)
- `VotingEscrow` veNFT (lock-up and voting power)
- `GaugeController` (emission allocation voting)
- `BribeMarketplace` and `RewardDistributor`
- `DEXFactory` / `DEXPair` (Paimon's own AMM)
- `Treasury.sol` RWA-backed CDP minting `HYD` synthetic asset (separate design exploration, not the live `PaimonTreasury`)
- `Tranche Vault` (sPP/jPP) — design only, no contract written

These are documented under their respective Phase-2 sections with explicit banners, and tracked in the [Roadmap](../roadmap.md).
