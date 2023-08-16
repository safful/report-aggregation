## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [deposit in ConvexStakingWrapper will most certainly revert](https://github.com/code-423n4/2022-02-concur-findings/issues/33) 

# Handle

wuwe1


# Vulnerability details

## Proof of Concept
https://github.com/code-423n4/2022-02-concur/blob/main/contracts/ConvexStakingWrapper.sol#L94-L99

```solidity
        address mainPool = IRewardStaking(convexBooster)
            .poolInfo(_pid)
            .crvRewards;
        if (rewards[_pid].length == 0) {
            pids[IRewardStaking(convexBooster).poolInfo(_pid).lptoken] = _pid;
            convexPool[_pid] = mainPool;
```

`convexPool[_pid]` is set to `IRewardStaking(convexBooster).poolInfo(_pid).crvRewards;`

`crvRewards` is a `BaseRewardPool` like this one https://etherscan.io/address/0x8B55351ea358e5Eda371575B031ee24F462d503e#code.

`BaseRewardPool` does not implement `poolInfo`


https://github.com/code-423n4/2022-02-concur/blob/main/contracts/ConvexStakingWrapper.sol#L238

```solidity
IRewardStaking(convexPool[_pid]).poolInfo(_pid).lptoken
```

Above line calls `poolInfo` of  `crvRewards` which causes revert.

## Recommended Mitigation Steps

According to Booster's code 

https://etherscan.io/address/0xF403C135812408BFbE8713b5A23a04b3D48AAE31#code

```solidity
    //deposit lp tokens and stake
    function deposit(uint256 _pid, uint256 _amount, bool _stake) public returns(bool){
        require(!isShutdown,"shutdown");
        PoolInfo storage pool = poolInfo[_pid];
        require(pool.shutdown == false, "pool is closed");

        //send to proxy to stake
        address lptoken = pool.lptoken;
        IERC20(lptoken).safeTransferFrom(msg.sender, staker, _amount);
```

`convexBooster` requires `poolInfo[_pid].lptoken`.

change L238 to 

```solidity
IRewardStaking(convexBooster).poolInfo(_pid).lptoken
```

