# Audit Reports

Paimon Finance is committed to security and transparency. All smart contracts undergo rigorous third-party security audits before deployment.

## Smart Contract Audits

| Auditor | Date | Scope | Report |
|---------|------|-------|--------|
| CertiK | January 19, 2026 | Vault Smart Contracts | [Download PDF](../.gitbook/assets/audit-report-2026-01.pdf) |

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

## Audited Contracts

- AssetController.sol
- PPT.sol
- PPTTypes.sol
- RedemptionManager.sol
- RedemptionVoucher.sol
- IPPTContracts.sol

## Ongoing Security

- Continuous monitoring of deployed contracts
- Bug bounty program (coming soon)
- Regular security reviews for upgrades
