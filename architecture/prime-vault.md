# Prime Vault: ERC-4626 Aggregation Layer

## PP and NAV

**PP (Paimon Prime)** is an ERC-20 share token issued by Prime Vault, representing the holder's proportional claim on the Vault's net assets.

> **Implementation Note**: The token symbol in the deployed contract is `PP` (Paimon Prime).

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
| **Minimum Deposit** | 500 tokens | Prevents dust deposits and reduces gas overhead |
| **Locked Mint Period** | Configurable | New deposits may be locked during accumulation periods |

### Key Principle

Short-term DEX pool prices **shall not serve as** direct inputs for NAV. NAV must primarily rely on audited/reconciled/valued sources, with market prices functioning solely as risk control signals (e.g., deviation triggers).

## Tiered Liquidity Model

Prime categorizes assets into three tiers based on liquidation cycles:

| Tier | Name | Liquidity Cycle | Purpose | Typical Assets |
|------|------|-----------------|---------|----------------|
| **L1** | Instant Liquidity | T+0 | Emergency Redemption | Stablecoins, highly liquid tokens |
| **L2** | Standard Liquidity | T+7 | Standard Redemption | Moderately liquid assets, short-term RWA |
| **L3** | Quarterly Liquidity | T+90 | Long-Term Allocation | Private equity, long-lockup assets |

### Portfolio Allocation

```
┌─────────────────────────────────────────┐
│           Prime Vault Portfolio          │
├─────────────────────────────────────────┤
│  L1: Instant Liquidity (Budget-0)  ~5-15%│
│  ─────────────────────────────────────  │
│  L2: Standard Liquidity (Budget-7)~25-35%│
│  ─────────────────────────────────────  │
│  L3: Quarterly Liquidity (Budget-L)~55-65%│
│       └── Includes HYD (High-Yield)     │
└─────────────────────────────────────────┘
```

## HYD: Third-Tier High-Yield Allocation

**HYD (High-Yield Distribution)** represents the third layer (L3) of allocation within the Prime portfolio, typically comprising **55-65%** of total assets. This tier provides exposure to high-yield alternative assets including:

- Real World Assets (RWA)
- Private Equity
- Private Fund interests

> **Important Notice**: HYD **is not** a stablecoin; its value fluctuates based on the performance of underlying assets. See [Glossary](../appendix/glossary.md) for detailed token definitions.

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

- **Tiered redemption channels** (T+0, T+7, Queued)
- **NAV-based pricing** with protection band integration
- **Budget-constrained withdrawals**
- **Queue priority calculation**

## State Machine

```
User Request
     │
     ▼
┌──────────────┐
│ Check Budget │
└──────┬───────┘
       │
   ┌───┴───┬────────┬─────────┐
   ▼       ▼        ▼         ▼
Budget-0  Budget-7  Queue   Rejected
(T+0)     (T+7)     Entry   (No Budget)
   │        │         │
   ▼        ▼         ▼
Instant   Pending   Waiting
Execute   7 days    In Queue
   │        │         │
   ▼        ▼         ▼
Complete  Execute   Batch
          After 7d  Settlement
```
