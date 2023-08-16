## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- Strategy

# [Unused cauldron_ parameter](https://github.com/code-423n4/2021-08-yield-findings/issues/54) 

# Handle

0xRajeev


# Vulnerability details

## Impact

That cauldron_ parameter is not used here and ladle_.cauldron() is used instead. The Ladle constructor initializes its cauldron value and so the only way this could differ from the parameter is if the argument to this function is specified incorrectly.

## Proof of Concept
https://github.com/code-423n4/2021-08-yield/blob/4dc46470e616dd0cbd9db9b4742e36c4d809e02c/contracts/yieldspace/Strategy.sol#L100

https://github.com/code-423n4/2021-08-yield/blob/4dc46470e616dd0cbd9db9b4742e36c4d809e02c/contracts/yieldspace/Strategy.sol#L106-L107

https://github.com/code-423n4/2021-08-yield/blob/4dc46470e616dd0cbd9db9b4742e36c4d809e02c/contracts/Ladle.sol#L33


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Either use parameter or remove it in favor of the value from ladle_.cauldron().

