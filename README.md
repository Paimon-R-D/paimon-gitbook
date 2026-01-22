# Paimon Finance

## DeFi-Native Liquidity Protocol for RWA

---

## Disclaimer

This document outlines the mechanism design and system architecture of the Paimon Protocol and does not constitute investment advice, an offer to sell securities, or a solicitation of an offer to buy securities. The Protocol involves alternative assets such as Real-World Assets (RWA), private equity, and private funds, whose legal attributes, investor suitability, and information disclosure are subject to the regulations of relevant jurisdictions. All parameters are subject to the on-chain contracts and governance charter.

---

## What is Paimon Finance?

**Core Problem**: Tokenized alternative assets (RWAs, private equity, private funds) face a structural dilemma—a fundamental mismatch between on-chain liquidity and the off-chain asset realization cycle. Existing solutions either sacrifice liquidity for compliance or security for instant exit, offering no compromise.

**Paimon's Solution**: Constructing a three-tier closed-loop architecture that deeply integrates real returns from alternative assets with native DeFi liquidity mechanisms:

| Tier | Features | Core Outputs |
|------|----------|--------------|
| **Launchpad** | Compliance-driven issuance and access screening | Audited On-Chain Assets |
| **Prime Vault** | ERC-4626 Asset Aggregation and NAV Management | PP Share Token |
| **DEX + Vote** | Secondary Market Liquidity and Governance Incentives | Price Discovery + Yield Distribution |

## Key Innovations

- **Yield Tiering (Tranche)**: PP holders may choose Senior (sPP, fixed 4% yield) or Junior (jPP, risk-bearing for leveraged returns), catering to diverse risk preferences
- **vePAIMON Governance Model**: PAIMON may be locked into vePAIMON to obtain time-weighted governance power for gauge voting, protocol parameter signaling, and value-return mechanisms
- **Tiered Liquidity Budget**: L1 (Instant)/L2 (T+7)/L3 (Quarterly), aligning with real asset liquidation cycles
- **Two-Channel Redemption Mechanism**: T+0 (Penalized)/T+7 (Standard) with approval workflow, balancing liquidity demand and system security
- **Discount/Premium Protection Band**: Automatic suspension of emergency redemptions at ±15% deviation to prevent run feedback loops

## Target Users

- DeFi-native institutions seeking alternative asset exposure
- Liquidity providers pursuing real yields
- Issuers aiming to tokenize assets

---

## Quick Navigation

- [Executive Summary](overview/README.md) - High-level protocol overview
- [System Architecture](architecture/README.md) - Technical architecture details
- [Tranche Vault](products/README.md) - sPP/jPP yield stratification
- [Token Economics](governance/tokenomics.md) - PAIMON tokenomics
- [Risk Disclosure](risk-and-transparency/README.md) - Known risks and mitigations

---

## Contact & Resources

- Website: [paimon.finance](https://paimon.finance)
- X (Twitter): [@Paimon_Finance](https://x.com/Paimon_Finance)
- GitHub: [Paimon-R-D](https://github.com/Paimon-R-D)
- Telegram: [paimon_rwa](https://t.me/paimon_rwa)
