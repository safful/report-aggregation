## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Self transfer can lead to unlimited mint](https://github.com/code-423n4/2021-08-notional-findings/issues/1) 

# Handle

Omik


# Vulnerability details

## Impact
The implementation of the transfer function in the https://github.com/code-423n4/2021-08-notional/blob/main/contracts/external/actions/nTokenAction.sol is the different from the usual erc20 token transfer function, this happen because it count the incentive that the user get, but the with self tranfer it can lead to unlimited mint, because https://github.com/code-423n4/2021-08-notional/blob/main/contracts/external/actions/nTokenAction.sol#L278 it makes the amount to negative, but in the https://github.com/code-423n4/2021-08-notional/blob/main/contracts/external/actions/nTokenAction.sol#L279 it return the value to amount that not negative, so in the https://github.com/code-423n4/2021-08-notional/blob/main/contracts/external/actions/nTokenAction.sol#L281-282 it finalize the positive value only since the negative value is change to the positive value, you can interact this transfer function through https://github.com/code-423n4/2021-08-notional/blob/main/contracts/external/adapters/nTokenERC20Proxy.sol

## Proof of Concept
Provide direct links to all referenced code in GitHub. Add screenshots, logs, or any other relevant proof that illustrates the concept.

## Tools Used
Manual

## Recommended Mitigation Steps
add (sender != recipient)

