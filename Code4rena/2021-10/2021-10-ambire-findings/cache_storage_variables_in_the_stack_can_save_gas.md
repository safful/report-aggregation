## Tags

- bug
- sponsor confirmed
- G (Gas Optimization)
- resolved

# [Cache storage variables in the stack can save gas](https://github.com/code-423n4/2021-10-ambire-findings/issues/30) 

# Handle

WatchPug


# Vulnerability details

For the storage variables that will be accessed multiple times, cache them in the stack can save ~100 gas from each extra read (`SLOAD` after Berlin).

For example:

- `scheduled[hashTx]` in `QuickAccManager.sol#cancel()`
    https://github.com/code-423n4/2021-10-ambire/blob/bc01af4df3f70d1629c4e22a72c19e6a814db70d/contracts/wallet/QuickAccManager.sol#L92-L92

- `scheduled[hash]` in `QuickAccManager.sol#execScheduled()`
    https://github.com/code-423n4/2021-10-ambire/blob/bc01af4df3f70d1629c4e22a72c19e6a814db70d/contracts/wallet/QuickAccManager.sol#L102-L102

