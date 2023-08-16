## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Tokens Can Be Stolen By Frontrunning `VestedRewardPool.vest()` and `VestedRewardPool.lock()`](https://github.com/code-423n4/2021-10-mochi-findings/issues/92) 

# Handle

leastwood


# Vulnerability details

## Impact

The `VestedRewardPool.sol` contract is a public facing contract aimed at vesting tokens for a minimum of 90 days before allowing the recipient to withdraw their `mochi`. The `vest()` function does not utilise `safeTransferFrom()` to ensure that vested tokens are correctly allocated to the recipient. As a result, it is possible to frontrun a call to `vest()` and effectively steal a recipient's vested tokens. The same issue applies to the `lock()` function.

## Proof of Concept

https://github.com/code-423n4/2021-10-mochi/blob/main/projects/mochi-core/contracts/emission/VestedRewardPool.sol#L36-L46
https://github.com/code-423n4/2021-10-mochi/blob/main/projects/mochi-core/contracts/emission/VestedRewardPool.sol#L54-L64

## Tools Used

Manual code review
Discussions with the Mochi team

## Recommended Mitigation Steps

Ensure that users understand that this function should not be interacted directly as this could result in lost `mochi` tokens. Additionally, it might be worthwhile creating a single externally facing function which calls `safeTransferFrom()`, `vest()` and `lock()` in a single transaction. 

