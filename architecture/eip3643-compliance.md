# EIP-3643 Compliance Architecture

The Paimon Pre-IPO product line uses a **two-layer token architecture** to satisfy regulatory requirements without sacrificing DeFi composability. This document describes the contract-level design.

## Why EIP-3643

EIP-3643 is the Ethereum standard for "permissioned" or "regulated" tokens. Beyond the basic ERC-20 surface it adds the primitives that securities issuers actually need:

- **KYC-gated transfer** — every `transfer` / `transferFrom` checks the receiver's identity status
- **Pause** — authorized agents can halt all transfers globally
- **Per-address freeze** — block a specific user from sending or receiving
- **Partial freeze** — freeze a portion of a user's balance (e.g. unvested tranche)
- **Forced transfer** — agent-mediated movement (court orders, inheritance, frozen-account recovery)
- **Identity registry** — pluggable KYC source so issuers can swap providers without redeploying

Paimon's `EIP3643Token` (`src/eip3643/EIP3643Token.sol`) is a UUPS-upgradeable implementation of these primitives backed by a `KYCAggregator` that can fan out to multiple `SimpleKYCProvider` (or future) sources.

## Component Overview

```
                          ┌──────────────────────────────────────┐
                          │           KYCAggregator              │
                          │  (one per deployment, multi-source)  │
                          │  isVerified(addr) = OR of providers  │
                          └─────────────────┬────────────────────┘
                                            │
                            ┌───────────────┼─────────────────┐
                            ▼               ▼                 ▼
                  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐
                  │ SimpleKYC    │  │ Future       │  │ Future       │
                  │ Provider     │  │ Sumsub       │  │ Onfido       │
                  │ (whitelist)  │  │ Provider     │  │ Provider     │
                  └──────────────┘  └──────────────┘  └──────────────┘

                                            ▲
                                            │ isVerified() called on every
                                            │ transfer of pSPCX / on every
                                            │ TokenBridge.redeem()
                                            │
            ┌───────────────────┬───────────┴────────┬────────────────────┐
            │                   │                    │                    │
            ▼                   ▼                    ▼                    ▼
   ┌─────────────────┐  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐
   │  EIP3643Token   │  │ TokenBridge  │    │ EIP3643Token │    │ TokenBridge  │
   │     (pSPCX)     │◄─┤   (1 hub,    ├───►│   (future    │◄───┤   (same)     │
   │                 │  │  N pairs)    │    │   pPNTC...)  │    │              │
   └─────────────────┘  └──────┬───────┘    └──────────────┘    └──────────────┘
                               │                                        │
                               │ mint/burn 1:ratio                      │ mint/burn 1:ratio
                               ▼                                        ▼
                       ┌──────────────┐                          ┌──────────────┐
                       │ ShadowERC20  │                          │ ShadowERC20  │
                       │   (xSPCX)    │                          │ (future...)  │
                       └──────────────┘                          └──────────────┘
```

The hub-and-spoke design lets one `TokenBridge` instance manage `N` pairs of (security token, shadow token) at independent ratios, which is how Paimon will scale beyond the SpaceX SPV to additional pre-IPO assets without redeploying core infrastructure.

## EIP3643Token Contract (pSPCX)

**Source**: `src/eip3643/EIP3643Token.sol` (UUPS upgradeable)
**Mainnet**: pSPCX `0x6DC9a487bF8Fd047e41AB336003AE6e4FE602646`

### State

| Variable | Type | Purpose |
|----------|------|---------|
| `name`, `symbol`, `decimals` | string / uint8 | ERC-20 metadata. `decimals` is hard-fixed at 18 |
| `totalSupply` | uint256 | Tracked manually because every transfer routes through the KYC check |
| `kycProvider` | `IIdentityRegistry` | Pluggable; switchable by `ADMIN_ROLE` |
| `paused` | bool | Global transfer halt |
| `_balances`, `_allowances` | mapping | Standard ERC-20 ledger |
| `_frozen[address]` | bool | Full-address freeze flag |
| `_frozenTokens[address]` | uint256 | Quantity of tokens locked within an account |

