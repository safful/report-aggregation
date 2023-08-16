## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`TimeswapPair.sol#mint()` Malicious user/attacker can mint new liquidity with an extremely small amount of `yIncrease` and malfunction the pair with the maturity](https://github.com/code-423n4/2022-01-timeswap-findings/issues/165) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Convenience/contracts/libraries/MintMath.sol#L14-L34

The current implementation of `TimeswapPair.sol#mint()` allows the caller to specify an arbitrary value for `yIncrease`.

However, since `state.y` is expected to be a large number based at `2**32`, once the initial `state.y` is set to a small number (1 wei for example), the algorithm won't effectively change `state.y` with regular market operations (`borrow`, `lend` and `mint`).

https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Core/contracts/libraries/BorrowMath.sol#L17-L37

The pair with the maturity will malfunction and can only be abandoned.

A malicious user/attacker can use this to frontrun other users or the platform's `newLiquidity()` call to initiate a griefing attack.

If the desired `maturity` is a meaningful value for the user/platform, eg, end of year/quarter. This can be a noteworthy issue.

## Recommendation

Consider adding validation of minimal `state.y` for new liquidity.

Can be `2**32 / 10000` for example.

