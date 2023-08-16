## Tags

- bug
- sponsor confirmed
- 0 (Non-critical)

# [Follow Curve's convention: `_getYD` and `_getY`](https://github.com/code-423n4/2021-09-sushitrident-findings/issues/156) 

# Handle

0xsanson


# Vulnerability details

## Impact
In HybridPool.sol, functions `_getY` and `_getYD` are identical (only differences are variables' names), and only `_getY` is used in the contract.

Since these functions are supposed to mimic those of Curve, it would make more sense to follow their naming conventions.
In particular, `_getYD` correctly mimics Curve's `_get_y_D` ([ref](https://github.com/curvefi/curve-contract/blob/master/contracts/pool-templates/base/SwapTemplateBase.vy#L614)), while `_getY` does not mimic Curve's `_get_y` ([ref](https://github.com/curvefi/curve-contract/blob/master/contracts/pool-templates/base/SwapTemplateBase.vy#L379)).

## Recommended Mitigation Steps
Consider eliminating `_getY` and using `_getYD` instead in the contract.

