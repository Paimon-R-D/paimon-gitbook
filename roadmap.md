# Roadmap

## Where We Are Today

**Live on BSC Mainnet** as of this writing:

- ✅ **Paimon Prime Vault (PP)** — ERC-4626 RWA fund with two-channel redemption
- ✅ **Pre-IPO SPV Tokens (pSPCX / xSPCX)** — EIP-3643 + Shadow ERC-20 + TokenBridge, first asset issued: SpaceX SPV via xSPCX V4 drop (April 2026)
- ✅ **Launchpad** — 4-Layer Drop with points-and-USDT double gate, V4.2.2
- ✅ **Compliance Layer** — KYCAggregator + SimpleKYCProvider (institutional onboarding only)
- ✅ **Points + Badges** — PointsHubV2, StakingModule (PPT staking with 1×–2× boost), LPStakingModule, PaimonBadge (Soulbound)
- ✅ **External Adapters** — CashPlusAdapter (live external RWA integration)
- ✅ **Operational Backend** — KEEPER / REBALANCER service accounts, off-chain RBAC, event-driven listener with gap recovery, automated settlement workflows

**Not deployed** (described in this gitbook with a "Phase 2 Concept" banner):

- Tranche Vault (sPP / jPP) — design only, no contract written
- PAIMON token / vePAIMON / esPAIMON — token not yet issued
- Paimon DEX (DEXFactory, DEXPair) — designed, not deployed
- HYD synthetic asset, PSM, RWA-collateral CDP — separate design exploration
- GaugeController / BribeMarketplace / RewardDistributor — designed, not deployed
- Protection Band automation — manual KEEPER intervention only today

## Phases

| Phase | Focus | Key Deliverables | Status |
|-------|-------|------------------|--------|
| **M0** | Paimon Prime | PP Vault, two-channel redemption, NAV / Voucher | ✅ Live |
| **M1** | Asset Onboarding & Launchpad | LaunchpadDrop V4, PointsHubV2, StakingModule, PaimonBadge | ✅ Live |
| **M2** | Pre-IPO Tokenization | EIP-3643 + Shadow ERC-20 + Bridge, KYCAggregator; first issuance: xSPCX SpaceX SPV | ✅ Live |
| **M3** | Multi-Asset Pre-IPO | Add additional pre-IPO assets (Anthropic SPV etc.) on the same EIP-3643 + Bridge architecture; expand KYC providers | 🚧 In Progress |
| **M4** | Retail KYC | Open KYC onboarding to retail wallets so end users can self-serve into pSPCX redemption | Planned |
| **M5** | Tranche Vault | sPP (Senior, fixed yield) / jPP (Junior, leveraged) split on top of PP | Planned (design phase) |
| **M6** | DEX & Liquidity | Paimon DEX deployment with PP / xSPCX trading pairs; LP incentives | Planned |
| **M7** | Token & Governance | PAIMON token issuance, vePAIMON locking, Gauge voting, BribeMarketplace, fee sharing | Planned |
| **M8** | Protection Band Automation | TWAP-based deviation monitoring, automatic T+0 suspension trigger | Planned |
| **M9** | Cross-Chain | Bridge to additional chains, unified PP / xSPCX across networks | Planned |

## Phase Details

### M0: Paimon Prime ✅

Core asset management infrastructure:

- PPT contract (ERC-4626) with NAV / share price calculation
- Two-channel redemption (Standard T+7, Emergency T+0) with KEEPER approval workflow above thresholds
- AssetController + adapters (CashPlus live)
- RedemptionVoucher (ERC-721) for delayed settlements
- CertiK audit completed January 2026

### M1: Launchpad & Reputation ✅

- LaunchpadDrop V4.2.2 with 4-Layer points-gated drops
- PointsHubV2 hub with REWARD_ROLE / DEDUCTOR_ROLE design for pluggable modules
- StakingModule v2.3 (credit-card points model with lock-duration boost)
- LPStakingModule for AMM positions
- PaimonBadge (Soulbound, 5 badge types)
- PointsRedemption for off-chain reward conversion

