## Tags

- bug
- 2 (Med Risk)
- satisfactory
- sponsor confirmed
- selected for report
- M-11

# [`viewPrice` doesn't always report dampened price](https://github.com/code-423n4/2022-10-inverse-findings/issues/404) 

# Lines of code

https://github.com/code-423n4/2022-10-inverse/blob/3e81f0f5908ea99b36e6ab72f13488bbfe622183/src/Oracle.sol#L91


# Vulnerability details

## Impact
Oracle's `viewPrice` function doesn't report a dampened price until `getPrice` is called and today's price is updated. This will impact the public read-only functions that call it:
- [getCollateralValue](https://github.com/code-423n4/2022-10-inverse/blob/3e81f0f5908ea99b36e6ab72f13488bbfe622183/src/Market.sol#L312);
- [getCreditLimit](https://github.com/code-423n4/2022-10-inverse/blob/3e81f0f5908ea99b36e6ab72f13488bbfe622183/src/Market.sol#L334) (calls `getCollateralValue`);
- [getLiquidatableDebt](https://github.com/code-423n4/2022-10-inverse/blob/3e81f0f5908ea99b36e6ab72f13488bbfe622183/src/Market.sol#L578) (calls `getCreditLimit`);
- [getWithdrawalLimit](https://github.com/code-423n4/2022-10-inverse/blob/3e81f0f5908ea99b36e6ab72f13488bbfe622183/src/Market.sol#L370).

These functions are used to get on-chain state and prepare values for write calls (e.g. calculate withdrawal amount before withdrawing or calculate a user's debt that can be liquidated before liquidating it). Thus, wrong values returned by these functions can cause withdrawal of a wrong amount or liquidation of a wrong debt or cause reverts.
## Proof of Concept
```solidity
// src/test/Oracle.t.sol
function test_viewPriceNoDampenedPrice_AUDIT() public {
    uint collateralFactor = market.collateralFactorBps();
    uint day = block.timestamp / 1 days;
    uint feedPrice = ethFeed.latestAnswer();

    //1600e18 price saved as daily low
    oracle.getPrice(address(WETH), collateralFactor);
    assertEq(oracle.dailyLows(address(WETH), day), feedPrice);

    vm.warp(block.timestamp + 1 days);
    uint newPrice = 1200e18;
    ethFeed.changeAnswer(newPrice);
    //1200e18 price saved as daily low
    oracle.getPrice(address(WETH), collateralFactor);
    assertEq(oracle.dailyLows(address(WETH), ++day), newPrice);

    vm.warp(block.timestamp + 1 days);
    newPrice = 3000e18;
    ethFeed.changeAnswer(newPrice);

    //1200e18 should be twoDayLow, 3000e18 is current price. We should receive dampened price here.
    // Notice that viewPrice is called before getPrice.
    uint viewPrice = oracle.viewPrice(address(WETH), collateralFactor);
    uint price = oracle.getPrice(address(WETH), collateralFactor);
    assertEq(oracle.dailyLows(address(WETH), ++day), newPrice);

    assertEq(price, 1200e18 * 10_000 / collateralFactor);

    // View price wasn't dampened.
    assertEq(viewPrice, 3000e18);
}
```
## Tools Used
Manual review
## Recommended Mitigation Steps
Consider this change:
```diff
--- a/src/Oracle.sol
+++ b/src/Oracle.sol
@@ -89,6 +89,9 @@ contract Oracle {
             uint day = block.timestamp / 1 days;
             // get today's low
             uint todaysLow = dailyLows[token][day];
+            if(todaysLow == 0 || normalizedPrice < todaysLow) {
+                todaysLow = normalizedPrice;
+            }
             // get yesterday's low
             uint yesterdaysLow = dailyLows[token][day - 1];
             // calculate new borrowing power based on collateral factor
```