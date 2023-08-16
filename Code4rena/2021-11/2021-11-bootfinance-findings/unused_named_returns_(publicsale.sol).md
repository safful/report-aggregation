## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Unused Named Returns (PublicSale.sol)](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/25) 

# Handle

ye0lde


# Vulnerability details

## Impact

Removing unused named return variables can reduce gas usage and improve code clarity.

## Proof of Concept

https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/tge/contracts/PublicSale.sol#L187
https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/tge/contracts/PublicSale.sol#L194
https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/tge/contracts/PublicSale.sol#L231

## Tools Used
Visual Studio Code, Remix

## Recommended Mitigation Steps
Remove the unused named returns

