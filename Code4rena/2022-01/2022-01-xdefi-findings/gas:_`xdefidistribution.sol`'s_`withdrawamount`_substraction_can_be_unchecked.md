## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Gas: `XDEFIDistribution.sol`'s `withdrawAmount` substraction can be unchecked](https://github.com/code-423n4/2022-01-xdefi-findings/issues/49) 

# Handle

Dravee


# Vulnerability details

## Impact  
Waste of gas due to unnecessary underflow checks

## Proof of Concept  
On `XDEFIDistribution.sol:120` and `XDEFIDistribution.sol:175`, you can find the following substraction:
`uint256 withdrawAmount = amountUnlocked_ - lockAmount_;`

However, as the Solidity version is 0.8.10, default overflow and underflow checks are made, which cost some gas.

You can save this gas with the `unchecked` keyword to bypass these checks as 5 lines above (L115 and L170), a `require` statement already checks that `lockAmount_ <= amountUnlocked_`. 

Therefore, no underflow is possible.

## Tools Used  
VS Code  
  
## Recommended Mitigation Steps  
Use the "unchecked" keyword 

