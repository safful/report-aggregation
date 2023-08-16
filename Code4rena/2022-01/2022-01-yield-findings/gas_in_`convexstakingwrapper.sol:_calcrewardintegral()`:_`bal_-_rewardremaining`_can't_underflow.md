## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas in `ConvexStakingWrapper.sol:_calcRewardIntegral()`: `bal - rewardRemaining` can't underflow](https://github.com/code-423n4/2022-01-yield-findings/issues/67) 

# Handle

Dravee


# Vulnerability details

## Impact
Increased gas cost due to unnecessary automatic underflow checks.

Solidity version 0.8+ comes with implicit overflow and underflow checks on unsigned integers.

When an overflow or an underflow isn't possible (as an example, when a comparison is made before the arithmetic operation, or the operation doesn't depend on user input), some gas can be saved by using an `unchecked` block.

https://docs.soliditylang.org/en/v0.8.10/control-structures.html#checked-or-unchecked-arithmetic

## Proof of Concept
In `ConvexStakingWrapper.sol:_calcRewardIntegral()`, `bal - rewardRemaining` can't underflow at line 222 as the conditional statement line 221 prevents it:
```
File: ConvexStakingWrapper.sol
221:         if (_supply > 0 && (bal - rewardRemaining) > 0) {
222:             rewardIntegral = uint128(rewardIntegral) + uint128(((bal - rewardRemaining) * 1e20) / _supply); //@audit-info (bal - rewardRemaining) can't underflow because of above if statement
```
This substraction should get computed inside an `unchecked` block and stored in a variable, which would then be used in the checked calculation for `rewardIntegral`.

## Tools Used
VS Code

## Recommended Mitigation Steps
Uncheck arithmetic operations when the risk of underflow or overflow is already contained by wrapping them in an `unchecked` block



