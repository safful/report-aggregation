## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- resolved

# [improve readability of 1000000 ](https://github.com/code-423n4/2021-06-realitycards-findings/issues/8) 

# Handle

gpersoon


# Vulnerability details

## Impact
The number 1000000  is used in the constructor of RCTreasury.sol. This is difficult to read in a glance.
Solidity allows the use of an underscore ( _ ) to make numbers more readable.

## Proof of Concept
https://github.com/code-423n4/2021-06-realitycards/blob/main/contracts/RCTreasury.sol#L114
setMaxContractBalance(1000000 ether); // 1m

## Tools Used

## Recommended Mitigation Steps

Replace 1000000 with 1_000_000


