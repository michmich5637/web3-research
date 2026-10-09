# Security review: `michmich56` ERC-20 token

| | |
| --- | --- |
| **Date** | 2026-10 |
| **Target** | [michmich5637/challenge-scroll](https://github.com/michmich5637/challenge-scroll) |
| **Scope** | Contract `michmich56` (the flattened OpenZeppelin code is out of scope) |
| **Compiler** | Solidity `^0.8.24` |
| **Method** | Manual review |

## Summary

`michmich56` is an ERC-20 token that inherits from OpenZeppelin Contracts v5 `ERC20` and adds public `mint` and `burn` functions. The inherited code is sound. The added code removes every guarantee a token holder normally relies on: supply and balances can be changed by anyone.

The contract was written as a learning exercise and is not meant to hold value. It must not be deployed as is.

| Severity | Count |
| --- | --- |
| Critical | 2 |
| High | 0 |
| Medium | 0 |
| Low | 1 |
| Informational | 3 |

Severity follows the [classification](../methodology/severity-classification.md) used in this repository.

## System overview

- The constructor mints `initialSupply` to the deployer.
- `mint(to, amount)` and `burn(from, amount)` call the internal `_mint` and `_burn`.
- `transfer`, `approve` and `transferFrom` are overridden.
- `getBalanceOf` wraps `balanceOf`.

There is no owner, no role and no privileged actor.

## Findings

### [C-01] Anyone can mint an unlimited amount of tokens

**Location:** `michmich56.mint()`

**Description:** `mint` is `public` and has no access control. It forwards its arguments to `_mint`.

```solidity
function mint(address to, uint256 amount) public {
    _mint(to, amount);
}
```

**Impact:** Any account can mint any amount to itself. The supply is unbounded and the token cannot hold value. If the token were paired in a liquidity pool, an attacker would mint and drain the other side of the pool.

**Recommendation:** Restrict `mint` to a privileged role with `Ownable` or `AccessControl`, or remove it and fix the supply in the constructor. Consider `ERC20Capped` if a maximum supply is intended.

### [C-02] Anyone can burn tokens from any holder

**Location:** `michmich56.burn()`

**Description:** `burn` is `public`, takes an arbitrary `from` address and requires neither ownership nor allowance.

```solidity
function burn(address from, uint256 amount) public {
    _burn(from, amount);
}
```

**Impact:** Any account can destroy the full balance of any holder, including balances held by pools, vaults or bridges. Funds are lost permanently.

**Recommendation:** Use OpenZeppelin `ERC20Burnable`: `burn(amount)` acts on the caller's balance and `burnFrom(account, amount)` spends an allowance.

### [L-01] `transferFrom` override changes the standard allowance behaviour

**Location:** `michmich56.transferFrom()`

**Description:** The override moves the tokens first, then checks the allowance, then writes the new allowance with the event-emitting `_approve`.

Three differences with the inherited implementation follow:

1. An allowance of `type(uint256).max` is decremented. OpenZeppelin treats it as infinite and leaves it untouched, and many integrations rely on that.
2. The failure reverts with a string instead of the `ERC20InsufficientAllowance` custom error defined by ERC-6093, which the rest of the contract uses.
3. An `Approval` event is emitted on every `transferFrom`, which costs gas and differs from OpenZeppelin v5.

The transfer-before-check order is not exploitable because the revert undoes the transfer, but it goes against the checks-effects-interactions pattern.

**Impact:** No funds at risk. Integrations that expect infinite approvals to persist will need new approvals over time, and off-chain tooling that decodes custom errors will not recognise the failure.

**Recommendation:** Delete the override and keep the inherited `transferFrom`.

### [I-01] Redundant overrides

`transfer` and `approve` reproduce the inherited behaviour exactly, and `getBalanceOf` duplicates `balanceOf`. They add bytecode and review surface without adding features. Remove them.

### [I-02] Floating pragma

`pragma solidity ^0.8.24` allows any later 0.8.x compiler. Pin the exact version used for deployment and set the EVM version explicitly for the target chain.

### [I-03] Flattened source without a project structure

The repository contains a single flattened file. Dependencies cannot be updated or verified against their upstream version, and there are no tests. Move to a Foundry or Hardhat project, import OpenZeppelin as a dependency and add tests for every finding above.

## Out of scope

The flattened OpenZeppelin Contracts v5 code was read for context and not reviewed. Deployment parameters and any deployed instance were not reviewed.

## Disclaimer

This review is a time-boxed assessment. It does not guarantee the absence of vulnerabilities.
