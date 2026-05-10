# Audit Reports

Paimon Finance is committed to security and transparency. All smart contracts undergo rigorous third-party security audits before deployment.

## Smart Contract Audits

| Auditor | Date | Scope | Report |
|---------|------|-------|--------|
| CertiK | January 19, 2026 | Prime Vault smart contracts | [Download PDF](../.gitbook/assets/audit-report-2026-01.pdf) |
| SlowMist | March 2026 | Paimon (PPT) BSC contracts | [Download PDF](<../.gitbook/assets/Paimon(PPT) BSC Contracts - SlowMist Audit Report.pdf>) |

The Prime Vault contracts (`PPT`, `RedemptionManager`, `AssetController`, `RedemptionVoucher`, `PPTTypes`, `IPPTContracts`) have undergone two independent third-party audits. Both reports are published in full above.

## CertiK Audit Summary

**Audit Details:**
- **Type**: Vault
- **Ecosystem**: EVM Compatible
- **Methods**: Manual Review, Static Analysis
- **Language**: Solidity

**Findings Overview:**

| Severity | Total | Resolved | Acknowledged |
|----------|-------|----------|--------------|
| Critical | 0 | - | - |
| Major | 2 | 2 | 0 |
| Medium | 11 | 11 | 0 |
| Minor | 18 | 17 | 1 |
| Informational | 1 | 1 | 0 |
| Centralization | 1 | 0 | 1 |

**Total Findings: 33** | **Resolved: 31** | **Acknowledged: 2**

## SlowMist Audit Summary

**Audit Details:**
- **Type**: Smart-contract security audit (Paimon BSC contracts — PPT scope)
- **Ecosystem**: BNB Smart Chain (EVM compatible)
- **Methods**: Manual review + static analysis
- **Language**: Solidity

For the full list of findings, severity distribution, remediation status and re-test results, refer to the published PDF report linked above.

## Audited Contracts (Prime Vault scope)

- `AssetController.sol`
- `PPT.sol` (deployed as `PP` symbol)
- `PPTTypes.sol`
- `RedemptionManager.sol`
- `RedemptionVoucher.sol`
- `IPPTContracts.sol`

## Pre-IPO / Compliance Layer (External Audit Pending)

The Prime Vault scope above is covered by **two** independent external audits (CertiK + SlowMist). The remaining production contracts have not yet completed external audit:

- EIP-3643 contract suite — `EIP3643Token`, `ShadowERC20`, `TokenBridge`, `KYCAggregator`, `SimpleKYCProvider`
- Launchpad / Points / Badge — `LaunchpadDrop`, `LaunchpadSettlement`, `PaimonTreasury`, `PaimonBadge`, `PointsHubV2`, `StakingModule`, `LPStakingModule`, `PointsRedemption`

These contracts are deployed on BSC mainnet and have been **internally reviewed**. External audit reports for this scope will be published as they are completed.

## Ongoing Security

- Continuous monitoring of deployed contracts via the operational backend's event ingestion pipeline (redundant transports with gap recovery)
- Bug bounty program (coming soon)
- UUPS upgrade authority gated by multisig + timelock for every proxy
- Regular security reviews for contract upgrades
