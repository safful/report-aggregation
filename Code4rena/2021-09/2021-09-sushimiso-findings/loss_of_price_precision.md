## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Loss of price precision](https://github.com/code-423n4/2021-09-sushimiso-findings/issues/108) 

# Handle

cmichel


# Vulnerability details

The `DutchAuction._currentPrice()` is computed by multiplying with `priceDrop()`.
However, `priceDrop()` already performs a division and the final current price, therefore, loses precision.

Note that `priceDrop()` could even return `0` for ultra-low prices or very long auctions.

Imagine the actual payment per auction token price is `10^-12` => `startPrice` and `endPrice` are set with 18 decimals as ~`10^6`, but for auctions over a year (31,536,000 seconds > `10^6`) it'll then return 0.

## Impact
Precision can be lost leading to less accurate token auction results or even completely breaking the auction if the price is very low and the auctions are very long.

## Recommended Mitigation Steps
Perform all multiplications before divisions:

```solidity
uint256 priceDiff = block.timestamp.sub(uint256(marketInfo.startTime)).mul(
    uint256(_marketPrice.startPrice.sub(_marketPrice.minimumPrice))
  ) / uint256(_marketInfo.endTime.sub(_marketInfo.startTime));
```


