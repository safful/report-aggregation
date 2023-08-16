## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- resolved

# [Unchecking the ownership of `mph` in function `distributeFundingRewards` could cause several critical functions to revert](https://github.com/code-423n4/2021-05-88mph-findings/issues/23) 

# Handle

shw


# Vulnerability details

## Impact

In contract `MPHMinter`, the function `distributeFundingRewards` does not check whether the contract itself is the owner of `mph`. If the contract is not the owner of `mph`, `mph.ownerMint` could revert, causing functions such as `withdraw`, `rolloverDeposit`, `payInterestToFunders` in the contract `DInterest` to revert as well.

## Proof of Concept

Referenced code:
[MPHMinter.sol#L121](https://github.com/code-423n4/2021-05-88mph/blob/main/contracts/rewards/MPHMinter.sol#L121)
[DInterest.sol#L1253](https://github.com/code-423n4/2021-05-88mph/blob/main/contracts/DInterest.sol#L1253)
[DInterest.sol#L1420](https://github.com/code-423n4/2021-05-88mph/blob/main/contracts/DInterest.sol#L1420)

## Tools Used

None

## Recommended Mitigation Steps

Add a `mph.owner() != address(this)` check as in the other functions (e.g., `mintVested`).

