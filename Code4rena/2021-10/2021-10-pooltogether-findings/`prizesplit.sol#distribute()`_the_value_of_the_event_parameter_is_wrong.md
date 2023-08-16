## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- resolved

# [`PrizeSplit.sol#distribute()` The value of the event parameter is wrong](https://github.com/code-423n4/2021-10-pooltogether-findings/issues/35) 

# Handle

WatchPug


# Vulnerability details

https://github.com/pooltogether/v4-core/blob/055335bf9b09e3f4bbe11a788710dd04d827bf37/contracts/prize-strategy/PrizeSplitStrategy.sol#L51-L61

```solidity
function distribute() external override returns (uint256) {
    uint256 prize = prizePool.captureAwardBalance();

    if (prize == 0) return 0;

    _distributePrizeSplits(prize);

    emit Distributed(prize);

    return prize;
}
```

Based on the context, the value of the parameter of the `Distributed` event should be the distributed prize amount, which can be calculated based on the return value of `_distributePrizeSplits`.

### Recommendation

Change to:

```solidity

event Distributed(uint256 totalPrizeCaptured, uint256 totalPrizeDistributed);

function distribute() external override returns (uint256) {
    uint256 prize = prizePool.captureAwardBalance();

    if (prize == 0) return 0;

    uint remainingPrize = _distributePrizeSplits(prize);

    emit Distributed(prize, prize - remainingPrize);

    return prize;
}
```

