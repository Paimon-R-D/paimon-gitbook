# Launchpad: Compliance Issuance Layer

## Asset Scope

Launchpad supports three categories of alternative assets:

| Category | Examples |
|----------|----------|
| **RWA** | Revenue Rights, Accounts Receivable, Structured Product Shares |
| **Private Equity** | Pre-IPO shares, employee option pools, SPV interests |
| **Private Funds** | GP/LP interests, income rights certificates |

## Hard Thresholds: Non-Negotiable Entry Requirements

Any asset must **simultaneously meet** all three criteria below to qualify for the Prime candidate pool:

| Threshold | Requirement | Verification Method |
|-----------|-------------|---------------------|
| **Custody** | Clear custody/fiduciary structure with verifiable fund flow paths | Custody agreements, bank statements, on-chain custody proof |
| **Audit** | Audit responsibility entity and scope clearly defined, with disclosable outputs | Audit reports, audit letters, commitment to ongoing audits |
| **Disclosure Frequency** | Minimum disclosure cycle and field list | Disclosure templates, update commitments, historical records |

### Design Logic

These three elements constitute the minimum requirements for establishing a trustworthy mapping between "on-chain tokens → off-chain equity." The absence of any one element prevents investors from verifying the true value represented by the tokens.

## Soft Scoring: Impacting Risk Weighting and Incentives

After meeting hard thresholds, assets enter the soft scoring system, which determines:

- **Risk Level**: Affects allocation weight within Prime and redemption priority
- **Incentive Weighting**: Determines eligibility for mining emission allocation
- **Onboarding priority**: Review frequency and listing timeline

## Submission Flow

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│ Submission  │ ──→ │ Hard Gates  │ ──→ │ Soft Scoring │
└─────────────┘     └──────┬──────┘     └──────┬──────┘
                           │ Fail            │ Pass
                           ↓                 ↓
                    ┌─────────────┐     ┌─────────────┐
                    │ Rejected    │     │ Candidate   │
                    │ (Not Listed)│     │ Pool        │
                    └─────────────┘     └─────────────┘
```

## Disclosure Requirements by Asset Type

| Asset Type | Disclosure Frequency | Required Fields |
|------------|---------------------|-----------------|
| RWA | Monthly | Valuation, Cash Flow, Significant Events |
| Private Equity | Quarterly | Valuation, Milestones, Funding Updates |
| Private Equity Funds | Quarterly | NAV, Distributions, Portfolio Changes |

## Why These Requirements Matter

### Custody Requirement
Without clear custody, there's no verifiable link between the token and the underlying asset. Users cannot confirm that their tokens represent actual ownership claims.

### Audit Requirement
Without audits, NAV becomes a "black box." Users must trust issuer-provided numbers without independent verification.

### Disclosure Requirement
Without regular disclosure, information asymmetry grows over time. Users cannot make informed decisions about holding or redeeming.

## Governance Interaction

While Launchpad hard thresholds are **non-negotiable** and cannot be overridden by governance voting, the soft scoring system and weighting parameters can be adjusted through governance proposals.

This separation ensures:
- Core trust assumptions are protected from governance attacks
- Flexibility exists for optimizing incentive allocation
- Bad actors cannot "vote in" substandard assets
