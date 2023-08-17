## Tags

- bug
- 2 (Med Risk)
- satisfactory
- selected for report
- sponsor confirmed
- M-13

# [Function `stabilize()` might always revert because of overflow since Malt contract use solidity 0.8](https://github.com/code-423n4/2023-02-malt-findings/issues/15) 

# Lines of code

https://github.com/code-423n4/2023-02-malt/blob/700f9b468f9cf8c9c5cffaa1eba1b8dea40503f9/contracts/StabilityPod/StabilizerNode.sol#L161
https://github.com/code-423n4/2023-02-malt/blob/700f9b468f9cf8c9c5cffaa1eba1b8dea40503f9/contracts/DataFeed/MaltDataLab.sol#L326


# Vulnerability details

## Impact
MaltDataLab fetched `priceCumulative` directly from Uniswap V2 pool to calculate price of Malt token. However, it is noticed that Uniswap V2 pool use Solidity 0.5.16, which does not revert when overflow happen. In addition, it is actually commented in Uniswap code that 
> * never overflows, and + overflow is desired

https://github.com/Uniswap/v2-core/blob/ee547b17853e71ed4e0101ccfd52e70d5acded58/contracts/UniswapV2Pair.sol#L77-L81
```solidity
if (timeElapsed > 0 && _reserve0 != 0 && _reserve1 != 0) {
    // * never overflows, and + overflow is desired
    price0CumulativeLast += uint(UQ112x112.encode(_reserve1).uqdiv(_reserve0)) * timeElapsed;
    price1CumulativeLast += uint(UQ112x112.encode(_reserve0).uqdiv(_reserve1)) * timeElapsed;
}
```

However, MaltDataLab contracts use Solidity 0.8 and will revert when overflow. It will break the `stabilize()` function and always revert since `stabilize()` call to MaltDataLab contract to get state.

Please note that, with Solidity 0.5.16, when result of addition bigger than `max(uint256)`, it will overflow without any errors. For example, `max(uint256) + 2 = 1`. 

So when `price0CumulativeLast` is overflow, the new value of `price0CumulativeLast` will smaller than old value. As the result, when MaltDataLab doing a subtraction to calculate current price, it might get revert.

## Proof of Concept
Function `stabilize()` will call to `MaltDataLab.trackPool()` first
https://github.com/code-423n4/2023-02-malt/blob/700f9b468f9cf8c9c5cffaa1eba1b8dea40503f9/contracts/StabilityPod/StabilizerNode.sol#L163

```solidity
function stabilize() external nonReentrant onlyEOA onlyActive whenNotPaused {
    // Ensure data consistency
    maltDataLab.trackPool();
    ...
}
```

Function `trackPool()` used a formula that will revert when `priceCumulative` overflow in Uniswap pool.
https://github.com/code-423n4/2023-02-malt/blob/700f9b468f9cf8c9c5cffaa1eba1b8dea40503f9/contracts/DataFeed/MaltDataLab.sol#L323-L329

```solidity
price = FixedPoint
    .uq112x112(
      uint224(  
        // @audit might overflow with solidity 0.8.0
        (priceCumulative - maltPriceCumulativeLast) / 
          (blockTimestampLast - maltPriceTimestampLast)
      )
    )
```

Scenario:
1. `maltPriceCumulativeLast = max(uint256 - 10)` and `price = 10, timeElapsed = 10`. So the new `priceCumulative = max(uint256 - 10) + 10 * 10 = 99 (overflow)`

2. When doing calculation in Malt protocol, `priceCumulative < maltPriceCumulativeLast`, so `priceCumulative - maltPriceCumulativeLast` will revert and fail

## Tools Used
Manual Review

## Recommended Mitigation Steps
Consider using `unchecked` block to match handle overflow calculation in Uniswap V2.
