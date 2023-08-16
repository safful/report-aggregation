## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [`StabilizerNode.sol` The current implementation is misconfiguration-prone for rewardToken with non-18 decimals](https://github.com/code-423n4/2021-11-malt-findings/issues/293) 

# Handle

WatchPug


# Vulnerability details

The default `upperStabilityThreshold` and `lowerStabilityThreshold` assumes that `rewardToken.decimals()` is 18.

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/StabilizerNode.sol#L32-L33

```solidity=32
  uint256 public upperStabilityThreshold = (10**18) / 100; // 1%
  uint256 public lowerStabilityThreshold = (10**18) / 100;
```

When the `StabilizerNode.sol` contract is initialized with a rewardToken with decimals of 8 (eg. USDC). `upperThreshold` and `lowerThreshold` will be much larger than expected.

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/StabilizerNode.sol#L198-L206
```solidity
  function _shouldAdjustSupply(uint256 exchangeRate) internal view returns (bool) {
    uint256 decimals = rewardToken.decimals();
    uint256 priceTarget = maltDataLab.priceTarget();

    uint256 upperThreshold = priceTarget.mul(upperStabilityThreshold).div(10**decimals);
    uint256 lowerThreshold = priceTarget.mul(lowerStabilityThreshold).div(10**decimals);
```

### Recommendation

Consider changing to:

```solidity
  uint256 public upperStabilityThresholdBps = 100; // 1%
  uint256 public lowerStabilityThresholdBps = 100;
```

```solidity
  function _shouldAdjustSupply(uint256 exchangeRate) internal view returns (bool) {
    uint256 decimals = rewardToken.decimals();
    uint256 priceTarget = maltDataLab.priceTarget();

    uint256 upperThreshold = priceTarget.mul(upperStabilityThreshold).div(10000);
    uint256 lowerThreshold = priceTarget.mul(lowerStabilityThreshold).div(10000);
```

