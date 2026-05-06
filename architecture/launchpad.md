# Launchpad: 4-Layer Drop with Points-Gated Allocation

The Paimon Launchpad is the issuance venue for Paimon-curated alternative assets. It is **live on BSC mainnet** and has executed at least one production drop (xSPCX SpaceX SPV, V4 contract, April 2026).

| Module | Mainnet address |
|--------|-----------------|
| `LaunchpadDrop` (V4.2.2) | `0xea088Af719F3238982823fa5eE1C1FaCb2E0e231` |
| `LaunchpadSettlement` | `0x9F7eCde85815f3B616C25a54d181F8766c869a90` |
| `PaimonTreasury` (launchpad) | `0xD9312A3fa2ad5cBEA2C2a36124c72f30025AcAC7` |
| `PaimonBadge` | `0x48e9a6846D9722599621aF8A6af0F23b0dB8184a` |
| `PointsHubV2` | `0x748560eaCcd4C01FC29b3B5b72d3b8C85B2B5017` |
| Settlement currency | USDT BSC `0x55d398326f99059fF775485246999027B3197955` |

## Drop Lifecycle

A Drop progresses through eight on-chain phases, defined in `DropPhase` (`ILaunchpadDrop.sol:8-17`):

```
Created ──► QuestActive ──► Snapshot ──► Commitment ──► LayerSale ──► Settled ──► Finalized
                                                                          │
                                                                          └──► Cancelled (any phase)
```

| Phase | What happens | Triggered by |
|-------|--------------|--------------|
| `Created` | Drop config + 4 layers configured; RWA token escrowed; metadata linked | `createDrop` (KEEPER) |
| `QuestActive` | Pre-drop quest period (if any) — users earn points via tasks | `advancePhase` |
| `Snapshot` | Merkle snapshot of points balances taken; users below the per-layer threshold are filtered out | `setSnapshot` (KEEPER, off-chain merkle) |
| `Commitment` | Users call `commit(layerIdx, usdtAmount)` — locks points + escrows USDT | `advancePhase` |
| `LayerSale` | Layer-by-layer settlement: keeper calls `settleLayerBatch` (Batch mode) or sets `settlementRoot` (MerkleClaim mode); users `claimAllocation` | `advancePhase` |
| `Settled` | Refund window opens; over-committed USDT returned via `requestRefund`; per-layer `closeRefundWindow` ends the window | `advancePhase` |
| `Finalized` | All layers closed, USDT split between Treasury and vePool, leftover RWA returned to issuer, badges minted via `finalizeUserPoints` | `finalizeSettlement` |

## Layer Structure

Every drop has **exactly four layers** (`LAYER_COUNT = 4`, `LaunchpadDrop.sol:47`). Each layer is independently configured via the `LayerConfig` struct (`ILaunchpadDrop.sol:37-46`):

| Field | Meaning |
|-------|---------|
| `priceDiscount` | Discount in basis points off `basePrice` (e.g. `6032` = 60.32 % off) |
| `minUsdtRequired` | Minimum USDT a wallet must commit to participate in this layer (`0` = no floor) |
| `pointsPerUsd` | Points consumed per 1 USDT committed (`1e18` precision; e.g. `1.5e18` = 1.5 points per $1) |
| `allocationBps` | Layer's share of the total raise in basis points (the four layers must sum to `10000`) |
| `perWalletCap` | Maximum USDT a single wallet can commit in this layer (`0` = no cap) |
| `saleStartTime` / `saleEndTime` | Layer-specific sale window (layers can overlap or stagger) |
| `mode` | `SettlementMode.Batch` (keeper allocates on-chain, gas-bounded `MAX_BATCH_SIZE = 200`) or `SettlementMode.MerkleClaim` (off-chain merkle, user claims with proof) |

The points-and-USDT double-gate is what differentiates a Paimon drop from a plain "first-come-first-served" sale: lower layers offer steeper discounts but require more points, so users who consistently engage with the protocol (staking, social tasks, holding) get priority access to the best prices.

## Worked Example — xSPCX V4 (April 2026)

This is a real drop that executed on mainnet, with parameters as committed via the V4 contract.

| Metric | Value |
|--------|-------|
| Asset | `xSPCX` Paimon Tradable SpaceX SPV Token |
| Total supply for sale | 30,600 xSPCX |
| Total raise target | $200,000 USDT |
| Base price | $6.93 |
| Settlement mode | `MerkleClaim` |
| Refund window | 1 minute (operational, configurable) |
| Commitment window | 4/13 → 4/20 (7 days) |
| Sale window | 4/20 21:00 → 23:00 EDT (2 hours) |

