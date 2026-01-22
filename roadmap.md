# Roadmap

## Development Phases

| Phase | Milestones | Key Deliverables |
|-------|------------|------------------|
| **M0** | Prime MVP | Prime Vault, Three-Channel Redemption, NAV Calculation |
| **M1** | DEX Launch | PPT/USDC Trading Pair, Liquidity Incentives |
| **M2** | Tranche Launch | sPPT/jPPT Tiering, Epoch Settlement |
| **M3** | Mining Launch | jPPT Gauge, esPAIMON Emission |
| **M4** | Governance Launch | vePAIMON, Voting System, Protocol Fee Sharing |
| **M5** | Full Cycle | AMP System, Plugin System |
| **M6** | Evolution | v4 Hooks, Compliance Gating, Lending Layer Exploration |

## Phase Details

### M0: Prime MVP

**Objective**: Launch the core asset management infrastructure

**Deliverables**:
- Prime Vault (ERC-4626) smart contracts
- Three-channel redemption mechanism
- NAV calculation and oracle integration
- Basic UI for deposit/redemption

**Success Criteria**:
- Contracts audited
- Test deployment successful
- Documentation complete

---

### M1: DEX Launch

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

### M2: Tranche Launch

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

### M3: Mining Launch

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

### M4: Governance Launch

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

### M5: Full Cycle

**Objective**: Complete the protocol ecosystem

**Deliverables**:
- AMP system with tiered access
- Plugin system for delegates
- External incentive (Nitro) support
- Advanced analytics dashboard

**Success Criteria**:
- AMPs onboarded and active
- Plugins deployed
- Nitro incentives distributed

---

### M6: Evolution

**Objective**: Implement advanced features and expand capabilities

**Deliverables**:
- Uniswap v4 hooks integration
- Compliance gating options
- Lending layer exploration
- Cross-chain expansion research

**Success Criteria**:
- Hooks deployed and tested
- Compliance framework documented
- Expansion strategy defined

---

## Timeline Visualization

```
        M0       M1       M2       M3       M4       M5       M6
        │        │        │        │        │        │        │
        ▼        ▼        ▼        ▼        ▼        ▼        ▼
┌───────────────────────────────────────────────────────────────────┐
│                                                                   │
│   Prime   │  DEX   │ Tranche │ Mining │  Gov   │  Full  │  Evol  │
│   MVP     │ Launch │ Launch  │ Launch │ Launch │ Cycle  │ ution  │
│                                                                   │
└───────────────────────────────────────────────────────────────────┘
        │        │        │        │        │        │        │
     Core     Secondary  Yield   Token   Decen-   Eco-    Advanced
     Infra    Market     Split   Emission tralize  system  Features
```

## Dependencies

```
M0 (Prime MVP)
    │
    ├──────────────► M1 (DEX Launch)
    │                    │
    │                    ▼
    └──────────────► M2 (Tranche Launch)
                         │
                         ▼
                    M3 (Mining Launch)
                         │
                         ▼
                    M4 (Governance Launch)
                         │
                         ▼
                    M5 (Full Cycle)
                         │
                         ▼
                    M6 (Evolution)
```

## Risk Factors

| Phase | Key Risks | Mitigation |
|-------|-----------|------------|
| M0 | Smart contract bugs | Multiple audits, bug bounty |
| M1 | Insufficient liquidity | Incentive programs, AMP partners |
| M2 | Incorrect yield calculation | Extensive testing, formal verification |
| M3 | Token price volatility | Gradual emission, vesting tiers |
| M4 | Low governance participation | Incentive alignment, education |
| M5 | AMP gaming | Strict monitoring, penalty system |
| M6 | Regulatory changes | Flexible compliance framework |

## Success Metrics by Phase

| Phase | Primary Metric | Target |
|-------|----------------|--------|
| M0 | TVL | $10M |
| M1 | Daily Volume | $1M |
| M2 | Tranche Utilization | 50% of PPT |
| M3 | Staking Rate | 30% of jPPT |
| M4 | Voting Participation | 20% of vePAIMON |
| M5 | AMP Count | 5 active AMPs |
| M6 | v4 Hook Adoption | 3 custom hooks |
