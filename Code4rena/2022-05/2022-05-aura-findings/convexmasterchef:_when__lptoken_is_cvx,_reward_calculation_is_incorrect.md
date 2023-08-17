## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [ConvexMasterChef: When _lpToken is cvx, reward calculation is incorrect](https://github.com/code-423n4/2022-05-aura-findings/issues/151) 

# Lines of code

https://github.com/code-423n4/2022-05-aura/blob/4989a2077546a5394e3650bf3c224669a0f7e690/convex-platform/contracts/contracts/ConvexMasterChef.sol#L96-L118


# Vulnerability details

## Impact
In the ConvexMasterChef contract, a new staking pool can be added using the add() function. The staking token for the new pool is defined using the _lpToken variable. However, there is no additional checking whether the _lpToken is the same as the reward token (cvx) or not.
```
  function add(
      uint256 _allocPoint,
      IERC20 _lpToken,
      IRewarder _rewarder,
      bool _withUpdate
  ) public onlyOwner {
      if (_withUpdate) {
          massUpdatePools();
      }
      uint256 lastRewardBlock = block.number > startBlock
          ? block.number
          : startBlock;
      totalAllocPoint = totalAllocPoint.add(_allocPoint);
      poolInfo.push(
          PoolInfo({
              lpToken: _lpToken,
              allocPoint: _allocPoint,
              lastRewardBlock: lastRewardBlock,
              accCvxPerShare: 0,
              rewarder: _rewarder
          })
      );
  }
```
When the _lpToken is the same token as cvx, reward calculation for that pool in the updatePool() function can be incorrect. This is because the current balance of the _lpToken in the contract is used in the calculation of the reward. Since the _lpToken is the same token as the reward, the reward minted to the contract will inflate the value of lpSupply, causing the reward of that pool to be less than what it should be.
```
  function updatePool(uint256 _pid) public {
      PoolInfo storage pool = poolInfo[_pid];
      if (block.number <= pool.lastRewardBlock) {
          return;
      }
      uint256 lpSupply = pool.lpToken.balanceOf(address(this));
      if (lpSupply == 0) {
          pool.lastRewardBlock = block.number;
          return;
      }
      uint256 multiplier = getMultiplier(pool.lastRewardBlock, block.number);
      uint256 cvxReward = multiplier
          .mul(rewardPerBlock)
          .mul(pool.allocPoint)
          .div(totalAllocPoint);
      //cvx.mint(address(this), cvxReward);
      pool.accCvxPerShare = pool.accCvxPerShare.add(
          cvxReward.mul(1e12).div(lpSupply)
      );
      pool.lastRewardBlock = block.number;
  }
```
## Proof of Concept
https://github.com/code-423n4/2022-05-aura/blob/4989a2077546a5394e3650bf3c224669a0f7e690/convex-platform/contracts/contracts/ConvexMasterChef.sol#L96-L118
https://github.com/code-423n4/2022-05-aura/blob/4989a2077546a5394e3650bf3c224669a0f7e690/convex-platform/contracts/contracts/ConvexMasterChef.sol#L186-L206
## Tools Used
None
## Recommended Mitigation Steps
Add a check that _lpToken is not cvx in the add function or mint the reward token to another contract to prevent the amount of the staked token from being mixed up with the reward token.

