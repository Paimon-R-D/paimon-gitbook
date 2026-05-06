# Points and Badges

Paimon does not (currently) have a governance token. The protocol's incentive plumbing is built instead on **points** — a non-transferable, on-chain reputation balance — and **badges** — Soulbound NFTs minted on participation milestones. Together they govern who gets early / discounted access to Launchpad drops.

This is **live on BSC mainnet** and was the gating mechanism for the xSPCX V4 drop.

## Architecture

```
┌──────────────────────────────────────────────────────────────────┐
│                          PointsHubV2                              │
│         (single aggregator — total balance per user)              │
│                                                                   │
│   getTotalPoints(user)  ─►  Σ across modules                      │
│   getClaimablePoints(user) ─►  Σ minus already-deducted           │
└─────────────┬──────────────────────────────┬──────────────────────┘
              │                              │
              │  REWARD_ROLE                 │  DEDUCTOR_ROLE
              │  (writes to)                 │  (deducts on use)
              │                              │
   ┌──────────┴──────────┐         ┌─────────┴─────────┐
   ▼                     ▼         ▼                   ▼
┌─────────────┐  ┌──────────────┐  ┌────────────┐  ┌──────────────┐
│ StakingModule│ │ LPStaking    │  │ Launchpad  │  │ Points       │
│  (PPT stake) │ │ Module       │  │ Drop       │  │ Redemption   │
│              │ │              │  │            │  │              │
│ • Flexible   │ │ (Pancake LP) │  │  commit()  │  │  consume     │
│ • Locked     │ │              │  │  consumes  │  │  points for  │
│ • Boost 1-2x │ │              │  │  points    │  │  rewards     │
└──────────────┘  └──────────────┘  └────────────┘  └──────────────┘
```

| Module | Mainnet address | Role |
|--------|-----------------|------|
| `PointsHubV2` | `0x748560eaCcd4C01FC29b3B5b72d3b8C85B2B5017` | Hub — aggregates balances across modules |
| `StakingModule` (PPT staking) | `0x80D9b50f4f1ECdd30CD61E400bf8B9b74eC8795f` | Source — issues points for staked PPT |
| `LaunchpadDrop` | `0xea088Af719F3238982823fa5eE1C1FaCb2E0e231` | Sink — deducts points on `commit()` |
| `PointsRedemption` | (deployed) | Sink — points → off-chain rewards |
| `PaimonBadge` | `0x48e9a6846D9722599621aF8A6af0F23b0dB8184a` | Achievement NFT (Soulbound, ERC-721) |

## Point Sources

Points flow into a user's balance from registered modules. Each module implements `IPointsModule` and the hub holds `REWARD_ROLE` permission to credit balances on the user's behalf.

### Source 1: PPT Staking (live)

Contract: `StakingModule v2.3` (`src/point/StakingModule.sol`).

Users stake **PPT** (the Paimon Prime fund share) to earn points over time. The accrual formula is:

```
points = stakedAmount × boost × pointsRatePerSecond × elapsed / BOOST_BASE
```

| Stake type | Lock window | Boost | Notes |
|------------|-------------|-------|-------|
| `Flexible` | None | 1.0× | Withdraw anytime |
| `Locked`   | 7 → 365 days | Linear 1.02× → 2.00× | Early unlock = 50 % penalty on the unvested portion |

Constants worth knowing:

| Constant | Value | Source |
|----------|-------|--------|
| `MIN_STAKE_AMOUNT` | 10 PPT | `StakingModule.sol:56` |
| `MAX_STAKES_PER_USER` | 100 active stakes | `StakingModule.sol:48` |
| `MIN_LOCK_DURATION` | 7 days | `StakingModule.sol:45` |
| `MAX_LOCK_DURATION` | 365 days | `StakingModule.sol:46` |
| `EARLY_UNLOCK_PENALTY_BPS` | 5000 (50 %) | `StakingModule.sol:47` |
| `MAX_EXTRA_BOOST` | 10000 (= +1.00× → max 2.00×) | `StakingModule.sol:44` |

The "credit card points" model is the design pivot here: each user's accrual depends only on their own stake size, lock duration, and time held — late entrants do **not** dilute earlier participants the way emission-rate models do.

### Source 2: LP Staking (live)

Contract: `LPStakingModule` (`src/point/LPStakingModule.sol`) — registered with PointsHubV2 alongside PPT staking, allowing LP positions on supported AMM pools to earn points on the same hub balance.

