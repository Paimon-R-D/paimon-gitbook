# Paimon Finance

## Compliant On-Chain Access to Alternative Assets

---

## Disclaimer

This document outlines the mechanism design and system architecture of the Paimon Protocol and does not constitute investment advice, an offer to sell securities, or a solicitation of an offer to buy securities. The Protocol involves alternative assets such as Real-World Assets (RWA), pre-IPO equity and private fund interests, whose legal attributes, investor suitability, and information disclosure are subject to the regulations of relevant jurisdictions. All parameters are subject to the on-chain contracts and governance charter.

---

## What is Paimon Finance?

Paimon Finance is a BSC-deployed protocol that lets DeFi users hold and trade exposure to assets that historically only existed inside traditional finance — money-market funds, pre-IPO equity SPVs, private credit. We do this through **two live product lines** that share a common issuance and reputation infrastructure.

### Two Live Products

| Product | What it is | Who it serves | How users hold it |
|---------|------------|---------------|--------------------|
| **PP — Paimon Prime Vault** | An ERC-4626 RWA fund holding a tiered portfolio (cash → money market → long-duration RWA) | DeFi users wanting diversified yield exposure with predictable redemption rules | `PP` ERC-20 share token |
| **Pre-IPO SPV Tokens** | Tokenization of single-issuer private equity SPVs (today: SpaceX SPV) using a two-token EIP-3643 + Shadow ERC-20 design | DeFi users wanting targeted single-asset exposure without going through KYC, plus institutional holders who want full settlement rights | Retail: `xSPCX` (free transfer)<br>Institutional: `pSPCX` (KYC-gated security token) |

Both products are issued through a shared **Launchpad** (4-Layer points-gated drop) and accumulate a shared **on-chain reputation** (Points + Badges).

### Shared Infrastructure

- **Launchpad** — `LaunchpadDrop` V4 with 4-Layer pricing, points-and-USDT double gate, refund window, and Batch / MerkleClaim settlement modes
- **Points System** — `PointsHubV2` aggregates points from `StakingModule` (PPT staking, 1×–2× boost), `LPStakingModule`, and future modules
- **Soulbound Badges** — `PaimonBadge` mints non-transferable achievement NFTs per drop participation
- **Compliance Layer** — `EIP3643Token` + `ShadowERC20` + `TokenBridge` + `KYCAggregator`, currently used for pSPCX/xSPCX, designed to scale to additional pre-IPO assets

## Mainnet Footprint

| Contract | BSC Mainnet address |
|----------|---------------------|
| `PPT` (Prime Vault) | `0x8505c32631034A7cE8800239c08547e0434EdaD9` |
| `RedemptionManager` | `0xd614a6fe8C35aC9af4F59cd14849877179cDCdB9` |
| `AssetController` | `0x6F5170956132588E9b2844478f1fF1B387573A3D` |
| `RedemptionVoucher` | `0x73F42b0D657785fE844e3BF486Fe1e15fFE13514` |
| `CashPlusAdapter` | `0xf3a17a5362b6f5b2bCB1AE0C0DE86b70e1ae4a53` |
| `pSPCX` (EIP-3643) | `0x6DC9a487bF8Fd047e41AB336003AE6e4FE602646` |
| `xSPCX` (Shadow) | `0x05353Dabf163Fb2fec87f9e0f00f94Eae4AC1631` |
| `TokenBridge` | `0xE4Eeba287494e694DFf63d7723B0A046506C8910` |
| `LaunchpadDrop` V4 | `0xea088Af719F3238982823fa5eE1C1FaCb2E0e231` |
| `LaunchpadSettlement` | `0x9F7eCde85815f3B616C25a54d181F8766c869a90` |
| `PaimonTreasury` | `0xD9312A3fa2ad5cBEA2C2a36124c72f30025AcAC7` |
| `PaimonBadge` | `0x48e9a6846D9722599621aF8A6af0F23b0dB8184a` |
| `PointsHubV2` | `0x748560eaCcd4C01FC29b3B5b72d3b8C85B2B5017` |
| `StakingModule` | `0x80D9b50f4f1ECdd30CD61E400bf8B9b74eC8795f` |
| `KYCAggregator` | `0xF8DfE25C8C565e057420716F0d2b2905eF8f5227` |
| `SimpleKYCProvider` | `0x2f5503A0D0F9Aba32B82AFbD7395D2096c7FaA2d` |

## What's *Not* Live (Phase 2 Roadmap)

To avoid confusion: documents such as the Tranche Vault (sPP/jPP), the PAIMON token, vePAIMON governance, esPAIMON vesting, the Paimon DEX, and the automated Protection Band describe **future designs**, not deployed code. Sections covering these features carry a "Phase 2 Concept" banner. Refer to the [Roadmap](roadmap.md) for sequencing.

## Target Users

- DeFi users seeking diversified RWA yield via PP
- DeFi users seeking single-asset pre-IPO exposure via xSPCX
- Institutional holders that need settled, KYC-clean positions in pSPCX
- Asset issuers seeking a compliant tokenization + distribution venue

---

## Quick Navigation

- [Executive Summary](overview/README.md)
- [System Architecture](architecture/README.md)
- [Paimon Prime Vault (PP)](architecture/prime-vault.md) — ERC-4626 RWA fund
- [Pre-IPO SPV Tokens (pSPCX / xSPCX)](products/pre-ipo-spv.md) — SpaceX SPV tokenization
- [Launchpad: 4-Layer Drop](architecture/launchpad.md)
- [EIP-3643 Compliance](architecture/eip3643-compliance.md)
- [Points and Badges](products/points-and-badge.md)
- [Risk Disclosure](risk-and-transparency/README.md)

---

## Contact & Resources

- Website: [paimon.finance](https://paimon.finance)
- X (Twitter): [@Paimon_Finance](https://x.com/Paimon_Finance)
- GitHub: [Paimon-R-D](https://github.com/Paimon-R-D)
- Telegram: [paimon_rwa](https://t.me/paimon_rwa)