### Layer pricing

| Layer | Public name | Discount | Effective price | Min commit | Per-wallet cap | Points/USD | Implicit points floor |
|-------|-------------|----------|-----------------|------------|----------------|-----------|----------------------|
| L0 — Pioneer | The Founders | 60.32 % | $2.75 | $2,000 | $20,000 | 1.5 | 3,000 pts |
| L1 — Builder | The Core | 33.91 % | $4.58 | $500 | $10,000 | 1.0 | 500 pts |
| L2 — Explorer | Early Movers | 7.65 % | $6.40 | $100 | $5,000 | 0.5 | 50 pts |
| L3 — Open | Community | 0 % | $6.93 | — | $2,000 | 0 | 0 |

Points required to fully fill a layer's per-wallet cap = `cap × pointsPerUsd`. So a user who wants to max out the Pioneer layer ($20,000 cap) needs **30,000 points**, while the Open layer is freely available to anyone.

## Fee Routing

Per drop, the platform takes a configurable issuance fee (`issuanceFeeBps`, default 100 bps = 1 %), split between the protocol Treasury and the vePool:

| Default split | Bucket | Purpose |
|---------------|--------|---------|
| 70 % of fee | `PaimonTreasury` | Protocol operations, audit, compliance reserves |
| 30 % of fee | `vePool` | Reserved for future vePAIMON / governance distribution |
| Project revenue (raise minus fee) | Issuer Treasury | Settles to the issuer per drop config |
| Unsold RWA | Returned to issuer | Reduces dilution of secondary holders |

All routing is recorded in `DropTreasuryRecord` (`ILaunchpadDrop.sol:84-89`) and re-derivable from `LaunchpadDrop` events.

## Soulbound Badges

Successful commitment in a drop mints a non-transferable `PaimonBadge` (`PaimonBadge.sol:208-211`):

| Enum | Badge name | Earned by |
|------|------------|-----------|
| `SpacePioneer` | Space Pioneer | Participating in a SpaceX-themed drop (e.g. xSPCX) |
| `AIBeliever` | AI Believer | Participating in an AI-themed drop (e.g. future Anthropic SPV) |
| `DualHolder` | Dual Holder | Holding badges across multiple categories |
| `TopReferrer` | Top Referrer | Driving qualified referrals into a drop |
| `QuestMaster` | Quest Master | Completing all pre-drop quests |

Badges are mapped per `(user, BadgeType)` and each combination can only be minted once (`userBadges[user][type] == false → mintable`). They serve as on-chain proof of participation and unlock multipliers in the points system.

## Refund Window

After a layer settles in MerkleClaim mode, users who committed but did not receive their full requested allocation can call `requestRefund`. Once the operator calls `closeRefundWindow` for a layer, no further refunds can be claimed. The refund window duration is global (`refundWindowDuration`, default 3 days; can be reduced operationally — for the xSPCX V4 it was set to 1 minute since the entire sale was a 2-hour event).

## Operator Flow

For a fund manager running a drop end-to-end through the admin dashboard:

1. `PUT /config/refund-window {duration_seconds}` — set refund window
2. `POST /admin/launchpad/drops` — create drop (auto-approves RWA token transfer + on-chain `createDrop`)
3. `POST /metadata/{metadataId}/link/{dropId}` — attach project info / image / description
4. **"Launch Drop"** button → `Created → QuestActive`
5. Wait for commitment window to close
6. **"Advance Phase"** → `LayerSale`; for each layer, `setSettlementRoot(layerIdx, root)` if in MerkleClaim mode
7. Users `claimAllocation` with merkle proof
8. **"Advance Phase"** → `Settled`
9. `closeRefundWindow(layerIdx)` × 4 → `finalizeSettlement(dropId)`

## What This Replaces

A previous draft of this document described a "Hard Gate Review + Soft Scoring + Candidate Pool" governance flow for asset onboarding. **That mechanism is not implemented in any contract.** Asset curation today is performed off-chain by the Paimon team prior to drop creation; on-chain enforcement is limited to the drop / layer / settlement mechanics described above.

The on-chain points-and-USDT double-gate is what enforces "fair access" — it does not screen *which* assets get listed (that is a curatorial decision), but it does control *who* gets early access at favourable prices.
