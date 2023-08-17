## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- sponsor confirmed
- selected for report
- M-06

# [Reward can be over- or undercounted in `extendPledge` and `increasePledgeRewardPerVote`](https://github.com/code-423n4/2022-10-paladin-findings/issues/163) 

# Lines of code

https://github.com/code-423n4/2022-10-paladin/blob/d6d0c0e57ad80f15e9691086c9c7270d4ccfe0e6/contracts/WardenPledge.sol#L387
https://github.com/code-423n4/2022-10-paladin/blob/d6d0c0e57ad80f15e9691086c9c7270d4ccfe0e6/contracts/WardenPledge.sol#L432


# Vulnerability details

## Impact
Total reward amount in `extendPledge` and `increasePledgeRewardPerVote` can be calculated incorrectly due to cached `pledgeParams.votesDifference`, which can lead to two outcomes:
1. total reward amount is higher, thus a portion of it won't be claimable;
1. total reward amount is lower, thus the pledge target won't be reached.

## Proof of Concept
When a pledge is created, the creator chooses the target–the total amount of votes they want to reach with the pledge. Based on a target, the number of missing votes is calculated, which is then used to calculated the total reward amount ([WardenPledge.sol#L325-L327](https://github.com/code-423n4/2022-10-paladin/blob/d6d0c0e57ad80f15e9691086c9c7270d4ccfe0e6/contracts/WardenPledge.sol#L325-L327)):
```solidity
function createPledge(
    address receiver,
    address rewardToken,
    uint256 targetVotes,
    uint256 rewardPerVote, // reward/veToken/second
    uint256 endTimestamp,
    uint256 maxTotalRewardAmount,
    uint256 maxFeeAmount
) external whenNotPaused nonReentrant returns(uint256){
    ...
    // Get the missing votes for the given receiver to reach the target votes
    // We ignore any delegated boost here because they might expire during the Pledge duration
    // (we can have a future version of this contract using adjusted_balance)
    vars.votesDifference = targetVotes - votingEscrow.balanceOf(receiver);

    vars.totalRewardAmount = (rewardPerVote * vars.votesDifference * vars.duration) / UNIT;
    ...
  }
```

When extending a pledge or increasing a pledge reward per vote, current veToken balance of the pledge's receiver (`votingEscrow.balanceOf(receiver)`) can be different from the one it had when the pledge was created (e.g. the receiver managed to lock more CRV or some of locked tokens have expired). However `pledgeParams.votesDifference` is not recalculated ([WardenPledge.sol#L387](https://github.com/code-423n4/2022-10-paladin/blob/d6d0c0e57ad80f15e9691086c9c7270d4ccfe0e6/contracts/WardenPledge.sol#L387), [WardenPledge.sol#L432](https://github.com/code-423n4/2022-10-paladin/blob/d6d0c0e57ad80f15e9691086c9c7270d4ccfe0e6/contracts/WardenPledge.sol#L432)):
```solidity
function extendPledge(
    uint256 pledgeId,
    uint256 newEndTimestamp,
    uint256 maxTotalRewardAmount,
    uint256 maxFeeAmount
) external whenNotPaused nonReentrant {
    ...
    Pledge storage pledgeParams = pledges[pledgeId];
    ...
    uint256 totalRewardAmount = (pledgeParams.rewardPerVote * pledgeParams.votesDifference * addedDuration) / UNIT;
    ...
}

function increasePledgeRewardPerVote(
    uint256 pledgeId,
    uint256 newRewardPerVote,
    uint256 maxTotalRewardAmount,
    uint256 maxFeeAmount
) external whenNotPaused nonReentrant {
    ...
    Pledge storage pledgeParams = pledges[pledgeId];
    ...
    uint256 totalRewardAmount = (rewardPerVoteDiff * pledgeParams.votesDifference * remainingDuration) / UNIT;
    ...
}
```

This can lead to two consequences:
1. When receiver's veToken balance has increased (i.e. `votesDifference` got in fact smaller), pledge creator will overpay for pledge extension and pledge reward per vote increase. This extra reward cannot be received by pledgers because a receiver cannot get more votes than `pledgeParams.targetVotes` (which is not updated when modifying a pledge):
    ```solidity
    function _pledge(uint256 pledgeId, address user, uint256 amount, uint256 endTimestamp) internal {
        ...
        // Check that this will not go over the Pledge target of votes
        if(delegationBoost.adjusted_balance_of(pledgeParams.receiver) + amount > pledgeParams.targetVotes) revert Errors.TargetVotesOverflow();
        ...
    }
    ```
1. When receiver's veToken balance has decreased (i.e. `votesDifference` got in fact bigger), the pledge target cannot be reached because the reward amount was underpaid in `extendPledge`/`increasePledgeRewardPerVote`.

## Tools Used
Manual review
## Recommended Mitigation Steps
Consider updating `votesDifference` when extending a pledge or increasing a pledge reward per vote.