### Roles

| Role | Granted to | Purpose |
|------|-----------|---------|
| `DEFAULT_ADMIN_ROLE`, `ADMIN_ROLE` | Multisig | Manage agents, swap KYC provider |
| `UPGRADER_ROLE` | Timelock | UUPS implementation upgrade |
| `AGENT_ROLE` | Operations multisig + `TokenBridge` | `mint`, `burn`, `freezeAddress`, `freezePartialTokens`, `unfreeze`, `forcedTransfer`, `pause` |

### Transfer Rules

Every standard `transfer` / `transferFrom` checks:

1. `!paused` (global)
2. `!_frozen[from] && !_frozen[to]` (per-address freeze)
3. `_balances[from] - _frozenTokens[from] ≥ amount` (partial freeze)
4. `kycProvider.isVerified(to) == true` (KYC on receiver)

If any check fails the transfer reverts. Agents can bypass these via `forcedTransfer` (audited path for compliance-mandated movements).

### Mint / Burn

Only `AGENT_ROLE` can mint. Mint is the path used by:

- Initial issuance (operations multisig mints supply against off-chain SPV documentation)
- `TokenBridge.redeem` flow (bridge holds pSPCX in escrow; conceptually it does not need to mint, but the agent role allows operational adjustments)

Burn (`burnFrom`) likewise requires `AGENT_ROLE`.

## ShadowERC20 Contract (xSPCX)

**Source**: `src/eip3643/ShadowERC20.sol` (UUPS upgradeable)
**Mainnet**: xSPCX `0x05353Dabf163Fb2fec87f9e0f00f94Eae4AC1631`

The shadow contract is intentionally minimal:

| Behaviour | Implementation |
|-----------|----------------|
| ERC-20 standard interface | Plain implementation, no KYC check anywhere |
| Mint / burn | **Restricted to `bridge` address only** via `onlyBridge` modifier |
| `bridge` setter | `ADMIN_ROLE` only, used at deployment + bridge upgrades |
| Total supply | Updated only by bridge mint/burn → guaranteed equal to escrowed pSPCX × ratio |

Because `mint` and `burn` are gated to a single contract, the supply is always reconcilable against `TokenBridge` state. Every xSPCX in circulation is backed 1:N by a pSPCX locked in the bridge.

## TokenBridge Contract

**Source**: `src/eip3643/TokenBridge.sol` (UUPS upgradeable)
**Deployment**: live on BSC mainnet (operational; address not enumerated)

A single bridge instance can manage `N` pairs. Each pair stores:

| Field | Purpose |
|-------|---------|
| `securityToken` | Address of the EIP-3643 token (e.g. pSPCX) |
| `shadowToken` | Address of the corresponding ShadowERC20 (e.g. xSPCX) |
| `ratio` | Mint/burn ratio (currently `10` for pSPCX/xSPCX) |
| `lockedAmount` | Amount of security token currently held in escrow |

Lookup is bidirectional: `getShadowToken[security] → shadow` and `pairs[shadow] → Pair{ security, ratio, locked }`.

### deposit (security → shadow)

```
user holds pSPCX
  │
  │  TokenBridge.deposit(shadowToken=xSPCX, amount=1)
  ▼
Bridge calls pSPCX.forcedTransfer(user → bridge, 1)
  │   (bridge has AGENT_ROLE on pSPCX)
  ▼
Bridge updates pairs[xSPCX].lockedAmount += 1
  │
  ▼
Bridge calls xSPCX.mint(user, 1 × ratio = 10)
  │
  ▼
emit Deposited(xSPCX, user, 1, 10)
```

The user must already hold pSPCX and therefore must already have passed KYC at issuance time. No KYC check is added here because the source token is itself KYC-restricted.

### redeem (shadow → security)

