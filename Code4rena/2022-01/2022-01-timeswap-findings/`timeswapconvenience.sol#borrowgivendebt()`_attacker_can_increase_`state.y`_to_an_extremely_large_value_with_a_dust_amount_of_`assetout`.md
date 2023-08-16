## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [`TimeswapConvenience.sol#borrowGivenDebt()` Attacker can increase `state.y` to an extremely large value with a dust amount of `assetOut`](https://github.com/code-423n4/2022-01-timeswap-findings/issues/169) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Convenience/contracts/libraries/BorrowMath.sol#L19-L53

This issue is similar to the two previous issues related to `state.y` manipulation. Unlike the other two issues, this function is not on `TimeswapPair.sol` but on `TimeswapConvenience.sol`, therefore this can not be solved by adding `onlyConvenience` modifier.

Actually, we believe that it does not make sense for the caller to specify the interest they want to pay, we recommend removing this function.

## Impact

- When `pool.state.y` is extremely large, many core features of the protocol will malfunction, as the arithmetic related to `state.y` can overflow. For example:

LendMath.check(): https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Core/contracts/libraries/LendMath.sol#L28-L28

BorrowMath.check(): https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Core/contracts/libraries/BorrowMath.sol#L31-L31

- An attacker can set `state.y` to a near overflow value, then `lend()` to get a large amount of extra interest (as Bond tokens) with a small amount of asset tokens. This way, the attacker can steal funds from other lenders and liquidity providers.


