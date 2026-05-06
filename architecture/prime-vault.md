# Paimon Prime Vault: ERC-4626 Aggregation Layer

## PP and NAV

**PP (Paimon Prime)** is the share token issued by Prime Vault, representing the holder's proportional claim on the Vault's net assets.

{% hint style="info" %}
**Naming convention used throughout this document**
- **Contract name / source code**: `PPT` (Paimon Prime Token) — see `src/ppt/PPT.sol`
- **On-chain ERC-20 `name` / `symbol`**: `PP Token` / `PP` (this is what wallets display)
- **Standard**: ERC-4626 vault, UUPS upgradeable
- **BSC mainnet address**: `0x8505c32631034A7cE8800239c08547e0434EdaD9`

Whenever this documentation says "PP", it refers to the same token your wallet shows as `PP`. The internal contract identifier `PPT` is used in audit reports and source code.
{% endhint %}

### NAV Calculation Formula

$$
\mathrm{NAV}_t=\frac{V_{\text{assets}}-V_{\text{liabilities}}}{S_{\text{effective}}}
$$

Where:
- **V_assets**: Fair value of assets held by Prime (grossValue from disclosures/audits)
- **V_liabilities**: Total redemption liabilities + withdrawable fees
- **S_effective**: Effective supply = Total PP supply - Locked shares (shares pending settlement)

### Deposit Constraints

| Parameter | Value | Description |
|-----------|-------|-------------|
| **Minimum Deposit** | 10 tokens (`MIN_DEPOSIT = 10e18`) | Hard-coded in `PPTTypes.sol` to prevent dust deposits |
| **Locked Mint Assets** | Tracked per period via `lockedMintAssets` | Recently minted underlying may be temporarily ineligible for redemption coverage |
| **Virtual Offset** | `1e18` | Anti-inflation-attack offset applied in `sharePrice()` calculation |

### Key Principle

Short-term DEX pool prices **shall not serve as** direct inputs for NAV. NAV must primarily rely on audited/reconciled/valued sources, with market prices functioning solely as risk control signals (e.g., deviation triggers).

## Tiered Liquidity Model

Prime categorizes assets into three liquidity tiers, defined on-chain by the `LiquidityTier` enum (`PPTTypes.sol:37-41`):

| Tier | Enum | Liquidity Cycle | Purpose | Typical Assets |
|------|------|-----------------|---------|----------------|
| **L1** | `TIER_1_CASH` | Instant (T+0) | Emergency redemption coverage | USDT cash, instantly-liquid stable assets |
| **L2** | `TIER_2_MMF` | Short (~T+7) | Standard redemption coverage | Money market funds, short-term yield assets (e.g. CashPlus / CASH+) |
| **L3** | `TIER_3_HYD` | Quarterly+ (T+90 or longer) | Long-duration yield | Private equity, private credit, long-lockup RWA |

{% hint style="info" %}
**About the name "HYD" in this Tier-3 enum**

The enum value `TIER_3_HYD` stands for **"High-Yield"** — it is a category label for the long-duration tranche of the portfolio, **not** a token symbol. Do not confuse this with the synthetic-asset token of the same name described in some Phase-2 design documents; that token is **not deployed**.
{% endhint %}

### Portfolio Allocation

The actual L1/L2/L3 target ratios are **not hard-coded** in the contracts. They are configured at runtime through the off-chain `RebalanceStrategyService` (single-active-strategy pattern) and pushed to `AssetController` as `LayerConfig { targetRatio, minRatio, maxRatio }` values.

The contract layer only enforces two safety thresholds (`PPTTypes.sol:26-27`):

| Constant | Value | Purpose |
|----------|-------|---------|
| `LOW_LIQUIDITY_THRESHOLD` | 1500 bps (15 %) | Low-liquidity warning |
| `CRITICAL_LIQUIDITY_THRESHOLD` | 1000 bps (10 %) | Critical-liquidity warning |

Typical strategy targets (subject to governance / fund manager update):

```
┌─────────────────────────────────────────┐
│           Prime Vault Portfolio          │
├─────────────────────────────────────────┤
│  L1 (TIER_1_CASH)   ~5-15 % typical     │
│  L2 (TIER_2_MMF)    ~25-35 % typical    │
│  L3 (TIER_3_HYD)    ~55-65 % typical    │
└─────────────────────────────────────────┘
```

## L3: Long-Duration Yield Allocation

The Tier-3 sleeve typically holds **55-65 %** of total assets and provides exposure to:

- Real World Assets (RWA) with quarterly liquidity windows
- Private equity / private credit
- Tokenized fund interests

PP itself is **not** a stablecoin; its NAV fluctuates with underlying asset performance.

### Operational Commitments

| Commitment Items | Specific Requirements |
|------------------|----------------------|
| NAV Disclosure Frequency | At least once per week |
| Redemption Window | Fixed quarterly window, announced 30 days in advance |
| Single Window Cap | Maximum redemption amount subject to liquidity budget constraints |
| Contingency Reserve | Special Disposal Fund to address liquidity events |

## ERC-4626 Compliance

Prime Vault implements the ERC-4626 tokenized vault standard, providing:

### Standard Functions

| Function | Description |
|----------|-------------|
| `deposit(assets, receiver)` | Deposit assets and receive shares |
| `withdraw(assets, receiver, owner)` | Withdraw assets by burning shares |
| `redeem(shares, receiver, owner)` | Redeem shares for assets |
| `convertToShares(assets)` | Preview share amount for given assets |
| `convertToAssets(shares)` | Preview asset amount for given shares |

### Custom Extensions

Paimon extends ERC-4626 with:

- **Three redemption channels** — `STANDARD` (T+7), `EMERGENCY` (T+0, fee-penalized), `SCHEDULED` (large-window, currently unused but reserved) — see `RedemptionChannel` enum in `PPTTypes.sol:57-61`
- **NAV-based pricing** via `sharePrice()` with virtual-offset protection against inflation attacks
- **Budget-constrained withdrawals** via `emergencyQuota` (refreshed by KEEPER) and `standardQuotaRatio` (default 70 % of available liquidity)
- **Direct redeem disabled** — users must go through `RedemptionManager` so the approval / locking / settlement flow can run
- **Locked share accounting** — `totalLockedShares` and `lockedSharesOf[user]` track shares pending redemption settlement; these are excluded from `effectiveSupply()` used in NAV math

## Live Asset Adapters

Prime Vault holds underlying assets directly when they are simple ERC-20s, and uses adapters for assets with off-chain settlement flows:

| Adapter | Mainnet address | Underlying | Notes |
|---------|-----------------|------------|-------|
| `CashPlusAdapter` | `0xf3a17a5362b6f5b2bCB1AE0C0DE86b70e1ae4a53` | CashPlus Vault `0x1775504c5873e179Ea2f8ABFcE3861EC74D159bc` | External RWA money-market fund (CASH+ token, ~$107). Subscription requires **off-chain SPV approval**; backend probes `claim()` via `eth_call` once per hour and auto-advances `SUBSCRIPTION_LINKED → SPV_APPROVED → CONFIRMED` |
| `PaimonOracleAdapter` | — | Multi-asset price oracle | Aggregates Chainlink + custodian NAV feeds with circuit-breaker checks |

See [Redemption Mechanism](redemption-mechanism.md) for the full request → approval → settlement state machine.
