# Risk Disclosure

## Known Risk Categories

| Risk Type | Description | Mitigation Measures |
|-----------|-------------|---------------------|
| **Valuation Risk** | Outdated or conflicting NAV input data | Multi-source verification, audit cycle constraints, outlier exclusion |
| **Liquidity Risk** | Redemption surges exceeding budgeted capacity | Layered liquidity, three-channel redemptions, protective buffer mechanism |
| **Market Manipulation** | PPT pool depth may be insufficient, making prices susceptible to manipulation | TWAP sampling, buffer trigger, NAV disregarding short-term prices |
| **Governance Attacks** | Vote-buying leading to poor-quality assets being listed | Layered governance, hard thresholds non-voting coverage, timelock |
| **Smart Contract Risks** | Code vulnerabilities leading to fund losses | Multi-round audits, bug bounties, phased rollout |
| **Underlying Asset Risks** | RWA/Private Equity/Fund Credit/Market Risk | Launchpad hard thresholds, continuous disclosure, diversified allocation |

## Contingency Planning for Extreme Scenarios

### Scenario: Mass Redemption Wave

1. **T+0 budget depletion** → Automatic suspension of emergency redemptions
2. **T+7 budget exhausted** → New redemptions enter queue
3. **Queue depth disclosed** → Market-driven pricing of premiums/discounts
4. **Risk control multi-signature intervention** → Evaluate whether to initiate emergency response

### Scenario: Significant NAV Decline

1. **Underlying asset impairment** → NAV downward adjustment
2. **PPT market price may react prematurely** → Discount widens
3. **Protection band triggered** → Suspension of T+0 trading
4. **Ongoing disclosure** → Market receives updated information
5. **Asset disposal completed** → NAV stabilizes → Normal operations resume

## Emergency Reserve Fund

| Parameters | Configuration |
|------------|---------------|
| Funding Sources | Agreement fee deductions + Initial reserves |
| Target Size | 3-5% of Prime TVL |
| Trigger Conditions | Weekly NAV decline >10% or liquidity event |
| Authorization Usage | Risk Control Multi-Signature (3/5) |
| Disclosure Requirements | Weekly public disclosure of balances and transaction records |

## Risk Control Flow

```
Normal Operations
       │
       ▼
┌──────────────────┐
│ Monitor Metrics  │
│ - NAV deviation  │
│ - Liquidity usage│
│ - Queue depth    │
└────────┬─────────┘
         │
    ┌────┴────┐
    │ Alert?  │
    └────┬────┘
         │
    Yes  │  No
    ┌────┴────┬────────┐
    ▼         ▼        │
┌────────┐ Continue    │
│ Assess │ Normal      │
│ Level  │             │
└───┬────┘             │
    │                  │
┌───┴───┬──────┐      │
▼       ▼      ▼      │
Low   Medium  High    │
│       │      │      │
▼       ▼      ▼      │
Warning Partial Full  │
Only    Pause  Pause  │
        │      │      │
        ▼      ▼      │
      Review Emergency │
      Metrics Response │
        │      │      │
        └──────┴──────┘
               │
               ▼
         Resolution
               │
               ▼
         Return to
         Normal ────────┘
```

## Valuation Risk Deep Dive

### NAV Data Sources

| Source Type | Update Frequency | Weight |
|-------------|------------------|--------|
| External Audit | Quarterly | High |
| Custodian Report | Monthly | Medium |
| Issuer Disclosure | Weekly minimum | Medium |
| On-chain Oracle | Real-time | Low (signal only) |

### Conflict Resolution

When sources disagree:
1. Flag the conflict publicly
2. Use most conservative value
3. Escalate to risk committee
4. Publish resolution rationale

## Liquidity Risk Deep Dive

### Budget Stress Testing

| Scenario | L1 Draw | L2 Draw | Queue Activation |
|----------|---------|---------|------------------|
| Normal | <50% | <30% | No |
| Moderate Stress | 50-80% | 30-60% | Possible |
| Severe Stress | >80% | >60% | Yes |
| Crisis | Depleted | >80% | Full queue mode |

### Recovery Timeline

| Phase | Duration | Actions |
|-------|----------|---------|
| Immediate | 0-24h | Suspend T+0, assess situation |
| Short-term | 1-7d | Process T+7 queue, communicate clearly |
| Medium-term | 1-4w | Asset liquidation if needed |
| Recovery | 4-12w | Restore normal operations |

## Governance Attack Vectors

### Known Attack Types

| Attack | Description | Mitigation |
|--------|-------------|------------|
| Vote Buying | Purchasing votes to pass harmful proposals | Timelock, hard threshold protection |
| Flash Loan Governance | Borrow tokens, vote, return | Snapshot-based voting, lock requirements |
| Sybil Attack | Many fake accounts | Minimum stake requirements |
| Collusion | Coordinated voting by insiders | Transparent voting, community monitoring |

### Protected Parameters

These parameters **cannot** be changed by governance voting alone:

- Launchpad hard thresholds (custody, audit, disclosure)
- Protection band emergency triggers
- Supply cap (10B PAIMON)
- Core safety mechanisms

## Smart Contract Security

### Audit Status

| Component | Auditor | Status |
|-----------|---------|--------|
| Prime Vault | TBD | Planned |
| Tranche Vault | TBD | Planned |
| vePAIMON | TBD | Planned |
| Emission System | TBD | Planned |

### Bug Bounty Program

| Severity | Reward Range |
|----------|--------------|
| Critical | $50K - $500K |
| High | $10K - $50K |
| Medium | $2K - $10K |
| Low | $500 - $2K |

## User Responsibility

Users should understand:

1. **PPT is not a stablecoin** - NAV fluctuates with underlying assets
2. **Instant liquidity is not guaranteed** - Three-channel system has constraints
3. **Junior tranche (jPPT) has leverage risk** - Can lose entire principal
4. **Governance decisions affect returns** - Participate or accept outcomes
5. **Smart contract risk exists** - Despite audits, bugs are possible
