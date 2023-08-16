## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- ERC20Rewards
- Strategy

# [Changing function visibility from public to external saves gas](https://github.com/code-423n4/2021-08-yield-findings/issues/42) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Public functions need to copy their arguments from calldata to memory resulting in more bytecode and gas consumption. If functions are never called from within the contracts, they can be declared external in which case their parameters are always in calldata without being copied to memory. This results in gas savings.
There are many such public functions that don’t appear to be called from within the contract.

## Proof of Concept

https://github.com/code-423n4/2021-08-yield/blob/4dc46470e616dd0cbd9db9b4742e36c4d809e02c/contracts/utils/token/ERC20Rewards.sol#L74-L75

https://github.com/code-423n4/2021-08-yield/blob/4dc46470e616dd0cbd9db9b4742e36c4d809e02c/contracts/yieldspace/Strategy.sol#L100-L101

All functions declared in this range:
https://github.com/code-423n4/2021-08-yield/blob/4dc46470e616dd0cbd9db9b4742e36c4d809e02c/contracts/yieldspace/Strategy.sol#L127-L252

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Change function visibility from public to external 

