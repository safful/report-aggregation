## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- VaderRouter

# [`VaderRouter.calculateOutGivenIn` calculates wrong swap](https://github.com/code-423n4/2021-11-vader-findings/issues/162) 

# Handle

cmichel


# Vulnerability details

The 3-path hop in `VaderRouter.calculateOutGivenIn` is supposed to first swap **foreign** assets to native assets **in pool0**, and then the received native assets to different foreign assets again **in pool1**.

The first argument of `VaderMath.calculateSwap(amountIn, reserveIn, reserveOut)` must refer to the same token as the second argument `reserveIn`.
The code however mixes these positions up and first performs a swap in `pool1` instead of `pool0`:

```solidity
function calculateOutGivenIn(uint256 amountIn, address[] calldata path)
    external
    view
    returns (uint256 amountOut)
{
  if(...) {
  } else {
    return
        VaderMath.calculateSwap(
            VaderMath.calculateSwap(
                // @audit the inner trade should not be in pool1 for a forward swap. amountIn foreign => next param should be foreignReserve0
                amountIn,
                nativeReserve1,
                foreignReserve1
            ),
            foreignReserve0,
            nativeReserve0
        );
  }

 /** @audit instead should first be trading in pool0!
    VaderMath.calculateSwap(
        VaderMath.calculateSwap(
            amountIn,
            foreignReserve0,
            nativeReserve0
        ),
        nativeReserve1,
        foreignReserve1
    );
  */
```

## Impact
All 3-path swaps computations through `VaderRouter.calculateOutGivenIn` will return the wrong result.
Smart contracts or off-chain scripts/frontends that rely on this value to trade will have their transaction reverted, or in the worst case lose funds.

## Recommended Mitigation Steps
Return the following code instead which first trades in pool0 and then in pool1:

```solidity
return
  VaderMath.calculateSwap(
      VaderMath.calculateSwap(
          amountIn,
          foreignReserve0,
          nativeReserve0
      ),
      nativeReserve1,
      foreignReserve1
  );
```


