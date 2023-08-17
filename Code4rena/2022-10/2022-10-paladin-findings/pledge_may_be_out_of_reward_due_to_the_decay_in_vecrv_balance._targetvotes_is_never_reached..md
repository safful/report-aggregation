## Tags

- bug
- help wanted
- 2 (Med Risk)
- judge review requested
- primary issue
- sponsor confirmed
- selected for report
- M-03

# [Pledge may be out of reward due to the decay in veCRV balance. targetVotes is never reached.](https://github.com/code-423n4/2022-10-paladin-findings/issues/91) 

# Lines of code

https://github.com/code-423n4/2022-10-paladin/blob/d6d0c0e57ad80f15e9691086c9c7270d4ccfe0e6/contracts/WardenPledge.sol#L325-L335
https://github.com/code-423n4/2022-10-paladin/blob/d6d0c0e57ad80f15e9691086c9c7270d4ccfe0e6/contracts/WardenPledge.sol#L259-L268


# Vulnerability details

## Impact
Pledge may be out of reward due to the decay in veCRV balance. The receiver may lose his reward given to boosters but get nothing in return since her targetVotes is never reached.

## Proof of Concept
According to Curve documentation at https://curve.readthedocs.io/dao-vecrv.html

```
A user’s veCRV balance decays linearly as the remaining time until the CRV unlock decreases. For example, a balance of 4000 CRV locked for one year provides the same amount of veCRV as 2000 CRV locked for two years, or 1000 CRV locked for four years.
```

On creation, targetVotes = 100, balance = 20 -> votesDifference = 80 -> reward is allocated for 80 votes

```solidity
        // Get the missing votes for the given receiver to reach the target votes
        // We ignore any delegated boost here because they might expire during the Pledge duration
        // (we can have a future version of this contract using adjusted_balance)
        vars.votesDifference = targetVotes - votingEscrow.balanceOf(receiver);

        vars.totalRewardAmount = (rewardPerVote * vars.votesDifference * vars.duration) / UNIT;
        vars.feeAmount = (vars.totalRewardAmount * protocalFeeRatio) / MAX_PCT ;
        if(vars.totalRewardAmount > maxTotalRewardAmount) revert Errors.IncorrectMaxTotalRewardAmount();
        if(vars.feeAmount > maxFeeAmount) revert Errors.IncorrectMaxFeeAmount();

        // Pull all the rewards in this contract
        IERC20(rewardToken).safeTransferFrom(creator, address(this), vars.totalRewardAmount);
        // And transfer the fees from the Pledge creator to the Chest contract
        IERC20(rewardToken).safeTransferFrom(creator, chestAddress, vars.feeAmount);
```

Then 1 week passed, receiver's balance decay to 10

On creation, targetVotes = 100, balance = 10 but votesDifference stays 80, and reward has only allocated for 80 votes.

```solidity
        // Rewards are set in the Pledge as reward/veToken/sec
        // To find the total amount of veToken delegated through the whole Boost duration
        // based on the Boost bias & the Boost duration, to take in account that the delegated amount decreases
        // each second of the Boost duration
        uint256 totalDelegatedAmount = ((bias * boostDuration) + bias) / 2;
        // Then we can calculate the total amount of rewards for this Boost
        uint256 rewardAmount = (totalDelegatedAmount * pledgeParams.rewardPerVote) / UNIT;

        if(rewardAmount > pledgeAvailableRewardAmounts[pledgeId]) revert Errors.RewardsBalanceTooLow();
        pledgeAvailableRewardAmounts[pledgeId] -= rewardAmount;
```

A booster boosts 80 votes and takes all rewards in the pool. However, only 80 (From booster) + 10 (From receiver) = 90 votes is active. Not 100 votes that receiver promise in the targetVotes.

Then, if another booster tries to boost 10 votes, it will be reverted with RewardsBalanceTooLow since the first booster has taken all reward that is allocated for only 80 votes.

## Recommended Mitigation Steps
You should provide a way for the creator to provide additional rewards after the pledge creation. Or provide some reward refreshment function that recalculates votesDifference and transfers the required additional reward.