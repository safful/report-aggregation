## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Adding a new method `provider.currentPrice()` can save gas](https://github.com/code-423n4/2021-12-perennial-findings/issues/27) 

# Handle

WatchPug


# Vulnerability details

Every call to an external contract costs a decent amount of gas.

There are many times across the codebase that will perform two external calls to the provider to query the current `oraclePrice`:

https://github.com/code-423n4/2021-12-perennial/blob/fd7c38823833a51ae0c6ae3856a3d93a7309c0e4/protocol/contracts/product/types/position/AccountPosition.sol#L72-L72

```solidity=72
Fixed18 oraclePrice = provider.priceAtVersion(provider.currentVersion());
```

Consider adding a new method to the provider and return the current `oraclePrice` directly can combine two external calls into one and save some gas.

