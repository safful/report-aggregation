## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- ERC20Rewards

# [Missing input validation to check that end > start](https://github.com/code-423n4/2021-08-yield-findings/issues/49) 

# Handle

0xRajeev


# Vulnerability details

## Impact

setRewards() is missing input validation on parameters start and end to check if end > start. If accidentally set incorrectly, this will allow resetting new rewards while there is an ongoing one.


## Proof of Concept

https://github.com/code-423n4/2021-08-yield/blob/4dc46470e616dd0cbd9db9b4742e36c4d809e02c/contracts/utils/token/ERC20Rewards.sol#L74-L88

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Add a require() to check that end > start.

