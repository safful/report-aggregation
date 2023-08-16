## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Use of uint256 parameter instead of bool](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/112) 

# Handle

Reigada


# Vulnerability details

## Impact
In the contract Vesting, function vest(), the parameter _isRevocable is declared as an uint256 when it is used as a boolean.

## Proof of Concept
https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/Vesting.sol#L77

## Tools Used
Manual review

## Recommended Mitigation Steps
Declare the parameter _isRevocable as a bool.

