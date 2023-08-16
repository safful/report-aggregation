## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [Wrong `lastBuyBackPrice`](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/206) 

# Handle

cmichel


# Vulnerability details

The `sYETIToken.lastBuyBackPrice` is set in `buyBack` and hardcoded as:

```solidity
function buyBack(address routerAddress, uint256 YUSDToSell, uint256 YETIOutMin, address[] memory path) external onlyOwner {
    require(YUSDToSell > 0, "Zero amount");
    require(lastBuybackTime + 69 hours < block.timestamp, "Must have 69 hours pass before another buyBack");
    yusdToken.approve(routerAddress, YUSDToSell);
    uint256[] memory amounts = IRouter(routerAddress).swapExactTokensForTokens(YUSDToSell, YETIOutMin, path, address(this), block.timestamp + 5 minutes);
    lastBuybackTime = block.timestamp;
    // amounts[0] is the amount of YUSD that was sold, and amounts[1] is the amount of YETI that was gained in return. So the price is amounts[0] / amounts[1]
    // @audit this hardcoded lastBuybackPrice is wrong when using a different path (think path length 3)
    lastBuybackPrice = div(amounts[0].mul(1e18), amounts[1]);
    emit BuyBackExecuted(YUSDToSell, amounts[0], amounts[1]);
}
```

It divides the first and second return `amounts` of the swap, however, these amounts depend on the swap `path` parameter that is used by the caller.
If a swap path of length 3 is used, then this is obviously wrong.
It also assumes that each router sorts the pairs the same way (which is true for Uniswap/Sushiswap).

## Impact
The `lastBuyBackPrice` will be wrong when using a different path.
This will lead `rebase`s using a different yeti amount and the `effectiveYetiTokenBalance` being updated wrong.

## Recommended Mitigation Steps
Verify the first and last element of the path are YETI/YUSD and use the first and last amount parameter.


