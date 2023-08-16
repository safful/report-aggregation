## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Checking if `lpProfitCut > 0` can save gas](https://github.com/code-423n4/2021-11-malt-findings/issues/310) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/SwingTrader.sol#L136-L141

```solidity
if (profit > 0) {
    uint256 lpCut = profit.mul(lpProfitCut).div(1000);

    collateralToken.safeTransfer(address(rewardThrottle), lpCut);
    rewardThrottle.handleReward();
  }
```

Given that `lpProfitCut` can be `0`, checking if `lpProfitCut > 0` can avoid unnecessary code execution (including external calls) and save some gas.

