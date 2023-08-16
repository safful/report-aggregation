## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Cache array length in for loops can save gas](https://github.com/code-423n4/2022-01-timeswap-findings/issues/151) 

# Handle

WatchPug


# Vulnerability details

Reading array length at each iteration of the loop takes 6 gas (3 for mload and 3 to place memory_offset) in the stack.

Caching the array length in the stack saves around 3 gas per iteration.

Instances include:

- `TimeswapPair.sol#pay()`

    https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Core/contracts/TimeswapPair.sol#L359-L359


- `PayMath.sol#givenMaxAssetsIn()`

    https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Convenience/contracts/libraries/PayMath.sol#L21-L21


