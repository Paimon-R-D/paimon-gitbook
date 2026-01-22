# Roadmap

## Development Phases

| Phase | Milestones | Key Deliverables | Status |
|-------|------------|------------------|--------|
| **M0** | Paimon Prime | Prime Vault, Two-Channel Redemption, NAV Calculation | ✅ Implemented |
| **M1** | Launchpad | Asset Onboarding, Hard Gate Review, Soft Scoring | 🚧 In Progress |
| **M2** | DEX Launch | PPT/USDC Trading Pair, Liquidity Incentives | Planned |
| **M3** | Tranche Launch | sPPT/jPPT Tiering, Epoch Settlement | Planned |
| **M4** | Mining Launch | jPPT Gauge, esPAIMON Emission | Planned |
| **M5** | Governance Launch | vePAIMON, Voting System, Protocol Fee Sharing | Planned |
| **M6** | Full Cycle | AMP System, Protection Band Automation, Plugin System | Planned |
| **M7** | Evolution | v4 Hooks, Compliance Gating, Cross-chain Expansion | Planned |

## Phase Details

### M0: Paimon Prime ✅

**Objective**: Launch the core asset management infrastructure

**Deliverables**:
- Prime Vault (ERC-4626) smart contracts
- Two-channel redemption mechanism (T+0, T+7) with approval workflow
- NAV calculation via AssetController
- Redemption Voucher (ERC-721) for delayed settlements
- Basic UI for deposit/redemption

**Success Criteria**:
- Contracts audited
- Test deployment successful
- Documentation complete

---

### M1: Launchpad 🚧

**Objective**: Enable compliant asset onboarding to Prime Vault

**Deliverables**:
- Hard Gate Review system (custody, audit, disclosure requirements)
- Soft Scoring mechanism for risk assessment
- Asset metadata and disclosure portal
- Issuer onboarding workflow

**Success Criteria**:
- First asset successfully onboarded
- Review process documented
- Disclosure standards published

---

### M2: DEX Launch

**Objective**: Enable secondary market trading for PPT

**Deliverables**:
- PPT/USDC liquidity pool deployment
- Initial liquidity provision
- Trading interface integration
- Basic LP incentives

**Success Criteria**:
- Minimum $1M initial liquidity
- Stable trading operations
- TWAP oracle functioning

---

### M3: Tranche Launch

**Objective**: Introduce yield stratification products

**Deliverables**:
- Tranche Vault contracts (sPPT/jPPT)
- Epoch settlement mechanism
- Priority repayment logic
- Junior ratio constraints

**Success Criteria**:
- Correct yield distribution
- Proper priority handling
- Risk controls active

---

### M4: Mining Launch

**Objective**: Activate token emission and staking incentives

**Deliverables**:
- jPPT Gauge deployment
- esPAIMON emission system
- Vesting tier selection
- Boost mechanism

**Success Criteria**:
- Emission tracking accurate
- Vesting working correctly
- Boost calculations verified

---

### M5: Governance Launch

**Objective**: Enable decentralized governance

**Deliverables**:
- vePAIMON locking mechanism
- Gauge voting system
- Protocol fee distribution
- Proposal framework

**Success Criteria**:
- Voting power correctly calculated
- Fee distribution accurate
- Governance UI functional

---

### M6: Full Cycle

**Objective**: Complete the protocol ecosystem

**Deliverables**:
- AMP system with tiered access
- Protection Band automation (TWAP monitoring, auto-pause)
- Plugin system for delegates
- External incentive (Nitro) support
- Advanced analytics dashboard

**Success Criteria**:
- AMPs onboarded and active
- Protection Band triggers working
- Plugins deployed
- Nitro incentives distributed

---

### M7: Evolution

**Objective**: Implement advanced features and expand capabilities

**Deliverables**:
- Uniswap v4 hooks integration
- Compliance gating options
- Lending layer exploration
- Cross-chain expansion

**Success Criteria**:
- Hooks deployed and tested
- Compliance framework documented
- Expansion strategy defined

---

## Timeline Visualization

```
        M0       M1       M2       M3       M4       M5       M6       M7
        │        │        │        │        │        │        │        │
        ▼        ▼        ▼        ▼        ▼        ▼        ▼        ▼
┌────────────────────────────────────────────────────────────────────────────┐
│                                                                            │
│  Prime  │ Launch │  DEX   │Tranche │ Mining │  Gov   │  Full  │  Evol    │
│  Vault  │  pad   │ Launch │ Launch │ Launch │ Launch │ Cycle  │  ution   │
│   ✅    │   🚧   │        │        │        │        │        │          │
│                                                                            │
└────────────────────────────────────────────────────────────────────────────┘
        │        │        │        │        │        │        │        │
      Core    Asset   Secondary  Yield   Token   Decen-   Eco-   Advanced
      Infra   Onboard  Market    Split  Emission tralize  system Features
```

## Dependencies

```
M0 (Paimon Prime) ✅
    │
    └──► M1 (Launchpad) 🚧
              │
              ├──────────────► M2 (DEX Launch)
              │                    │
              │                    ▼
              └──────────────► M3 (Tranche Launch)
                                   │
                                   ▼
                              M4 (Mining Launch)
                                   │
                                   ▼
                              M5 (Governance Launch)
                                   │
                                   ▼
                              M6 (Full Cycle)
                                   │
                                   ▼
                              M7 (Evolution)
```

## Risk Factors

| Phase | Key Risks | Mitigation |
|-------|-----------|------------|
| M0 | Smart contract bugs | Multiple audits, bug bounty |
| M1 | Asset quality issues | Strict hard gate criteria |
| M2 | Insufficient liquidity | Incentive programs, AMP partners |
| M3 | Incorrect yield calculation | Extensive testing, formal verification |
| M4 | Token price volatility | Gradual emission, vesting tiers |
| M5 | Low governance participation | Incentive alignment, education |
| M6 | AMP gaming, Protection Band manipulation | Strict monitoring, penalty system |
| M7 | Regulatory changes | Flexible compliance framework |

## Success Metrics by Phase

| Phase | Primary Metric | Target |
|-------|----------------|--------|
| M0 | TVL | $10M |
| M1 | Assets Onboarded | 3 qualified assets |
| M2 | Daily Volume | $1M |
| M3 | Tranche Utilization | 50% of PPT |
| M4 | Staking Rate | 30% of jPPT |
| M5 | Voting Participation | 20% of vePAIMON |
| M6 | AMP Count | 5 active AMPs |
| M7 | v4 Hook Adoption | 3 custom hooks |
