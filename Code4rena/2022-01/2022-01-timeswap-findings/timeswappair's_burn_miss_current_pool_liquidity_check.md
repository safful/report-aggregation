## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [TimeswapPair's burn miss current pool liquidity check](https://github.com/code-423n4/2022-01-timeswap-findings/issues/94) 

# Handle

hyh


# Vulnerability details

## Impact

If there is no liquidity in the pool, burn operation will not make sense and be reverted on low level math of decreasing liquidity share. It will be more transparent and uniform to add a check for liquidity before the logic.


## Proof of Concept

` burn` doesn't check for liquidity of the pool:

https://github.com/code-423n4/2022-01-timeswap/blob/main/Timeswap/Timeswap-V1-Core/contracts/TimeswapPair.sol#L205


## Recommended Mitigation Steps

There is a check in other relevant functions, it can be added in the very same form here:
` require(pool.state.totalLiquidity > 0, 'E206')`

Error description can be updated to include ` burn `:

https://github.com/code-423n4/2022-01-timeswap/blob/main/Timeswap/Timeswap-V1-Core/ErrorCodes.md#e206


