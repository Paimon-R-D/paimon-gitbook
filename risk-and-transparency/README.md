# Risk Disclosure

## Known Risk Categories

| Risk Type | Description | Mitigation Measures |
|-----------|-------------|---------------------|
| **Valuation Risk** | Outdated or conflicting NAV input data | Multi-source verification, audit cycle constraints, outlier exclusion |
| **Liquidity Risk** | Redemption surges exceeding budgeted capacity | Layered liquidity, two-channel redemptions with approval, quota management |
| **Market Manipulation** | PP pool depth may be insufficient, making prices susceptible to manipulation | TWAP sampling, buffer trigger, NAV disregarding short-term prices |
| **Governance Attacks** | Vote-buying leading to poor-quality assets being listed | Layered governance, hard thresholds non-voting coverage, timelock |
| **Smart Contract Risks** | Code vulnerabilities leading to fund losses | Multi-round audits, bug bounties, phased rollout |
| **Underlying Asset Risks** | RWA/Private Equity/Fund Credit/Market Risk | Launchpad hard thresholds, continuous disclosure, diversified allocation |

## Contingency Planning for Extreme Scenarios

### Scenario: Mass Redemption Wave

1. **Emergency quota depletion** → T+0 redemptions rejected until quota refreshed
2. **Standard quota exhausted** → New T+7 redemptions rejected until quota refreshed
3. **Large redemptions** → Require keeper approval before processing
4. **Quota status disclosed** → Market-driven pricing of premiums/discounts
5. **Risk control intervention** → Keepers evaluate whether to refresh quotas or pause

### Scenario: Significant NAV Decline

1. **Underlying asset impairment** → NAV downward adjustment via AssetController
2. **PP market price may react prematurely** → Discount widens
3. **Keeper intervention** → May reduce quotas or pause redemptions (Protection Band automation planned for Phase 2)
4. **Ongoing disclosure** → Market receives updated information
5. **Asset disposal completed** → NAV stabilizes → Quotas restored → Normal operations resume

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

### Risk Level Trigger Thresholds

| Level | Deviation (D_t) | Quota Utilization | Pending Approvals | Response |
|-------|-----------------|-------------------|-------------------|----------|
| **Normal** | < 8% | < 70% | < 10 requests | Continue operations |
| **Low** | 8-12% | 70-85% | 10-20 requests | Warning issued, increased monitoring |
| **Medium** | 12-15% | 85-95% | 20-50 requests | Keeper review, may reduce quotas |
| **High** | > 15% | > 95% | > 50 requests | Emergency review, quotas paused |

### Response Timeline

| Phase | Duration | Actions |
|-------|----------|---------|
| **Detection** | 0-1h | Automated alerts, initial assessment |
| **Immediate** | 1-24h | Risk committee review, protective measures |
| **Short-term** | 1-7d | Root cause analysis, stakeholder communication |
| **Recovery** | 7-30d | Remediation, gradual return to normal |

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

### Quota Stress Testing

| Scenario | Emergency Quota | Standard Quota | Keeper Action |
|----------|-----------------|----------------|---------------|
| Normal | <50% utilized | <50% utilized | Routine refresh |
| Moderate Stress | 50-80% utilized | 50-70% utilized | Increased monitoring |
| Severe Stress | >80% utilized | >70% utilized | Reduced refresh, stricter approval |
| Crisis | Depleted | >90% utilized | Pause new redemptions |

### Recovery Timeline

| Phase | Duration | Actions |
|-------|----------|---------|
| Immediate | 0-24h | Pause emergency redemptions, assess situation |
| Short-term | 1-7d | Process pending T+7 redemptions, communicate clearly |
| Medium-term | 1-4w | Asset liquidation if needed |
| Recovery | 4-12w | Gradually restore quotas, return to normal operations |

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

All core contracts will undergo comprehensive security audits before mainnet deployment. The audit process includes:

1. **Pre-audit**: Internal code review and testing
2. **Primary audit**: Engagement with a reputable security firm
3. **Remediation**: Address all identified issues
4. **Re-audit**: Verification of fixes
5. **Public disclosure**: Full audit reports published

Audit reports will be made publicly available upon completion. Subscribe to our official channels ([X](https://x.com/Paimon_Finance), [Telegram](https://t.me/paimon_rwa)) for audit announcements.

### Bug Bounty Program

| Severity | Reward Range |
|----------|--------------|
| Critical | $50K - $500K |
| High | $10K - $50K |
| Medium | $2K - $10K |
| Low | $500 - $2K |

## User Responsibility

Users should understand:

1. **PP is not a stablecoin** - NAV fluctuates with underlying assets
2. **Instant liquidity is not guaranteed** - Redemptions are subject to quota availability and approval thresholds
3. **Large redemptions require approval** - Amounts exceeding thresholds (50K standard, 30K emergency) need keeper approval
4. **Junior tranche (jPP) has leverage risk** - Can lose entire principal *(Phase 2 feature)*
5. **Governance decisions affect returns** - Participate or accept outcomes
6. **Smart contract risk exists** - Despite audits, bugs are possible
