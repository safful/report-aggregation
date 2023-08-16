## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [`SwapUtils.sol` Wrong implementation](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/252) 

# Handle

WatchPug


# Vulnerability details

Based on the context, the `tokenPrecisionMultipliers` used in price calculation should be calculated in realtime based on `initialTargetPrice`, `futureTargetPrice`, `futureTargetPriceTime` and current time, just like `getA()` and `getA2()`.

However, in the current implementation, `tokenPrecisionMultipliers` used in price calculation is the stored value, it will only be changed when the owner called `rampTargetPrice()` and `stopRampTargetPrice()`.

As a result, the `targetPrice` set by the owner will not be effective until another `targetPrice` is being set or `stopRampTargetPrice()` is called.

### Recommendation

Consider adding `Swap.targetPrice` and changing the `_xp()` at L661 from:

https://github.com/code-423n4/2021-11-bootfinance/blob/f102ee73eb320532c5a7c1e833f225c479577e39/customswap/contracts/SwapUtils.sol#L661-L667

```solidity=661
function _xp(Swap storage self, uint256[] memory balances)
    internal
    view
    returns (uint256[] memory)
{
    return _xp(balances, self.tokenPrecisionMultipliers);
}
```

To:

```solidity=661
function _xp(Swap storage self, uint256[] memory balances)
    internal
    view
    returns (uint256[] memory)
{
    uint256[2] memory tokenPrecisionMultipliers = self.tokenPrecisionMultipliers;
    tokenPrecisionMultipliers[0] = self.targetPrice.originalPrecisionMultipliers[0].mul(_getTargetPricePrecise(self)).div(WEI_UNIT)
    return _xp(balances, tokenPrecisionMultipliers);
}
```

