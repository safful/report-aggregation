## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [[WP-H13] `MasterChef.sol` Users won't be able to receive the `concur` rewards](https://github.com/code-423n4/2022-02-concur-findings/issues/200) 

# Lines of code

https://github.com/code-423n4/2022-02-concur/blob/72b5216bfeaa7c52983060ebfc56e72e0aa8e3b0/contracts/MasterChef.sol#L135-L154


# Vulnerability details

According to:

-   README https://github.com/code-423n4/2022-02-concur#-masterchef
-   Implementation of `deposit()`: [/contracts/MasterChef.sol#L157-L180](https://github.com/code-423n4/2022-02-concur/blob/72b5216bfeaa7c52983060ebfc56e72e0aa8e3b0/contracts/MasterChef.sol#L157-L180)

MasterChef is only recording the deposited amount in the states, it's not actually holding the `depositToken`.

`depositToken` won't be transferred from `_msgSender()` to the MasterChef contract.

Therefore, in `updatePool()` L140 `lpSupply = pool.depositToken.balanceOf(address(this))` will always be `0`. And the `updatePool()` will be returned at L147.

https://github.com/code-423n4/2022-02-concur/blob/72b5216bfeaa7c52983060ebfc56e72e0aa8e3b0/contracts/MasterChef.sol#L135-L154

```solidity
function updatePool(uint _pid) public {
    PoolInfo storage pool = poolInfo[_pid];
    if (block.number <= pool.lastRewardBlock) {
        return;
    }
    uint lpSupply = pool.depositToken.balanceOf(address(this));
    if (lpSupply == 0 || pool.allocPoint == 0) {
        pool.lastRewardBlock = block.number;
        return;
    }
    if(block.number >= endBlock) {
        pool.lastRewardBlock = block.number;
        return;
    }        

    uint multiplier = getMultiplier(pool.lastRewardBlock, block.number);
    uint concurReward = multiplier.mul(concurPerBlock).mul(pool.allocPoint).div(totalAllocPoint);
    pool.accConcurPerShare = pool.accConcurPerShare.add(concurReward.mul(_concurShareMultiplier).div(lpSupply));
    pool.lastRewardBlock = block.number;
}
```


### Impact

- The MasterChef contract fail to implement the most essential function;
- Users won't be able to receive any `Concur` rewards from MasterChef;

### Recommendation

Consider creating a receipt token to represent the invested token and use the receipt tokens in MasterChef.

See: https://github.com/convex-eth/platform/blob/883ffd4ebcaee12e64d18f75bdfe404bcd900616/contracts/contracts/Booster.sol#L272-L277

