# Executive Summary

## Core Problem

Tokenized alternative assets — RWAs, pre-IPO equity, private fund interests — face a structural dilemma: a fundamental mismatch between on-chain liquidity and the off-chain asset realization cycle. Existing solutions either sacrifice liquidity for compliance (the asset is on-chain but secondary trading is dead) or sacrifice security for instant exit (no real legal claim under stress).

## Paimon's Approach

Rather than building one product that tries to satisfy every user simultaneously, Paimon ships **two product lines** that pick the right point on the compliance / liquidity spectrum for their target audience, and connects them through shared issuance and reputation infrastructure.

### Live Product 1: Paimon Prime Vault (PP)

An **ERC-4626 RWA fund** that aggregates user deposits into a tiered portfolio:

- **Tier 1 — Cash**: instant liquidity (USDT and equivalents)
- **Tier 2 — Money Market**: short-duration yield assets (currently CashPlus / CASH+)
- **Tier 3 — Long-Duration RWA**: quarterly+ liquidity assets

Holders receive `PP` shares, with NAV computed from gross assets minus liabilities and fees. Redemption is two-channel:

- **Standard (T+7, 0.5 % fee)** — aligned with Tier-2 settlement cycle
- **Emergency (T+0, 1.5 % total fee)** — capped by a KEEPER-managed `emergencyQuota` to prevent runs

Large redemptions (above 50 K standard / 30 K emergency, or 20 % of the relevant quota) require KEEPER approval before locking shares.

### Live Product 2: Pre-IPO SPV Tokens (pSPCX / xSPCX)

A **two-token compliance design** that tokenizes single-issuer alternative assets:

- **`pSPCX`** — EIP-3643 permissioned token, KYC-gated, the legal "security wrapper"
- **`xSPCX`** — plain ERC-20, freely transferable, the DeFi-friendly mirror

The two are linked through a `TokenBridge` that holds the invariant `totalSupply(xSPCX) = lockedAmount(pSPCX) × ratio` (currently 1:10). KYC users can hold either; non-KYC users can hold and trade xSPCX freely but cannot redeem back into pSPCX.

The first asset on this rail is a **SpaceX SPV** — issued April 2026 as the xSPCX V4 drop ($200 K raise, 30,600 tokens, 4-layer pricing from $2.75 to $6.93). The architecture is designed to scale to additional pre-IPO names without redeploying core infrastructure.

### Shared Infrastructure

| Module | What it does |
|--------|--------------|
| **Launchpad** | 4-Layer Drop with points-and-USDT double gate, refund window, Batch / MerkleClaim settlement |
| **Points** | `PointsHubV2` aggregates non-transferable reputation from PPT staking and LP staking; spent at Drop commit time |
| **Badges** | `PaimonBadge` mints Soulbound NFTs per drop participation (Space Pioneer, AI Believer, Dual Holder, Top Referrer, Quest Master) |
| **Compliance** | `KYCAggregator` + `SimpleKYCProvider` — currently used for institutional pSPCX onboarding; can fan out to additional KYC providers without contract changes |
| **Treasury** | `PaimonTreasury` collects platform fees and routes redemption payouts |

## Design Principles

1. **Liquidity is a budget, not a right.** Instant exit is a finite resource and has a cost. Both PP's Emergency channel and pSPCX-side primary settlement enforce this through fees and quotas.
2. **Compliance is a two-token problem.** Trying to make one token both KYC-gated and DeFi-composable forces a bad compromise. The pSPCX/xSPCX split lets the two requirements live in their proper homes.
3. **Reputation, not emission.** Until a governance token launches, points + badges drive sticky participation. Each user's accrual depends only on their own activity, not on emission rate sharing.
4. **Transparency over promise.** Quotas, NAV, redemption queue depth, drop commitments and settlements are all on-chain or backed by indexed events.

## What This Document Is *Not*

Several sections of this gitbook describe Phase-2 designs that are **not deployed**: the Tranche Vault (sPP/jPP), PAIMON / vePAIMON / esPAIMON tokenomics, the Paimon DEX, the automated Protection Band, the Gauge / Bribe market. These chapters carry a "Phase 2 Concept" banner. They reflect the team's intended sequencing but should not be read as live functionality. See the [Roadmap](../roadmap.md) for ordering.

## Target Users

| User Type | Today's product | Value proposition |
|-----------|----------------|-------------------|
| DeFi users seeking RWA yield | PP (Prime Vault) | Diversified portfolio with documented redemption rules |
| DeFi users seeking pre-IPO exposure | xSPCX | Free secondary transfer; SPV-backed exposure without KYC |
| Institutional / accredited investors | pSPCX | KYC-clean, settlement-ready security token |
| Asset issuers | Launchpad Drop | Compliant tokenization + 4-Layer fair-launch distribution |
| Long-term participants | Points + Badges | Reputation-driven access to future drops |
