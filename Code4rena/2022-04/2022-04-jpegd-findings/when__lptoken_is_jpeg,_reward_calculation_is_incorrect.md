## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [When _lpToken is jpeg, reward calculation is incorrect](https://github.com/code-423n4/2022-04-jpegd-findings/issues/1) 

# Lines of code

https://github.com/code-423n4/2022-04-jpegd/blob/main/contracts/farming/LPFarming.sol#L141-L154


# Vulnerability details

## Impact
In the LPFarming contract, a new staking pool can be added using the add() function. The staking token for the new pool is defined using the _lpToken variable. However, there is no additional checking whether the _lpToken is the same as the reward token (jpeg) or not.
```
    function add(uint256 _allocPoint, IERC20 _lpToken) external onlyOwner {
        _massUpdatePools();

        uint256 lastRewardBlock = _blockNumber();
        totalAllocPoint = totalAllocPoint + _allocPoint;
        poolInfo.push(
            PoolInfo({
                lpToken: _lpToken,
                allocPoint: _allocPoint,
                lastRewardBlock: lastRewardBlock,
                accRewardPerShare: 0
            })
        );
    }
```

When the _lpToken is the same token as jpeg, reward calculation for that pool in the updatePool() function can be incorrect. This is because the current balance of the _lpToken in the contract is used in the calculation of the reward. Since the _lpToken is the same token as the reward, the reward minted to the contract will inflate the value of lpSupply, causing the reward of that pool to be less than what it should be.
```
    function _updatePool(uint256 _pid) internal {
        PoolInfo storage pool = poolInfo[_pid];
        if (pool.allocPoint == 0) {
            return;
        }

        uint256 blockNumber = _blockNumber();
        //normalizing the pool's `lastRewardBlock` ensures that no rewards are distributed by staking outside of an epoch
        uint256 lastRewardBlock = _normalizeBlockNumber(pool.lastRewardBlock);
        if (blockNumber <= lastRewardBlock) {
            return;
        }
        uint256 lpSupply = pool.lpToken.balanceOf(address(this));
        if (lpSupply == 0) {
            pool.lastRewardBlock = blockNumber;
            return;
        }
        uint256 reward = ((blockNumber - lastRewardBlock) *
            epoch.rewardPerBlock *
            1e36 *
            pool.allocPoint) / totalAllocPoint;
        pool.accRewardPerShare = pool.accRewardPerShare + reward / lpSupply;
        pool.lastRewardBlock = blockNumber;
    }
```

## Proof of Concept
https://github.com/code-423n4/2022-04-jpegd/blob/main/contracts/farming/LPFarming.sol#L141-L154
https://github.com/code-423n4/2022-04-jpegd/blob/main/contracts/farming/LPFarming.sol#L288-L311
## Tools Used
None

## Recommended Mitigation Steps

Add a check that _lpToken is not jpeg in the add function or mint the reward token to another contract to prevent the amount of the staked token from being mixed up with the reward token.


