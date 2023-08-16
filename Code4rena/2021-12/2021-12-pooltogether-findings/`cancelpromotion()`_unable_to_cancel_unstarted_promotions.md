## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`cancelPromotion()` Unable to cancel unstarted promotions](https://github.com/code-423n4/2021-12-pooltogether-findings/issues/101) 

# Handle

WatchPug


# Vulnerability details

For unstarted promotions, `cancelPromotion()` will revert at `block.timestamp - _promotion.startTimestamp` in `_getCurrentEpochId()`.

Call stack: `cancelPromotion()` -> `_getRemainingRewards()` -> `_getCurrentEpochId()`.

https://github.com/pooltogether/v4-periphery/blob/0e94c54774a6fce29daf9cb23353208f80de63eb/contracts/TwabRewards.sol#L331-L336

```solidity=331
function _getRemainingRewards(Promotion memory _promotion) internal view returns (uint256) {
    // _tokensPerEpoch * _numberOfEpochsLeft
    return
        _promotion.tokensPerEpoch *
        (_promotion.numberOfEpochs - _getCurrentEpochId(_promotion));
}
```

https://github.com/pooltogether/v4-periphery/blob/0e94c54774a6fce29daf9cb23353208f80de63eb/contracts/TwabRewards.sol#L276-L279

```solidity=276
function _getCurrentEpochId(Promotion memory _promotion) internal view returns (uint256) {
    // elapsedTimestamp / epochDurationTimestamp
    return (block.timestamp - _promotion.startTimestamp) / _promotion.epochDuration;
}
```

### Recommendation

Consider checking if ` _promotion.startTimestamp > block.timestamp` and refund `_promotion.tokensPerEpoch * _promotion.numberOfEpochs` in `cancelPromotion()`.

