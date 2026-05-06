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

The full event surface is the canonical telemetry of Paimon protocols — every event below is emitted by a deployed contract. Tranche / Protection-Band / Gauge events are **not** emitted because the corresponding contracts are not deployed.

### Prime Vault (`PPT`)

```solidity
event Deposit(address indexed sender, address indexed owner, uint256 assets, uint256 shares);
event SharesLocked(address indexed owner, uint256 shares);
event SharesUnlocked(address indexed owner, uint256 shares);
event SharesBurned(address indexed owner, uint256 shares);
event SelfBurn(address indexed owner, uint256 shares);
event RedemptionFeeAdded(uint256 fee);
event RedemptionFeeReduced(uint256 fee);
event EmergencyQuotaRefreshed(uint256 amount);
event EmergencyQuotaRestored(uint256 amount);
event EmergencyQuotaReduced(uint256 amount);
event RedemptionLiabilityAdded(uint256 amount);
event RedemptionLiabilityRemoved(uint256 amount);
event LockedMintAssetsReset(uint256 oldAmount);
event AssetControllerUpdated(address indexed oldController, address indexed newController);
event RedemptionManagerUpdated(address indexed oldManager, address indexed newManager);
event StandardQuotaRatioUpdated(uint256 oldRatio, uint256 newRatio);
event PendingApprovalSharesAdded(address indexed owner, uint256 shares);
event PendingApprovalSharesRemoved(address indexed owner, uint256 shares);
event PendingApprovalSharesConverted(address indexed owner, uint256 shares);
event PPTUpgraded(address indexed newImplementation, uint256 timestamp, uint256 blockNumber);
```

### Redemption Manager

```solidity
event RedemptionRequested(uint256 indexed requestId, address indexed user, uint256 shares, uint8 channel);
event RedemptionApproved(uint256 indexed requestId, address indexed approver);
event RedemptionRejected(uint256 indexed requestId, address indexed rejecter, string reason);
event RedemptionSettled(uint256 indexed requestId, address indexed user, uint256 assets, uint256 fee);
event RedemptionCancelled(uint256 indexed requestId, address indexed user);
```

### Asset Controller

```solidity
event GrossValueUpdated(uint256 oldValue, uint256 newValue);
event DelayedSettlementCreated(uint256 indexed id, uint8 settlementType, uint256 amount);
event DelayedSettlementExecuted(uint256 indexed id);
```

### Pre-IPO SPV — EIP-3643 Token (`pSPCX`)

```solidity
event Transfer(address indexed from, address indexed to, uint256 value);
event Approval(address indexed owner, address indexed spender, uint256 value);
event Paused(address indexed agent);
event Unpaused(address indexed agent);
event AddressFrozen(address indexed userAddress, bool isFrozen, address indexed agent);
event TokensFrozen(address indexed userAddress, uint256 amount);
event TokensUnfrozen(address indexed userAddress, uint256 amount);
event KYCProviderSet(address indexed kycProvider);
```

### Pre-IPO SPV — TokenBridge

```solidity
event PairCreated(address indexed securityToken, address indexed shadowToken, uint256 ratio);
event Deposited(address indexed shadowToken, address indexed user, uint256 securityAmount, uint256 shadowMinted);
event Redeemed(address indexed shadowToken, address indexed user, uint256 shadowBurned, uint256 securityReleased);
event KYCProviderSet(address indexed kycProvider);
```

### Launchpad (`LaunchpadDrop` V4)

```solidity
event DropCreated(uint256 indexed dropId, address indexed rwaToken, uint256 totalSupply, uint256 basePrice);
event LayerConfigUpdated(uint256 indexed dropId, uint256 indexed layerIdx, /* … fields … */);
event PhaseAdvanced(uint256 indexed dropId, uint8 oldPhase, uint8 newPhase);
event Committed(uint256 indexed dropId, uint256 indexed layerIdx, address indexed user, uint256 pointsReserved, uint256 usdtAmount);
event LayerSettled(uint256 indexed dropId, uint256 indexed layerIdx, /* … */);
event AllocationClaimed(uint256 indexed dropId, uint256 indexed layerIdx, address indexed user, uint256 rwaAmount, uint256 usdtSettled);
event Refunded(uint256 indexed dropId, uint256 indexed layerIdx, address indexed user, uint256 usdtReturned);
event RefundWindowClosed(uint256 indexed dropId, uint256 indexed layerIdx);
event SettlementFinalized(uint256 indexed dropId);
```

### Soulbound Badges

```solidity
event BadgeMinted(address indexed user, uint8 badgeType, uint256 indexed dropId, uint64 mintTime);
```

### Points

```solidity
// PointsHubV2
event ModuleRegistered(address indexed module, string name, uint256 moduleIndex);
event RewardAdded(address indexed user, uint256 amount, bytes32 reason);
event PointsDeducted(address indexed user, uint256 amount, bytes32 reason);

// StakingModule
event Staked(address indexed user, uint256 indexed stakeId, uint256 amount, uint8 stakeType, uint64 lockEndTime);
event Unstaked(address indexed user, uint256 indexed stakeId, uint256 amount, uint256 penalty);
event PointsAccrued(address indexed user, uint256 indexed stakeId, uint256 amount);
```

## Dashboard Metrics

### Real-Time Metrics

| Metric | Update Frequency | Source |
|--------|------------------|--------|
| PP Total Supply | Per block | Prime Vault |
| NAV | Weekly (minimum) | Oracle + Audit |
| DEX Price (P_mkt) | Per block | DEX Pool |
| Deviation (D_t) | Per block | Calculated |
| L1 Budget Utilization | Per block | Prime Vault |
| L2 Budget Utilization | Per block | Prime Vault |
| Queue Depth | Per block | Redemption Queue |

### Weekly Reports

Automated weekly report includes:

1. **Supply Metrics**
   - Total PP supply
   - pSPCX locked in TokenBridge / xSPCX outstanding (per pair)
   - Active drops, total committed USDT, settled vs refunded

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

The protocol utilizes multiple oracle sources to ensure data reliability and manipulation resistance:

- **Price Feeds**: Aggregated from leading oracle providers with multi-source verification
- **NAV Verification**: Cross-validated against custodian reports and on-chain data
- **Fallback Mechanisms**: Manual override capability via multi-sig for emergency situations

### Infrastructure Partners

The protocol leverages battle-tested infrastructure:

- **Indexing**: The Graph protocol for historical data queries
- **Monitoring**: Real-time alerting via Discord and Telegram integrations
- **Dashboard**: Self-hosted analytics with public API access

Specific provider details will be announced closer to mainnet launch.

## Transparency Commitments

1. **No hidden data**: All material information is publicly accessible
2. **Real-time updates**: Critical metrics update per block
3. **Historical access**: Full history available via subgraph
4. **Open API**: Anyone can query protocol data
5. **Regular reporting**: Weekly automated reports published
