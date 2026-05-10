# Pre-IPO SPV Tokens (pSPCX / xSPCX)

Paimon's second live product line tokenizes single-issuer alternative assets — currently a SpaceX SPV — into a **two-token structure** that satisfies both the legal-compliance side (KYC-gated security token) and the DeFi-liquidity side (freely transferable shadow ERC-20).

This is **live on BSC mainnet**.

## Two Tokens, One Underlying

| Token | Purpose | Standard | KYC required to transfer? | Mainnet address |
|-------|---------|----------|---------------------------|-----------------|
| `pSPCX` | **Permissioned** primary token — legal "security" wrapper of the SPV | EIP-3643 | ✅ Yes | `0x6DC9a487bF8Fd047e41AB336003AE6e4FE602646` |
| `xSPCX` | **Tradable** shadow — DeFi-friendly mirror | Plain ERC-20 (UUPS) | ❌ No | `0x05353Dabf163Fb2fec87f9e0f00f94Eae4AC1631` |
| `TokenBridge` | The only mint/burn authority bridging the two | UUPS | — | (operational, not enumerated) |

The two tokens are **not interchangeable** through normal ERC-20 mechanics. They are linked exclusively through `TokenBridge`, which holds the invariant:

```
totalSupply(xSPCX) == TokenBridge.lockedAmount(pSPCX) × ratio
```

Today's pair runs at **ratio = 10**, i.e. depositing 1 pSPCX into the bridge mints 10 xSPCX, and burning 10 xSPCX releases 1 pSPCX (the latter only callable by KYC-verified addresses).

## Why a Two-Token Design

A single token cannot simultaneously satisfy two requirements:

1. **Regulatory**: securities tokenization typically requires KYC on every transfer (suitability, AML, sanctions). EIP-3643 adds enforcement primitives — `freeze`, `forcedTransfer`, agent-mediated mint/burn — that compliant SPVs need.
2. **DeFi composability**: AMMs, lending protocols, perpetual DEXes etc. cannot integrate a token that calls a KYC oracle on every `transfer()` — half the protocol surface area would revert for every counterparty in a swap.

The Paimon design splits these into two layers:

- **pSPCX** is the legal vehicle. It checks KYC on every transfer (`EIP3643Token.sol:11-15`), supports per-address freeze and partial-token freeze (`_frozenTokens`), and reserves `forcedTransfer` for the agent (used for compliance-mandated movements such as inheritance, court orders, or frozen-account remediation).
- **xSPCX** is a plain ERC-20 with a single special rule: **only the bridge can mint or burn** (`ShadowERC20.sol:8-13`). All other functions (`transfer`, `approve`, `transferFrom`, etc.) behave like a standard ERC-20, so xSPCX integrates with any AMM or DeFi primitive without modification.

## How Holdings Move Between the Two

```
                ┌─────────────────────────────────────────┐
                │            User journey                 │
                └─────────────────────────────────────────┘

   Buy on Launchpad      Deposit            Trade / use in DeFi
       (USDT)        ──────────────►  pSPCX  ──────► xSPCX  ──────► AMMs, lending,
                     (KYC required)    │              │              perp markets,
                                       │              │              transfers to
                                       │              │              non-KYC wallets
                                       │              │
                                       │              │  Redeem
                                       │  ◄───────────┘  (KYC required)
                                       │
                                       ▼
                                  Held by KYC user
                                  (claim against SPV)
```

| Direction | Caller requirement | What happens |
|-----------|-------------------|--------------|
| **Deposit** `pSPCX → xSPCX` | Source must be KYC-verified (was already, since they hold pSPCX) | `TokenBridge.deposit` calls `pSPCX.forcedTransfer(user → bridge)`, then `xSPCX.mint(user, amount × ratio)` |
| **Redeem** `xSPCX → pSPCX` | Caller must be KYC-verified by `KYCAggregator` | `TokenBridge.redeem` burns `xSPCX` from caller, then `pSPCX.transfer(bridge → caller)` |
| **Plain transfer of xSPCX** | None | Standard ERC-20, behaves normally |
| **Plain transfer of pSPCX** | Receiver must be KYC-verified | EIP-3643 enforced; reverts otherwise |

For details on the EIP-3643 contract, the KYCAggregator, agent permissions and the bridge invariant proofs, see [EIP-3643 Compliance Architecture](../architecture/eip3643-compliance.md).

## Lifecycle of a Drop

