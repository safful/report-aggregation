## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Evaluate security benefit vs gas usage trade-off for using nonreentrant modifier on different functions](https://github.com/code-423n4/2021-07-connext-findings/issues/43) 

# Handle

0xRajeev


# Vulnerability details

## Impact

While it may be considered extra-safe to have a nonreentrant modifier on all functions making any external calls even though they are to trusted contracts, when functions implement Checks-Effects-Interactions (CEI) pattern, it is helpful to evaluate the perceived security benefit vs gas usage trade-off for using nonreentrant modifier.

Functions adhering to the CEI pattern may consider not having the nonreentrant modifier which does two SSTORES (getting more expensive with the London fork EIP-3529) to its _status state variable. 

Example 1: In addLiquidity(), by moving the updating of router balance on L101 to before the transfers from L92, the function would adhere to CEI pattern and could be evaluated to remove the nonreentrant modifier.

Example 2: removeLiquidity() already adheres to CEI pattern and could be evaluated to remove the nonreentrant modifier.

prepare() can be slightly restructured to follow CEI pattern as well. However, fulfill() and cancel() are risky with multiple external calls and its safer to leave the nonreentrant call at the expense of additional gas costs.

Impact: Save gas by removing nonreentrant modifier if function is deemed to be reentrant safe. This can save gas costs of 2 SSTORES per function call that uses this modifier: _status SSTORE from 1 to 2 costs 5000 and _status SSTORE from 2 to 1 which costs 100 (because it was already accessed) which is significant at 5100 per call post-Berlin EIP-2929.


## Proof of Concept

https://github.com/code-423n4/2021-07-connext/blob/8e1a7ea396d508ed2ebeba4d1898a748255a48d2/contracts/TransactionManager.sol#L92-L101

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Evaluate security benefit vs gas usage trade-off for using nonreentrant modifier on functions that may already be reentrant safe or do not need this protection. It may indeed be safe to leave this modifier (while accepting the gas impact) if such an evaluation is tricky or depends on assumptions.

