## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- ERC20Rewards
- Timelock

# [Using parameters or local variables instead of state variables in event emits can save gas](https://github.com/code-423n4/2021-08-yield-findings/issues/44) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Event emits where there are equivalent local variables or parameters for state variables can save gas by using those instead of state variables because of the expensive SLOADs.

## Proof of Concept

rewardsToken: https://github.com/code-423n4/2021-08-yield/blob/4dc46470e616dd0cbd9db9b4742e36c4d809e02c/contracts/utils/token/ERC20Rewards.sol#L97

delay: https://github.com/code-423n4/2021-08-yield/blob/4dc46470e616dd0cbd9db9b4742e36c4d809e02c/contracts/utils/TimeLock.sol#L51


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Use parameters or local variables instead of state variables in event emits

