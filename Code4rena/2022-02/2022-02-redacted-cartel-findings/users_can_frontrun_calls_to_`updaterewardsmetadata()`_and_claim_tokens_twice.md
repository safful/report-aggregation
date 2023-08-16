## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Users Can Frontrun Calls to `updateRewardsMetadata()` And Claim Tokens Twice](https://github.com/code-423n4/2022-02-redacted-cartel-findings/issues/118) 

# Lines of code

https://github.com/code-423n4/2022-02-redacted-cartel/blob/main/contracts/RewardDistributor.sol#L97-L119
https://github.com/code-423n4/2022-02-redacted-cartel/blob/main/contracts/RewardDistributor.sol#L127-L209


# Vulnerability details

## Impact

The `updateRewardsMetadata()` function is called by the `BribeVault` contract by the admin role. The function will take a list of distributions which are used to update the associated reward metadata. It is expected that the merkle root will be updated to correctly identify which claimers have already claimed tokens. 

`reward.updateCount` is incremented to reset the claimed tracker, allowing users that may have previously claimed, to claim their updated reward. However, there is potential for mis-use if users frontrun calls to `updateRewardsMetadata()` and claim their reward after the new merkle root has been calculated and updated by the admin role. This may allow the claimer to double claim their rewards or lead to a loss in rewards if the reward metadata completely replaces the previous list of claimers.

## Proof of Concept

https://github.com/code-423n4/2022-02-redacted-cartel/blob/main/contracts/RewardDistributor.sol#L97-L119
https://github.com/code-423n4/2022-02-redacted-cartel/blob/main/contracts/RewardDistributor.sol#L127-L209

## Tools Used

Manual code review.

## Recommended Mitigation Steps

Consider implementing a delay where users cannot claim rewards before a call to `updateRewardsMetadata()` is made. This should ensure the admin role can construct a merkle tree based on the most up-to-date and correct data.

