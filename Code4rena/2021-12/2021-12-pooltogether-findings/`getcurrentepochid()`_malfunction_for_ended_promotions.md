## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [`getCurrentEpochId()` Malfunction for ended promotions](https://github.com/code-423n4/2021-12-pooltogether-findings/issues/109) 

# Handle

WatchPug


# Vulnerability details

For ended promotions, `getCurrentEpochId()` may return a `epochId` larger than `numberOfEpochs`.

If the result of this view method is to be used as parameters of `claimRewards()`, it may cause `claimRewards()` to fail.

https://github.com/pooltogether/v4-periphery/blob/0e94c54774a6fce29daf9cb23353208f80de63eb/contracts/TwabRewards.sol#L276-L279

```solidity=276
function _getCurrentEpochId(Promotion memory _promotion) internal view returns (uint256) {
    // elapsedTimestamp / epochDurationTimestamp
    return (block.timestamp - _promotion.startTimestamp) / _promotion.epochDuration;
}
```

### Recommendation

Consider checking if `block.timestamp > _promotionEndTimestamp` in `_getCurrentEpochId()` and return `_promotion.numberOfEpochs - 1` for ended promotions.

