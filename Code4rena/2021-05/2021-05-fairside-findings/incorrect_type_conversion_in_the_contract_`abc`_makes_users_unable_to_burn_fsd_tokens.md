## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- resolved

# [Incorrect type conversion in the contract `ABC` makes users unable to burn FSD tokens](https://github.com/code-423n4/2021-05-fairside-findings/issues/77) 

# Handle

shw


# Vulnerability details

## Impact

The function `_calculateDeltaOfFSD` of contract `ABC` incorrectly converts an `int256` type parameter, `_reserveDelta`, to `uint256` by explicit conversion, which in general results in an extremely large number when the provided parameter is negative. The extremely large number could cause a SafeMath operation `sub` at line 43 to revert, and thus the FSD tokens cannot be burned. (`_reserveDelta` is negative when burning FSD tokens)

## Proof of Concept

Simply calling `fsd.burn` after a successful `fsd.mint` will trigger this bug.

Referenced code:
[ABC.sol#L43](https://github.com/code-423n4/2021-05-fairside/blob/main/contracts/token/ABC.sol#L43)
[ABC.sol#L49](https://github.com/code-423n4/2021-05-fairside/blob/main/contracts/token/ABC.sol#L49)
[ABC.sol#L54](https://github.com/code-423n4/2021-05-fairside/blob/main/contracts/token/ABC.sol#L54)

## Recommended Mitigation Steps

Use the solidity function `abs` to get the `_reserveDelta` absolute value.