```
user holds 10 xSPCX
  │
  │  TokenBridge.redeem(shadowToken=xSPCX, shadowAmount=10)
  ▼
Bridge calls kycProvider.isVerified(msg.sender)  ← REQUIRED, reverts if false
  │
  ▼
Bridge calls xSPCX.burn(user, 10)
  │
  ▼
Bridge updates pairs[xSPCX].lockedAmount -= 1
  │
  ▼
Bridge calls pSPCX.transfer(bridge → user, 1)
  │   (this calls pSPCX's transfer which itself checks isVerified(user))
  ▼
emit Redeemed(xSPCX, user, 10, 1)
```

The KYC check in `redeem` is essential: it is the single gate that stops a non-KYC holder of xSPCX from "back-doering" into a security position. Even though xSPCX itself is freely transferable, exiting back into pSPCX requires an identity check.

## Invariants

The system maintains three contract-enforced invariants:

| Invariant | How enforced |
|-----------|--------------|
| `totalSupply(xSPCX) == bridge.locked(pSPCX) × ratio` | Bridge is the sole mint/burn authority on xSPCX; bridge updates `lockedAmount` atomically with mint/burn |
| Every pSPCX transfer to a non-zero address has a KYC-verified receiver | Hard-coded check in `EIP3643Token.transfer` / `transferFrom` |
| Every xSPCX → pSPCX redemption has a KYC-verified sender | Hard-coded check in `TokenBridge.redeem` |

A regulator or auditor can verify these by indexing `Deposited` / `Redeemed` / `Mint` / `Burn` / `Transfer` events.

## Operational Privileges and Their Limits

EIP-3643 grants agents real powers — this is intentional and required for compliance use cases — and the following table lists how Paimon constrains them:

| Power | Held by | Constrained by |
|-------|---------|----------------|
| `pause` (halt all transfers) | `AGENT_ROLE` | Multisig + emergency-only operating procedure; emits `Paused(agent)` event |
| `freezeAddress(user, true)` | `AGENT_ROLE` | Only used for sanctions / court-order compliance; events emitted |
| `freezePartialTokens(user, amount)` | `AGENT_ROLE` | Used for unvested-tranche enforcement; emits `TokensFrozen` |
| `forcedTransfer(from, to, amount)` | `AGENT_ROLE` | Bridge uses this for the deposit path; ops multisig uses for compliance remediation; receiver KYC still checked |
| `mint(to, amount)` | `AGENT_ROLE` | Receiver KYC still checked |
| `setKYCProvider(newProvider)` | `ADMIN_ROLE` | Multisig only; would invalidate any pending non-verified users until re-verified |
| Implementation upgrade | `UPGRADER_ROLE` | Timelock + multisig |

Two design choices are worth noting:

1. **The bridge has `AGENT_ROLE` on pSPCX.** This is what lets `deposit` use `forcedTransfer` to move pSPCX from the user into bridge escrow without an explicit `approve` step. The bridge itself only ever does this in response to a user-initiated `deposit` call, so the trust assumption is just "the bridge contract is correct" — not a human-mediated agent.
2. **Receiver-only KYC, not sender-only.** Standard EIP-3643 implementations check `isVerified(to)` rather than `isVerified(from)`, on the theory that anyone who already holds the token must have passed KYC to receive it. Paimon follows this convention. The redeem path adds an extra `isVerified(msg.sender)` to harden against the attack where a sender's KYC has been revoked since they received the token.

## Roadmap to Multi-Asset

Adding a second pre-IPO asset (e.g. an Anthropic SPV → `pPNTC` / `xPNTC`) requires:

1. Deploy a new `EIP3643Token` proxy with the new name/symbol, pointing at the existing `KYCAggregator`
2. Deploy a new `ShadowERC20` proxy
3. Call `TokenBridge.createPair(newSecurity, newShadow, ratio)` from `OPERATOR_ROLE`
4. Grant `AGENT_ROLE` on the new security to the bridge; set `bridge` as the bridge address on the new shadow

No KYC re-onboarding is required because the same `KYCAggregator` whitelist applies to all assets in the bridge.
