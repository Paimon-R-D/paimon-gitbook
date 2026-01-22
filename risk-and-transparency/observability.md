# Observability and Transparency

## Data Disclosure Principles

All critical operations are recorded via on-chain events, accessible for public query and verification:

- Deposits
- Redemption requests
- Redemption settlements
- Queue updates
- NAV updates
- Protection band triggers

## Launchpad Disclosure Requirements

Minimum disclosure requirements by asset class:

| Asset Type | Disclosure Frequency | Required Fields |
|------------|---------------------|-----------------|
| RWA | Monthly | Valuation, Cash Flow, Significant Events |
| Private Equity | Quarterly | Valuation, Milestones, Funding Updates |
| Private Equity Funds | Quarterly | NAV, Distributions, Portfolio Changes |

## On-Chain Events

### Prime Vault Events

```solidity
event Deposit(address indexed user, uint256 assets, uint256 shares);
event RedemptionRequested(address indexed user, uint256 shares, uint8 channel);
event RedemptionSettled(address indexed user, uint256 assets, uint256 fee);
event NAVUpdated(uint256 oldNAV, uint256 newNAV, uint256 timestamp);
```

### Tranche Vault Events

```solidity
event EpochSettled(uint256 epoch, int256 yield, uint256 sPPTYield, int256 jPPTYield);
event TrancheDeposit(address indexed user, bool isSenior, uint256 pptAmount, uint256 shares);
event TrancheRedemption(address indexed user, bool isSenior, uint256 shares, uint256 pptAmount);
```

### Protection Band Events

```solidity
event ProtectionBandTriggered(uint256 deviation, uint256 timestamp);
event ProtectionBandRecovered(uint256 deviation, uint256 timestamp);
event EmergencyRedemptionPaused(uint256 timestamp);
event EmergencyRedemptionResumed(uint256 timestamp);
```

## Dashboard Metrics

### Real-Time Metrics

| Metric | Update Frequency | Source |
|--------|------------------|--------|
| PPT Total Supply | Per block | Prime Vault |
| NAV | Weekly (minimum) | Oracle + Audit |
| DEX Price (P_mkt) | Per block | DEX Pool |
| Deviation (D_t) | Per block | Calculated |
| L1 Budget Utilization | Per block | Prime Vault |
| L2 Budget Utilization | Per block | Prime Vault |
| Queue Depth | Per block | Redemption Queue |

### Weekly Reports

Automated weekly report includes:

1. **Supply Metrics**
   - Total PPT supply
   - sPPT / jPPT breakdown
   - PAIMON circulating supply
   - Unvested esPAIMON amounts

2. **Liquidity Metrics**
   - L1/L2/L3 budget utilization
   - Queue depth and estimated wait times
   - DEX depth and volume

3. **Price Metrics**
   - NAV history
   - P_mkt history
   - Deviation (D_t) history
   - Protection band events

4. **Emission Metrics**
   - Weekly emission amount
   - Channel allocation breakdown
   - Unused budget returned

5. **Governance Metrics**
   - Gauge weights
   - vePAIMON distribution
   - Proposal activity

## API Access

### Public Endpoints

| Endpoint | Description | Rate Limit |
|----------|-------------|------------|
| `/api/v1/nav` | Current NAV and history | 100/min |
| `/api/v1/liquidity` | Budget utilization | 100/min |
| `/api/v1/queue` | Queue status | 100/min |
| `/api/v1/emissions` | Emission schedule | 60/min |
| `/api/v1/gauges` | Gauge weights | 60/min |

### Data Formats

All endpoints return JSON with consistent structure:

```json
{
  "success": true,
  "timestamp": 1704067200,
  "data": { ... }
}
```

## Subgraph (The Graph)

A public subgraph indexes all on-chain events for historical queries:

### Example Queries

**Get historical NAV:**
```graphql
query {
  navUpdates(first: 100, orderBy: timestamp, orderDirection: desc) {
    timestamp
    oldNAV
    newNAV
    blockNumber
  }
}
```

**Get redemption activity:**
```graphql
query {
  redemptions(where: { user: "0x..." }) {
    channel
    shares
    assets
    fee
    timestamp
  }
}
```

## Alerting System

### Public Alerts

Users can subscribe to alerts for:

| Alert Type | Trigger | Channel |
|------------|---------|---------|
| Protection Band | D_t approaches 15% | Discord, Telegram |
| High Queue Depth | Queue > 10% of TVL | Discord, Telegram |
| NAV Update | New NAV published | Discord, Telegram |
| Governance | New proposal created | Discord, Telegram |

### Risk Committee Alerts

Internal alerts with higher urgency:

| Alert Type | Trigger | Response |
|------------|---------|----------|
| Critical | D_t > 12% | Immediate review |
| Emergency | D_t > 15% | Emergency meeting |
| Audit Delay | NAV update > 14 days late | Escalation |
| Budget Crisis | L1 + L2 > 90% | Contingency activation |

## Audit Trail

### Immutable Records

All critical decisions are recorded immutably:

- Governance proposals and votes
- Multi-sig transactions
- Parameter changes
- Emergency actions

### Access Logs

- API access logged (anonymized)
- Contract interactions indexed
- Admin actions recorded

## Third-Party Integrations

### Oracle Providers

| Provider | Purpose | Backup |
|----------|---------|--------|
| TBD | Price feeds | Multi-source |
| TBD | NAV verification | Manual fallback |

### Analytics Partners

| Partner | Service |
|---------|---------|
| TBD | Dashboard hosting |
| TBD | Subgraph indexing |
| TBD | Alert distribution |

## Transparency Commitments

1. **No hidden data**: All material information is publicly accessible
2. **Real-time updates**: Critical metrics update per block
3. **Historical access**: Full history available via subgraph
4. **Open API**: Anyone can query protocol data
5. **Regular reporting**: Weekly automated reports published