Pre-IPO SPV tokens are issued through the [Launchpad](../architecture/launchpad.md) using a 4-Layer Drop. Below is the full path a real holder takes, illustrated against the **xSPCX V4 drop** (April 2026).

### 1. Mint at issuance (Launchpad)

| Step | Where | Requires |
|------|-------|----------|
| Earn points pre-drop | PointsHubV2 (staking, social tasks, referrals) | Wallet only |
| Commit USDT in commitment window | `LaunchpadDrop.commit(layerIdx, usdt)` | Wallet + sufficient points for chosen layer + KYC if buying pSPCX directly |
| Receive allocation after settlement | `claimAllocation(dropId, layerIdx, proof)` | Merkle proof |

For the xSPCX V4 drop, allocations were distributed as **xSPCX directly** (the drop minted into the tradable shadow), allowing post-drop secondary trading without a KYC check. The drop layers ranged from a **60.32 % discount at $2.75** (Pioneer, requires 3,000+ points) down to **0 % discount at $6.93** (Open, no points required).

### 2. Hold or trade (xSPCX)

A retail user who only wants exposure to the SpaceX SPV exposure curve — without going through KYC — stays in **xSPCX** indefinitely. They can:

- Transfer xSPCX freely
- Sell on any compatible AMM
- Use as collateral in DeFi protocols that whitelist xSPCX

The protocol does **not** currently operate an official xSPCX/USDT pool; secondary liquidity is at the discretion of independent LPs.

### 3. Convert to pSPCX (institutions only)

KYC-verified holders — currently **institutional accounts only**, not retail — can call `TokenBridge.redeem(xSPCX, amount)` to burn xSPCX and receive pSPCX. This is the path required for any holder who wants to:

- Settle directly against the SPV (vote, receive distributions, claim equity-like rights)
- Maintain a regulatory-clean position record
- Move balances onto a custodian / off-exchange settlement venue

{% hint style="info" %}
**KYC scope**: The `SimpleKYCProvider` and `KYCAggregator` contracts are deployed and active on mainnet, but the KYC onboarding service is currently offered to **B2B / institutional partners only**. Retail wallets cannot self-serve KYC at present and should treat xSPCX as the user-facing token.
{% endhint %}

## Roles and Powers

| Role | On which contracts | What it can do |
|------|-------------------|----------------|
| `DEFAULT_ADMIN_ROLE` / `ADMIN_ROLE` | All | Set parameters, grant/revoke roles, swap KYC provider |
| `UPGRADER_ROLE` | All UUPS proxies | UUPS implementation upgrade (held by timelock) |
| `AGENT_ROLE` | `EIP3643Token` (pSPCX) | Mint, burn, `freeze`, `freezePartialTokens`, `forcedTransfer` |
| `OPERATOR_ROLE` | `TokenBridge` | `createPair`, parameter ops |
| KYC verification authority | `SimpleKYCProvider` agents | Add/remove users from the KYC whitelist used by `KYCAggregator` |
| `TokenBridge` | `pSPCX`, `xSPCX` | Has agent rights on pSPCX (for `forcedTransfer`) and is the only `bridge` allowed to `mint`/`burn` xSPCX |

## Relation to the Prime Vault (PP)

Pre-IPO SPV products are **a separate product line** from PP — they are *not* held inside the PP vault and they do not appear in PP's NAV calculation. A user can independently:

- Hold PP and xSPCX in the same wallet without interaction
- Subscribe to a Pre-IPO drop without touching PP
- Redeem PP without affecting xSPCX

The two products share infrastructure (Launchpad, Points, Badge, Treasury) but the asset accounting is fully isolated.

## Risk Disclosures

- **SPV-specific risk** — xSPCX value tracks the underlying SPV's claim on SpaceX equity, which is itself subject to SpaceX private-market valuation, secondary-sale frictions and SPV-administrator solvency.
- **KYC-gated redemption** — non-KYC holders can hold and trade xSPCX freely, but **cannot** redeem to pSPCX. If primary-market settlement against the SPV becomes the only liquidity venue, non-KYC holders will need to find a KYC counterparty to exit.
- **Bridge solvency** — the `TokenBridge` invariant is enforced on every deposit/redeem call; the contract is UUPS-upgradeable behind a multisig + timelock. An upgrade error or compromised admin key could in principle break the 1:N peg.
- **Regulatory change** — the EIP-3643 compliance layer is enforced on-chain, but securities-law treatment of the underlying SPV interest depends on the user's jurisdiction. This documentation is not legal advice.

See the broader [Risk Disclosure](../risk-and-transparency/README.md) for system-level risk handling.
