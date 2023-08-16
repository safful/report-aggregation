## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Upgrade pragma to at least 0.8.4](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/29) 

# Handle

defsec


# Vulnerability details

## Impact

Using newer compiler versions and the optimizer gives gas optimizations
and additional safety checks are available for free.

The advantages of versions 0.8.* over <0.8.0 are:

- Safemath by default from 0.8.0 (can be more gas efficient than
library based safemath.)
- Low level inliner : from 0.8.2, leads to cheaper runtime gas. Especially relevant when the contract has small functions. For example, OpenZeppelin libraries typically have a lot of small helper functions and if they are not inlined, they cost an additional 20 to 40 gas because of 2 extra jump instructions and additional stack operations needed for function calls.
- Optimizer improvements in packed structs: Before 0.8.3, storing packed structs, in some cases used an
additional storage read operation. After EIP-2929, if the slot was already cold, this means unnecessary stack operations and extra deploy time costs. However, if the slot was already warm, this means
additional cost of 100 gas alongside the same unnecessary stack operations and extra deploy time costs.
- Custom errors from 0.8.4, leads to cheaper deploy time cost and run time cost. Note: the run time cost is only relevant when the revert condition is met. In short, replace revert strings by custom errors.



## Proof of Concept

1. The contest repository contracts contain pragma 0.6.12^. The contracts pragma version should be updated to 0.8.4.

(https://github.com/code-423n4/2021-11-bootfinance/blob/main/customswap/contracts/Swap.sol )
(https://github.com/code-423n4/2021-11-bootfinance/blob/main/customswap/contracts/SwapUtils.sol#L3)
(https://github.com/code-423n4/2021-11-bootfinance/blob/main/customswap/contracts/MathUtils.sol#L3)
(https://github.com/code-423n4/2021-11-bootfinance/blob/main/tge/contracts/PublicSale.sol#L2)
(https://github.com/code-423n4/2021-11-bootfinance/blob/main/tge/contracts/PublicSaleBatchWithdraw.sol#L2)

## Tools Used

None

## Recommended Mitigation Steps

Consider to upgrade pragma to at least 0.8.4.

