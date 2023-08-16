## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Cache storage variables in the stack can save gas](https://github.com/code-423n4/2021-12-sublime-findings/issues/117) 

# Handle

WatchPug


# Vulnerability details

For the storage variables that will be accessed multiple times, cache them in the stack can save ~100 gas from each extra read (`SLOAD` after Berlin).

For example:

- `wethGateway` in `AaveYield#_withdrawETH()`

    https://github.com/code-423n4/2021-12-sublime/blob/9df1b7c4247f8631647c7627a8da9bdc16db8b11/contracts/yield/AaveYield.sol#L307-L312

