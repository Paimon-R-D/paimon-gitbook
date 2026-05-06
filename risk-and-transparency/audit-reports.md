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

## Audited Contracts (Prime Vault scope)

- `AssetController.sol`
- `PPT.sol` (deployed as `PP` symbol)
- `PPTTypes.sol`
- `RedemptionManager.sol`
- `RedemptionVoucher.sol`
- `IPPTContracts.sol`

## Pre-IPO / Compliance Layer

The EIP-3643 contract suite (`EIP3643Token`, `ShadowERC20`, `TokenBridge`, `KYCAggregator`, `SimpleKYCProvider`) and the Launchpad / Points / Badge contracts (`LaunchpadDrop`, `LaunchpadSettlement`, `PaimonTreasury`, `PaimonBadge`, `PointsHubV2`, `StakingModule`) are deployed on BSC mainnet and have been internally reviewed. External audit reports for this scope will be published as they are completed.

## Ongoing Security

- Continuous monitoring of deployed contracts via the backend event listener (WebSocket + polling + gap-fill)
- Bug bounty program (coming soon)
- UUPS upgrade authority gated by multisig + timelock for every proxy
- Regular security reviews for contract upgrades