### M2: Pre-IPO Tokenization ✅

- EIP3643Token implementation (UUPS upgradeable, agent / KYC / freeze / forced-transfer)
- ShadowERC20 (UUPS, mint/burn restricted to bridge)
- TokenBridge hub with N-pair support
- KYCAggregator + SimpleKYCProvider (institutional whitelist)
- First issuance: **xSPCX SpaceX SPV V4 drop, April 13–20 2026** — $200K raise, 30,600 tokens, 4 layers ($2.75 / $4.58 / $6.40 / $6.93), MerkleClaim settlement

### M3: Multi-Asset Pre-IPO 🚧

Goal: add additional pre-IPO assets (e.g. Anthropic SPV) on the existing infrastructure. Adding a new asset requires only:

1. Deploy new `EIP3643Token` proxy (e.g. pPNTC)
2. Deploy new `ShadowERC20` proxy (e.g. xPNTC)
3. `TokenBridge.createPair(newSecurity, newShadow, ratio)`
4. Grant agent / bridge roles

The same KYCAggregator whitelist applies to all assets — no re-onboarding of existing KYC users.

### M4: Retail KYC

Open the KYC pipeline to retail wallets. Today's KYCAggregator + SimpleKYCProvider are deployed but onboarding is institutional-only. M4 brings the user-facing KYC flow online so any wallet can self-serve verification and access pSPCX redemption.

### M5: Tranche Vault

Senior / Junior yield stratification on top of PP. Senior receives a fixed yield (e.g. 4% APY); Junior receives the residual after Senior is paid, providing leveraged exposure but bearing first-loss risk. **Design phase only** — no contract written, no parameters finalized.

### M6: DEX & Liquidity

Deploy Paimon's own AMM (V2-style with 0.25 % fee, 70 / 30 voter / treasury split). Initial pairs: PP/USDT, xSPCX/USDT. Provides protocol-controlled venue for both products' secondary liquidity, plus a fee surface that flows into Treasury / future vePAIMON.

### M7: Token & Governance

Issue PAIMON token (10B max supply, capped). Activate vePAIMON locking, GaugeController for emission allocation across pools, BribeMarketplace for external incentives, RewardDistributor for merkle-root reward distribution. Migration of off-chain RBAC + KEEPER multisig to on-chain governance proposals (with timelock + tiered risk levels).

### M8: Protection Band Automation

Today's "protection band" is a manual KEEPER lever — operators can pause Emergency redemptions if they observe NAV / market price divergence. M8 automates this via TWAP deviation monitoring with automatic suspension at ±15 % threshold.

### M9: Cross-Chain

Bridge PP and xSPCX to additional EVM chains (Ethereum mainnet, Arbitrum, Base) via LayerZero or comparable infrastructure. Maintain unified KYC across chains.

## Risk Factors per Phase

| Phase | Key Risk | Mitigation |
|-------|----------|-----------|
| M0 | Smart-contract bugs in core vault | CertiK audit complete; ongoing monitoring |
| M1 | Launchpad griefing via points spam | StakingModule MIN_STAKE_AMOUNT = 10 PPT; ratio limits per layer |
| M2 | Bridge invariant break | UUPS upgrades behind multisig + timelock; on-chain invariant verification |
| M3 | Operational scale-out (multi-asset) | Hub-and-spoke design — KYC layer reused; per-asset risk isolated |
| M4 | KYC provider failure / data breach | Multi-source aggregation; provider can be swapped without contract redeploy |
| M5 | Tranche pricing model | Extensive simulation, formal verification of yield distribution |
| M6 | LP impermanent loss | Incentive design + protocol-owned liquidity |
| M7 | Token launch volatility | Vesting schedules, gradual emission |
| M8 | TWAP manipulation | Multi-block window, outlier exclusion |
| M9 | Cross-chain bridge exploits | Audit-heavy bridge selection; insurance |
