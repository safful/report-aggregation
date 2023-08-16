## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- Resolved

# [External Call Made Before State Change](https://github.com/code-423n4/2021-08-realitycards-findings/issues/23) 

# Handle

leastwood


# Vulnerability details

## Impact
There are a number of functions in `RCTreasury.sol` which make external calls to another contract before updating the underlying market balances. More specifically, these affected functions are `deposit()`, `sponsor()`, and `topupMarketBalance()`. As a result, these functions would be prone to reentrancy exploits. However, as `safeTransferFrom()` operates on a trusted ERC20 token (RealityCard's token), this issue is of low severity.

## Proof of Concept
https://github.com/code-423n4/2021-08-realitycards/blob/main/contracts/RCTreasury.sol#L385-L391
https://github.com/code-423n4/2021-08-realitycards/blob/main/contracts/RCTreasury.sol#L561-L563
https://github.com/code-423n4/2021-08-realitycards/blob/main/contracts/RCTreasury.sol#L459-L461

## Tools Used

Manual code review

## Recommended Mitigation Steps

Modify the aforementioned functions such that all state changes are made before a call to the ERC20 token using the `safeTransferFrom()` function.

