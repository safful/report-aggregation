## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- VaderRouter

# [`VaderRouter._swap` performs wrong swap](https://github.com/code-423n4/2021-11-vader-findings/issues/161) 

# Handle

cmichel


# Vulnerability details

The 3-path hop in `VaderRouter._swap` is supposed to first swap **foreign** assets to native assets, and then the received native assets to different foreign assets again.

The `pool.swap(nativeAmountIn, foreignAmountIn)` accepts the foreign amount as the **second** argument.
The code however mixes these positional arguments up and tries to perform a `pool0` foreign -> native swap by using the **foreign** amount as the **native amount**:

```solidity
function _swap(
    uint256 amountIn,
    address[] calldata path,
    address to
) private returns (uint256 amountOut) {
    if (path.length == 3) {
      // ...
      // @audit calls this with nativeAmountIn = amountIn. but should be foreignAmountIn (second arg)
      return pool1.swap(0, pool0.swap(amountIn, 0, address(pool1)), to);
    }
}

// @audit should be this instead
return pool1.swap(pool0.swap(0, amountIn, address(pool1)), 0, to);
```

## Impact
All 3-path swaps through the `VaderRouter` fail in the pool check when `require(nativeAmountIn = amountIn <= nativeBalance - nativeReserve = 0)` is checked, as foreign amount is sent but _native_ amount is specified.

## Recommended Mitigation Steps
Use `return pool1.swap(pool0.swap(0, amountIn, address(pool1)), 0, to);` instead.