### Source 3: Future modules

`PointsHubV2` is intentionally pluggable. Adding a new earning mechanism (e.g. per-drop quests, referral milestones, governance voting once vePAIMON ships) only requires deploying a new `IPointsModule` implementation and granting it `REWARD_ROLE` on the hub.

## Point Sinks

### Sink 1: Launchpad Drops (live)

This is the primary use of points today. Layer-by-layer, a Drop's `pointsPerUsd` value sets the conversion: committing $1 USDT to a layer with `pointsPerUsd = 1.5` consumes 1.5 points.

Worked numbers from the xSPCX V4 drop:

| Layer | `pointsPerUsd` | Points to enter at min commit | Points to fill per-wallet cap |
|-------|----------------|------------------------------|-----------------------------|
| L0 Pioneer | 1.5 | 3,000 (= $2,000 × 1.5) | 30,000 (= $20,000 × 1.5) |
| L1 Builder | 1.0 | 500 (= $500 × 1.0) | 10,000 (= $10,000 × 1.0) |
| L2 Explorer | 0.5 | 50 (= $100 × 0.5) | 2,500 (= $5,000 × 0.5) |
| L3 Open | 0 | — | — |

Points are deducted at `commit` time via `PointsHubV2.deductPoints(user, amount, reason)` (`PointsHubV2.sol:312-320`). If a layer fails to fill or the user later requests a refund, points are **not** automatically returned — they have already been "spent" to lock in the layer's price.

### Sink 2: Direct Redemption (live)

`PointsRedemption` lets users burn points for off-chain rewards (merch, whitelists, off-platform credits) — operationally configured per campaign.

## Soulbound Badges

Contract: `PaimonBadge` (`src/launchpad/PaimonBadge.sol`).

A `BadgeType` enum (`ILaunchpadDrop.sol:27-33`) defines five categories:

| Enum | Display name | Color | Earned by |
|------|--------------|-------|-----------|
| `SpacePioneer` | Space Pioneer | `#6C3CE1` | Participating in a SpaceX-themed drop (e.g. xSPCX) |
| `AIBeliever` | AI Believer | `#2563EB` | Participating in an AI-themed drop (future Anthropic / similar) |
| `DualHolder` | Dual Holder | `#059669` | Holding badges across multiple categories |
| `TopReferrer` | Top Referrer | `#D97706` | Driving qualified referrals into a drop |
| `QuestMaster` | Quest Master | — | Completing all pre-drop quests |

Each `(user, badgeType)` combination can be minted **at most once** — checked via `userBadges[user][type]` flag in `PaimonBadge.sol:48`. Badges are non-transferable (Soulbound), meaning they accumulate as a wallet's verifiable participation history.

Badges are minted in batches by KEEPER at drop finalization (`batchMint(recipients, types, dropId)` in `PaimonBadge.sol:108`) and emit `BadgeMinted(user, type, dropId, mintTime)` events for off-chain indexing.

### What badges unlock

Today, badges serve as **on-chain provenance**: a future drop's snapshot can require holding `SpacePioneer` for early-layer access, or boost effective `pointsPerUsd` for badge holders. The badge → boost mapping is configured per drop via the snapshot Merkle root rather than hard-coded — this lets each issuer tailor their access policy.

## Putting It Together — A Holder's Lifecycle

```
1. Buy PPT (deposit USDT into Prime Vault)         ──► Hold PP
                                                       │
2. Stake PPT into StakingModule (Flexible/Locked)  ──► Earn points (linear, with boost)
                                                       │
3. Quest period of next drop                       ──► Earn additional points (future module)
                                                       │
4. Drop commitment window opens                    ──► Snapshot lists who qualifies for which layer
                                                       │
5. commit(L0, $2,000)  consumes 3,000 points       ──► Lock USDT + lose points
                                                       │
6. Drop settles: receive xSPCX at $2.75            ──► Hold xSPCX
                                                       │
7. Drop finalizes: SpacePioneer badge minted       ──► Permanent on-chain record
                                                       │
8. Hold / trade xSPCX freely; or KYC and redeem    ──► Exit path (or compound into next drop)
```

Steps 1–2 are the "deposit → engage → accumulate reputation" loop. Steps 4–7 are the "spend reputation for early access" loop. The two loops together drive sticky participation without requiring an emission-driven token.
