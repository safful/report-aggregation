## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Cache external call results can save gas](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/242) 

# Handle

WatchPug


# Vulnerability details

Every call to an external contract costs a decent amount of gas. For optimization of gas usage, external call results should be cached if they are being used for more than one time.

For example:

https://github.com/code-423n4/2021-11-bootfinance/blob/f102ee73eb320532c5a7c1e833f225c479577e39/customswap/contracts/SwapUtils.sol#L1163-L1270

- `self.lpToken.totalSupply()` can be cached.

