## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- ERC20Rewards
- Strategy

# [Caching state variable in local variables for repeated reads saves gas by converting expensive SLOADs into much cheaper MLOADs](https://github.com/code-423n4/2021-08-yield-findings/issues/43) 

# Handle

0xRajeev


# Vulnerability details

## Impact

SLOADs cost 2100 gas for first time reads of state variables and then 100 gas for repeated reads in the context of a transaction (post Berlin fork). MLOADs cost 3 gas units. Therefore, caching state variable in local variables for repeated reads saves gas.

## Proof of Concept

Examples of state variables that are read at the lines shown and also later in that same function:

rewardsPeriod: https://github.com/code-423n4/2021-08-yield/blob/4dc46470e616dd0cbd9db9b4742e36c4d809e02c/contracts/utils/token/ERC20Rewards.sol#L80

_totalSupply: https://github.com/code-423n4/2021-08-yield/blob/4dc46470e616dd0cbd9db9b4742e36c4d809e02c/contracts/utils/token/ERC20Rewards.sol#L107

nextPool: https://github.com/code-423n4/2021-08-yield/blob/4dc46470e616dd0cbd9db9b4742e36c4d809e02c/contracts/yieldspace/Strategy.sol#L163

ladle: https://github.com/code-423n4/2021-08-yield/blob/4dc46470e616dd0cbd9db9b4742e36c4d809e02c/contracts/yieldspace/Strategy.sol#L172

base: https://github.com/code-423n4/2021-08-yield/blob/4dc46470e616dd0cbd9db9b4742e36c4d809e02c/contracts/yieldspace/Strategy.sol#L180

pool (600 gas savings): https://github.com/code-423n4/2021-08-yield/blob/4dc46470e616dd0cbd9db9b4742e36c4d809e02c/contracts/yieldspace/Strategy.sol#L183

pool: https://github.com/code-423n4/2021-08-yield/blob/4dc46470e616dd0cbd9db9b4742e36c4d809e02c/contracts/yieldspace/Strategy.sol#L208

cached: https://github.com/code-423n4/2021-08-yield/blob/4dc46470e616dd0cbd9db9b4742e36c4d809e02c/contracts/yieldspace/Strategy.sol#L262


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Consider caching state variables in local variables.

