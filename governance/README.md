# Governance Structure: Tiered Decision-Making

## Design Principles

Governance power requires boundaries. Mixing "incentive allocation" with "risk control hard thresholds" leads to:
- Bribery to list subpar assets
- Misguided decisions under information asymmetry
- Excessively low cost of governance attacks

## Three-Tier Governance Structure

| Risk Level | Decision Content | Implementation Approach |
|------------|------------------|------------------------|
| **Low Risk** | Pool emission weighting, cost allocation ratio | Voting Direct Execution |
| **Medium Risk** | New pool launch/removal, parameter adjustments | Vote Passed + Timelock |
| **High Risk** | Launchpad asset onboarding, major Prime allocation decisions | Hard Threshold Review + Multi-Signature Approval, Voting Serves Only as Soft Authorization |

## Governance Token Mechanics

| Mechanism | Function |
|-----------|----------|
| **vePAIMON** | Lock tokens to gain voting rights; longer lock-up periods yield higher weighting |
| **Gauge Voting** | Determines mining emission allocation across pools |
| **Protocol Fee Sharing** | vePAIMON holders share protocol revenue proportionally to their staking weight |
| **Plugin System** | Delegate voting rights to specific modules or strategies |

## Governance Proposal Workflow

```
                    ┌─────────────┐
                    │ Proposal    │
                    │ Submission  │
                    └──────┬──────┘
                           │
                    ┌──────┴──────┐
                    │ Risk        │
                    │ Assessment  │
                    └──────┬──────┘
                           │
         ┌─────────────────┼─────────────────┐
         ↓                 ↓                 ↓
    ┌─────────┐       ┌─────────┐       ┌─────────┐
    │ Low     │       │ Medium  │       │ High    │
    │ Risk    │       │ Risk    │       │ Risk    │
    └────┬────┘       └────┬────┘       └────┬────┘
         ↓                 ↓                 ↓
    ┌─────────┐       ┌─────────┐       ┌──────────────┐
    │ Vote    │       │ Vote    │       │ Hard-Gate     │
    │         │       │         │       │ Review        │
    └────┬────┘       └────┬────┘       └────┬──────────┘
         ↓                 ↓                 ↓
    ┌─────────┐       ┌─────────┐       ┌──────────────┐
    │ Execute │       │ Timelock│       │ Multisig      │
    │ Directly│       │ Execute │       │ Approval      │
    └─────────┘       └─────────┘       └────┬──────────┘
                                             ↓
                                       ┌─────────┐
                                       │ Execute │
                                       └─────────┘
```

## vePAIMON Details

### Lock Duration

| Lock Period | Voting Power Multiplier |
|-------------|------------------------|
| 1 week | 1x (minimum) |
| 1 year | ~12x |
| 2 years | ~24x |
| 4 years | ~52x (maximum) |

### Power Decay

Voting power decays linearly toward the unlock date. This incentivizes:
- Longer commitments for greater influence
- Continuous re-locking to maintain power
- Alignment of governance with long-term protocol health

### NFT Representation

vePAIMON positions are represented as NFTs (veNFT), enabling:
- Position transfers (with cooldown restrictions)
- Composability with other DeFi protocols
- Clear ownership and delegation

## Gauge System

### How Gauges Work

1. Each incentivized activity (LP pool, jPPT staking, etc.) has a Gauge
2. vePAIMON holders vote to allocate emissions across Gauges
3. Votes are tallied weekly (Epoch-based)
4. Emissions flow to Gauges based on vote share

### KPI Constraints

Gauges are subject to performance requirements:

| Metric | Threshold | Consequence |
|--------|-----------|-------------|
| Performance Score | < 60 for 1 week | 30% weight reduction |
| Performance Score | < 60 for 2 weeks | Further reduction to 40% |
| Recovery | ≥ 70 for 2 weeks | Weight restored |

## External Incentives (Nitro)

Third parties can add incentives to specific Gauges:

1. Project proposes incentive program
2. vePAIMON vote approves
3. Incentive assets locked for 4 weeks
4. Distribution to eligible voters/LPs

```
Start[External project proposes] → Vote[ve vote]
Vote →|Approved| Lock[Lock incentive assets for 4 weeks]
Lock → Dist[Distribute by eligibility]
Vote →|Rejected| End[End]
Dist →|Unused allocation| Return to Project or Treasury
```

## Protected Decisions

The following cannot be changed by governance voting alone:

| Category | Example | Protection Mechanism |
|----------|---------|---------------------|
| Hard Thresholds | Launchpad custody/audit requirements | Immutable or multi-sig only |
| Core Safety | Protection band emergency triggers | Multi-sig override only |
| Supply Cap | 10B PAIMON maximum | Contract immutable |

## Parameter Change Process

### Standard Parameters (Low/Medium Risk)

| Step | Duration | Requirement |
|------|----------|-------------|
| 1. Proposal Submission | - | Minimum 10,000 vePAIMON |
| 2. Discussion Period | 3 days | Community feedback |
| 3. Voting Period | 5 days | Quorum: 4% of total vePAIMON |
| 4. Timelock | 24-72h | Based on risk level |
| 5. Execution | - | Automatic if passed |

### Protected Parameters (High Risk)

| Step | Duration | Requirement |
|------|----------|-------------|
| 1. Proposal Submission | - | Risk Committee or 100,000 vePAIMON |
| 2. Hard-Gate Review | 7 days | Technical and security assessment |
| 3. Multi-sig Approval | - | 3/5 signatures required |
| 4. Extended Timelock | 7 days | Public notice period |
| 5. Execution | - | Multi-sig execution |

### Multi-sig Configuration

- **Signers**: 5 designated key holders (mix of team and community)
- **Threshold**: 3/5 for standard operations, 4/5 for emergency actions
- **Rotation**: Annual review with community input

For detailed token mechanics, see [Token Economics](tokenomics.md). For the AMP market maker system, see [AMP System](amp-system.md).
