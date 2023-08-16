## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Avoid unnecessary external calls can save gas](https://github.com/code-423n4/2021-11-fairside-findings/issues/55) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-11-fairside/blob/20c68793f48ee2678508b9d3a1bae917c007b712/contracts/network/FSDNetwork.sol#L310-L315

```solidity=310
// 20% as staking rewards
fsd.safeTransfer(address(fsd), stakingRewards);
fsd.addRegistrationTribute(stakingRewards);

// 7.5% towards governance
fsd.safeTransfer(address(fsd), governancePoolRewards);
```

The 2 transfers to the same address can be done in one external call to save gas

### Recommendation

Change to:

```solidity=310
// 20% as staking rewards
// 7.5% towards governance
fsd.safeTransfer(address(fsd), governancePoolRewards + stakingRewards);
fsd.addRegistrationTribute(stakingRewards);
```

