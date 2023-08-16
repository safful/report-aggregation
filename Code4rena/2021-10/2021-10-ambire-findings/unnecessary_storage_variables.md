## Tags

- bug
- sponsor confirmed
- G (Gas Optimization)
- resolved

# [Unnecessary storage variables](https://github.com/code-423n4/2021-10-ambire-findings/issues/29) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-10-ambire/blob/bc01af4df3f70d1629c4e22a72c19e6a814db70d/contracts/wallet/Zapper.sol#L69-L72

Some storage variables include `admin`, `lendingPool` and `aaveRefCode` are unnecessary as they will never be changed.

Change to `immutable` can save gas.

