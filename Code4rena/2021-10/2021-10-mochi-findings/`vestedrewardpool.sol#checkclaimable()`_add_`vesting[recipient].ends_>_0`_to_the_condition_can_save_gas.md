## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [`VestedRewardPool.sol#checkClaimable()` Add `vesting[recipient].ends > 0` to the condition can save gas](https://github.com/code-423n4/2021-10-mochi-findings/issues/102) 

# Handle

WatchPug


# Vulnerability details

When the vesting ends, `vesting[recipient].ends` will be `0` which always passes the check of `vesting[recipient].ends < block.timestamp` and causes unnecessary code execution.

Adding a check of `vesting[recipient].ends > 0` can avoid unnecessary code execution and save gas.

https://github.com/code-423n4/2021-10-mochi/blob/8458209a52565875d8b2cefcb611c477cefb9253/projects/mochi-core/contracts/emission/VestedRewardPool.sol#L22-L29

```solidity
modifier checkClaimable(address recipient) {
    if (vesting[recipient].ends < block.timestamp) {
        vesting[recipient].claimable += vesting[recipient].vested;
        vesting[recipient].vested = 0;
        vesting[recipient].ends = 0;
    }
    _;
}
```

### Recommendation

Change to:

```solidity
modifier checkClaimable(address recipient) {
    if (vesting[recipient].ends > 0 && vesting[recipient].ends < block.timestamp) {
        vesting[recipient].claimable += vesting[recipient].vested;
        vesting[recipient].vested = 0;
        vesting[recipient].ends = 0;
    }
    _;
}
```

