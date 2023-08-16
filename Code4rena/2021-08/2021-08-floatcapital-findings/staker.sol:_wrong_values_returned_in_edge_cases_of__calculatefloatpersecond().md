## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- resolved
- fixed-in-upstream-repo

# [Staker.sol: Wrong values returned in edge cases of _calculateFloatPerSecond()](https://github.com/code-423n4/2021-08-floatcapital-findings/issues/6) 

# Handle 

hickuphh3

# Vulnerability details

### Impact

In `_calculateFloatPerSecond()`, the edge cases where full rewards go to either the long or short token returns

`return (1e18 * k * longPrice, 0);` and

`return (0, 1e18 * k * shortPrice);` respectively. 

This is however `1e18` times too large. We can verify this by checking the equivalent calculation in the 'normal case', where we assume all the rewards go to the short token, ie. `longRewardUnscaled = 0`  and `shortRewardUnscaled = 1e18`. Plugging this into the calculation below,

`return ((longRewardUnscaled * k * longPrice) / 1e18, (shortRewardUnscaled * k * shortPrice) / 1e18);` results in

`(0, 1e18 * k * shortPrice / 1e18)` or `(0, k * shortPrice)`.

As we can see, this would result in an extremely large float token issuance rate, which would be disastrous.

### Recommended Mitigation Steps

The edge cases should return `(k * longPrice, 0)` and `(0, k * shortPrice)` in the cases where rewards should go fully to long and short token holders respectively.