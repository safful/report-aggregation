## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`UniswapHandler.maltMarketPrice` returns wrong decimals](https://github.com/code-423n4/2021-11-malt-findings/issues/255) 

# Handle

cmichel


# Vulnerability details

The `UniswapHandler.maltMarketPrice` function returns a tuple of the `price` and the `decimals` of the price.
However, the returned `decimals` do not match the computed `price` for the `else if (rewardDecimals < maltDecimals)` branch:

```solidity
else if (rewardDecimals < maltDecimals) {
  uint256 diff = maltDecimals - rewardDecimals;
  price = (rewardReserves.mul(10**diff)).mul(10**rewardDecimals).div(maltReserves);
  decimals = maltDecimals;
}
```

Note that `rewardReserves` are in reward token decimals, `maltReserves` is a malt balance amount (18 decimals).
Then, the returned amount is in `rewardDecimals + diffDecimals + rewardDecimals - maltDecimals = maltDecimals + rewardDecimals - maltDecimals = rewardDecimals`.
However `decimals = maltDecimals` is wrongly returned.

## Impact
Callers to this function will receive a price in unexpected decimals and might inflate or deflate the actual amount.
Luckily, the `AuctionEscapeHatch` decides to completely ignore the returned `decimals` and as all prices are effectively in `rewardDecimals`, even if stated in `maltDecimals`, it currently does not seem to lead to an issue.

## Recommendation
Fix the function by returning `rewardDecimals` instead of `maltDecimals` in the `rewardDecimals < maltDecimals` branch.

