## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`withdrawFromWETH` always reverts ](https://github.com/code-423n4/2021-09-sushitrident-findings/issues/90) 

# Handle

cmichel


# Vulnerability details

The `TridentHelper.withdrawFromWETH` (used in `TridentRouter.unwrapWETH`) function performs a low-level call to `WETH.withdraw(amount)`.

It then checks if the return `data` length is more or equal to `32` bytes, however `WETH.withdraw` returns `void` and has a return value of `0`.
Thus, the function always reverts even if `success == true`.

```solidity
function withdrawFromWETH(uint256 amount) internal {
    // @audit WETH.withdraw returns nothing, data.length always zero. this always reverts
    require(success && data.length >= 32, "WITHDRAW_FROM_WETH_FAILED");
}
```

## Impact
The `unwrapWETH` function is broken and makes all transactions revert.
Batch calls to the router cannot perform any unwrapping of WETH.

## Recommended Mitigation Steps
Remove the `data.length >= 32` from the require and only check if `success` is true.

