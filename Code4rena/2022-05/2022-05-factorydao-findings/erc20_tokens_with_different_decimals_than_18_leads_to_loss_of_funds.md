## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [ERC20 tokens with different decimals than 18 leads to loss of funds](https://github.com/code-423n4/2022-05-factorydao-findings/issues/47) 

# Lines of code

https://github.com/code-423n4/2022-05-factorydao/blob/db415804c06143d8af6880bc4cda7222e5463c0e/contracts/PermissionlessBasicPoolFactory.sol#L169
https://github.com/code-423n4/2022-05-factorydao/blob/db415804c06143d8af6880bc4cda7222e5463c0e/contracts/PermissionlessBasicPoolFactory.sol#L282


# Vulnerability details

## Impact
Contract `PermissionlessBasicPoolFactory` calculates rewards by using hardcoded value of decimals `18` (1e18) for ERC20 tokens. This leads to wrong rewards calculations and effectively loss of funds for all pools that will be using ERC20 tokens with different decimals than `18`. Example of such a token is USDC that has 6 decimals only.

## Proof of Concept
* https://github.com/code-423n4/2022-05-factorydao/blob/db415804c06143d8af6880bc4cda7222e5463c0e/contracts/PermissionlessBasicPoolFactory.sol#L169
* https://github.com/code-423n4/2022-05-factorydao/blob/db415804c06143d8af6880bc4cda7222e5463c0e/contracts/PermissionlessBasicPoolFactory.sol#L282

## Tools Used
Manual Review / VSCode

## Recommended Mitigation Steps
It is recommended to add support for different number of decimals than `18` by dynamically checking `decimals()` for the tokens that are part of the rewards calculations. Alternatively if such a support is not needed, new require statements should be added to `addPool` that will be checking that the number of decimals for all ERC20 tokens is `18`.

