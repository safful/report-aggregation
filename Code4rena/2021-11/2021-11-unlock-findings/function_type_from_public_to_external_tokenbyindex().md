## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Function type from public to external tokenByIndex()](https://github.com/code-423n4/2021-11-unlock-findings/issues/76) 

# Handle

BouSalman


# Vulnerability details

## Vulnerability Description
Some of the implemented functions inside the smart contracts are of type Public, However these functions are not used within the contracts. The function **tokenByIndex()** is part of the EIP721 which define it as external function.

## Impact
Coding style quality.

## Proof of Concept
https://github.com/code-423n4/2021-11-unlock/blob/ec41eada1dd116bcccc5603ce342257584bec783/smart-contracts/contracts/mixins/MixinERC721Enumerable.sol#L35

## Tools Used
manual code review.

## Recommended Mitigation Steps
Change the function to external and follow the ERC721 Specs when implementing: https://eips.ethereum.org/EIPS/eip-721#specification

