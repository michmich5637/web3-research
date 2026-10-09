# Security review: `SimpleTokenSwap`

| | |
| --- | --- |
| **Date** | 2026-10 |
| **Target** | [michmich5637/tokenswapsimple.sol](https://github.com/michmich5637/tokenswapsimple.sol) |
| **Scope** | Contract `SimpleTokenSwap` (the flattened Uniswap and OpenZeppelin interfaces are out of scope) |
| **Compiler** | Solidity `^0.8.24` |
| **Method** | Manual review |

## Summary

`SimpleTokenSwap` wraps a single-hop Uniswap V3 swap. It pulls `tokenIn` from the caller, approves the router and calls `exactInputSingle`. The contract holds no funds between transactions and has no privileged actor, which limits the damage a bug can do.

No finding lets a third party take funds directly. The main issues are a deadline that protects nothing and token handling that fails with widely used tokens.

The contract was written as a learning exercise and must not be used in production as is.

| Severity | Count |
| --- | --- |
| Critical | 0 |
| High | 0 |
| Medium | 2 |
| Low | 3 |
| Informational | 4 |

Severity follows the [classification](../methodology/severity-classification.md) used in this repository.

## System overview

- The constructor stores the router address and a WETH address.
- `swap(tokenIn, tokenOut, amountIn, amountOutMinimum, recipient)` transfers `amountIn` from the caller, approves the router for `amountIn` and swaps through the 0.3% pool.

## Trust assumptions

- The router address set at deployment is the genuine Uniswap V3 `SwapRouter`. A malicious router would receive an approval and take the input tokens.
- The caller computes `amountOutMinimum` off-chain.

## Findings

### [M-01] The deadline is always satisfied

**Location:** `SimpleTokenSwap.swap()`

**Description:** The swap is sent with `deadline: block.timestamp`. The router checks `block.timestamp <= deadline` in the same transaction, so the check always passes.

**Impact:** A transaction that stays in the mempool can be included much later, at a time chosen by whoever orders transactions. The caller is still protected by `amountOutMinimum`, but a minimum computed for an old price can be far below the current fair value, and the difference can be extracted.

**Recommendation:** Add a `deadline` parameter set by the caller and pass it to the router.

### [M-02] Incompatible with tokens that do not return a boolean

**Location:** `SimpleTokenSwap.swap()`

**Description:** The contract wraps `transferFrom` and `approve` in `require(...)` through the `IERC20` interface, which expects a `bool` return value. Some widely used tokens, USDT on Ethereum mainnet being the best known, return nothing. Decoding the missing return value reverts.

**Impact:** Every swap whose input is such a token reverts. No funds are lost, but a core use case does not work.

**Recommendation:** Use OpenZeppelin `SafeERC20`: `safeTransferFrom` for the pull and `forceApprove` for the approval.

### [L-01] A zero `recipient` sends the output to the router

**Location:** `SimpleTokenSwap.swap()`

**Description:** `recipient` is not validated. The Uniswap V3 `SwapRouter` treats `address(0)` as "keep the output in the router", a mode meant to be combined with a later `sweepToken` or `unwrapWETH9` call in the same multicall. This contract makes no such call.

**Impact:** A caller who passes `address(0)` leaves the output tokens in the router, where anyone can sweep them.

**Recommendation:** Revert when `recipient == address(0)`.

### [L-02] Hard-coded fee tier

**Location:** `SimpleTokenSwap.swap()`

**Description:** `fee` is fixed to `3000`. Pairs without a 0.3% pool cannot be swapped, and when the deepest liquidity sits in another tier the caller gets a worse price.

**Recommendation:** Make `fee` a parameter.

### [L-03] Fee-on-transfer tokens are not supported

**Location:** `SimpleTokenSwap.swap()`

**Description:** The contract assumes it receives exactly `amountIn`. With a token that takes a fee on transfer it receives less, then asks the router to spend `amountIn`, and the swap reverts.

**Recommendation:** Document the limitation, or measure the balance before and after the pull and swap the amount actually received.

### [I-01] Redundant output check

`require(amountOut >= amountOutMinimum)` repeats a check the router already performs. Remove it.

### [I-02] Unused and mutable state

`WETH` is stored and never read. `swapRouter` and `WETH` are set once and can be `immutable`, which saves a storage read per swap.

### [I-03] Interface tied to the original `SwapRouter`

The `ExactInputSingleParams` struct includes `deadline`, which matches the original `SwapRouter`. `SwapRouter02` uses a struct without that field, so its function selector differs. On networks where only `SwapRouter02` is deployed, this contract cannot be used without changing the interface.

### [I-04] No events, no tests, flattened source

The contract emits no event, which makes swaps hard to index. The repository has no tests and inlines its dependencies. Move to a Foundry project, import dependencies and add fork tests against a real pool.

## Out of scope

The Uniswap V3 and OpenZeppelin interfaces were read for context and not reviewed. No deployed instance was reviewed.

## Disclaimer

This review is a time-boxed assessment. It does not guarantee the absence of vulnerabilities.
