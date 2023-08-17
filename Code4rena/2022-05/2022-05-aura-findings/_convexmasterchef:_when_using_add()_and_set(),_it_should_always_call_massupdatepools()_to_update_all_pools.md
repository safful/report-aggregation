## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [ ConvexMasterChef: When using add() and set(), it should always call massUpdatePools() to update all pools](https://github.com/code-423n4/2022-05-aura-findings/issues/147) 

# Lines of code

https://github.com/code-423n4/2022-05-aura/blob/4989a2077546a5394e3650bf3c224669a0f7e690/convex-platform/contracts/contracts/ConvexMasterChef.sol#L96-L138


# Vulnerability details

## Impact
Same as IDX-003 in https://public-stg.inspex.co/report/Inspex_AUDIT2021024_LuckyLion_Farm_FullReport_v2.0.pdf
The totalAllocPoint variable is used to determine the portion that each pool would get from the total reward, so it is one of the main factors used in the rewards calculation. Therefore, whenever the totalAllocPoint variable is modified without updating the pending reward first, the reward of each pool will be incorrectly calculated.
For example, when  _withUpdate is false, in the add() shown below, the totalAllocPoint variable will be modified without updating the rewards (massUpdatePools()).
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
## Proof of Concept
https://github.com/code-423n4/2022-05-aura/blob/4989a2077546a5394e3650bf3c224669a0f7e690/convex-platform/contracts/contracts/ConvexMasterChef.sol#L96-L138
## Tools Used
None
## Recommended Mitigation Steps
Removing the _withUpdate variable in the add() and set() functions and always calling the massUpdatePools() function before updating totalAllocPoint variable

