## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas Optimizations](https://github.com/code-423n4/2022-02-concur-findings/issues/265) 

1) Use != instead of > for uint256 in StakingReward.sol: 

Both amount and reward are of type uin256 comparing, checking for inequality instead of a greater than relation saves gas. The comparisons can be found in the lines below

https://github.com/code-423n4/2022-02-concur/blob/72b5216bfeaa7c52983060ebfc56e72e0aa8e3b0/contracts/StakingRewards.sol#L94

https://github.com/code-423n4/2022-02-concur/blob/72b5216bfeaa7c52983060ebfc56e72e0aa8e3b0/contracts/StakingRewards.sol#L119

https://github.com/code-423n4/2022-02-concur/blob/72b5216bfeaa7c52983060ebfc56e72e0aa8e3b0/contracts/StakingRewards.sol#L119

Cache length before for loop ConcurRewardPool.sol:

2) The length can be cached before the loop avoid calling length many times to save gas.

https://github.com/code-423n4/2022-02-concur/blob/72b5216bfeaa7c52983060ebfc56e72e0aa8e3b0/contracts/ConcurRewardPool.sol#L35

3) Assignment to default value 

In L51 uint totalAllocPoint is being declared with the default value of 0. Just totalAllocPoint; will save gas. The same thing can also be observed in StakingRewards.sol L21 and L22. Also in ConvexStakingWrapper.sol L36

Similarly bool transferSuccess is also being assigned False in L204

https://github.com/code-423n4/2022-02-concur/blob/72b5216bfeaa7c52983060ebfc56e72e0aa8e3b0/contracts/MasterChef.sol#L51

https://github.com/code-423n4/2022-02-concur/blob/72b5216bfeaa7c52983060ebfc56e72e0aa8e3b0/contracts/MasterChef.sol#L204

https://github.com/code-423n4/2022-02-concur/blob/72b5216bfeaa7c52983060ebfc56e72e0aa8e3b0/contracts/StakingRewards.sol#L21

https://github.com/code-423n4/2022-02-concur/blob/72b5216bfeaa7c52983060ebfc56e72e0aa8e3b0/contracts/StakingRewards.sol#L22

https://github.com/code-423n4/2022-02-concur/blob/72b5216bfeaa7c52983060ebfc56e72e0aa8e3b0/contracts/ConvexStakingWrapper.sol#L36

4) Resturcture if-else statement to remove else in StakingRewards.sol L142:

if (block.timestamp >= periodFinish) {
            rewardRate = reward / rewardsDuration;
        } else {
            uint256 remaining = periodFinish - block.timestamp;
            uint256 leftover = remaining * rewardRate;
            rewardRate = (reward + leftover) / rewardsDuration;
        }

can be reformulated as:

rewardRate = reward / rewardsDuration;
if (block.timestamp < periodFinish) {
            uint256 remaining = periodFinish - block.timestamp;
            uint256 leftover = remaining * rewardRate;
            rewardRate += (leftover / rewardsDuration);
        }


https://github.com/code-423n4/2022-02-concur/blob/72b5216bfeaa7c52983060ebfc56e72e0aa8e3b0/contracts/StakingRewards.sol#L142


5) use unchecked{...} to save gas

uint256 is way too big to realistically lead to overflows, unchecked can be used to save gas in the following situations.


https://github.com/code-423n4/2022-02-concur/blob/72b5216bfeaa7c52983060ebfc56e72e0aa8e3b0/contracts/ConvexStakingWrapper.sol#L121

https://github.com/code-423n4/2022-02-concur/blob/72b5216bfeaa7c52983060ebfc56e72e0aa8e3b0/contracts/ConvexStakingWrapper.sol#L219

https://github.com/code-423n4/2022-02-concur/blob/72b5216bfeaa7c52983060ebfc56e72e0aa8e3b0/contracts/ConcurRewardPool.sol#L34







