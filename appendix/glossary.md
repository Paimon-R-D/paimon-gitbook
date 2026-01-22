# Glossary

## Token Terms

| Term | Definition |
|------|------------|
| **PPT** | Paimon Prime Token, the ERC-20 share token of Prime Vault, representing a proportional claim on the underlying assets |
| **sPPT** | Senior PPT, the priority share of Tranche Vault, entitled to a fixed 4% annualized yield and priority repayment |
| **jPPT** | Junior PPT, the subordinated share of Tranche Vault, bears downside risk and receives all remaining returns (leverage effect) |
| **PAIMON** | Protocol Governance Token, lockable for vePAIMON |
| **esPAIMON** | Escrowed PAIMON, staked jPPT incentive tokens. Requires 90-day linear vesting to unlock as PAIMON, or can be directly used for Boost |
| **vePAIMON** | Vote-escrowed PAIMON, governance credentials obtained by locking PAIMON, used for Gauge voting and protocol fee sharing |
| **HYD** | High-Yield Distribution, third-tier high-yield asset allocation |

## Vault Terms

| Term | Definition |
|------|------------|
| **Prime Vault** | The core ERC-4626 vault that holds underlying assets and issues PPT shares |
| **Tranche Vault** | Yield Stratification Contract: Splits PPT returns into Senior (fixed) and Junior (floating) tiers |
| **NAV** | Net Asset Value, the net asset value per PPT share |
| **ERC-4626** | Ethereum standard for tokenized vaults, providing standardized deposit/withdraw interfaces |

## Liquidity Terms

| Term | Definition |
|------|------------|
| **Budget-0** | Instant liquidity budget (L1), used for T+0 emergency redemptions |
| **Budget-7** | Standard liquidity budget (L1 + L2), used for T+7 redemptions |
| **Budget-L** | Long-term liquidity budget (L3), quarterly liquidation cycle |
| **Protection Band** | PPT permitted deviation range between market price and NAV (±15%) |
| **TWAP** | Time-Weighted Average Price, used for manipulation-resistant price sampling |

## Governance Terms

| Term | Definition |
|------|------------|
| **Gauge** | Emission Allocation Voting Mechanism: vePAIMON holders determine incentive weightings for each pool |
| **Boost** | Reward amplification mechanism computed under eligibility rules; may incorporate vePAIMON voting power. esPAIMON carries no governance rights and must vest into PAIMON before PAIMON can be locked into vePAIMON |
| **Epoch** | Settlement cycle for Tranche Vaults, with rewards distributed every 7 days |
| **Timelock** | Delay period between governance approval and execution |

## Market Terms

| Term | Definition |
|------|------------|
| **AMP** | Authorized Market Participant, professional market makers with special privileges and obligations |
| **P_mkt** | Market price of PPT in the DEX pool |
| **D_t** | Deviation, the absolute percentage difference between P_mkt and NAV |
| **Nitro** | External incentive system allowing third parties to add rewards to Gauges |
| **Weakly Coupled Pricing** | A pricing model where the on-chain market price (P_mkt) is loosely tied to the off-chain NAV, allowing for market-driven price discovery within the protection band (±15%) while NAV serves as the ultimate redemption reference |

## Risk Terms

| Term | Definition |
|------|------------|
| **Hard Threshold** | Non-negotiable requirements that cannot be overridden by governance (custody, audit, disclosure) |
| **Soft Scoring** | Adjustable risk weighting system for assets that pass hard thresholds |
| **Emergency Reserve** | Fund set aside for liquidity events and crisis response |

## Technical Terms

| Term | Definition |
|------|------------|
| **Merkle Root** | Cryptographic commitment to a set of reward claims |
| **Multi-sig** | Multi-signature wallet requiring multiple approvers for transactions |
| **Oracle** | External data source providing price and NAV information |
| **Subgraph** | Indexed on-chain data for efficient querying |

## Asset Terms

| Term | Definition |
|------|------------|
| **RWA** | Real World Assets, tokenized physical or financial assets |
| **Private Equity** | Pre-IPO shares, employee options, SPV interests |
| **Private Funds** | GP/LP interests, income rights certificates |

## Abbreviations

| Abbreviation | Full Form |
|--------------|-----------|
| AMM | Automated Market Maker |
| APY | Annual Percentage Yield |
| DEX | Decentralized Exchange |
| KYC | Know Your Customer |
| LP | Liquidity Provider |
| TVL | Total Value Locked |
| ve | Vote Escrow |
