## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Use immutable variables can save gas](https://github.com/code-423n4/2021-12-perennial-findings/issues/29) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-12-perennial/blob/fd7c38823833a51ae0c6ae3856a3d93a7309c0e4/protocol/contracts/oracle/ChainlinkOracle.sol#L20-L20
```solidity=20
IChainlinkFeed public feed;
```

https://github.com/code-423n4/2021-12-perennial/blob/fd7c38823833a51ae0c6ae3856a3d93a7309c0e4/protocol/contracts/oracle/ChainlinkOracle.sol#L29-L29
```solidity=29
uint256 private _decimalOffset;
```

In `ChainlinkOracle.sol`, `feed` and `_decimalOffset` will never change, use immutable variables instead of storage variables can save gas.


https://github.com/code-423n4/2021-12-perennial/blob/fd7c38823833a51ae0c6ae3856a3d93a7309c0e4/protocol/contracts/product/ProductProviderBase.sol#L13-L13

```solidity=13
IOracle public oracle;
```

In `ProductProviderBase.sol`, `oracle` will never change, use immutable variables instead of storage variables can save gas.

