## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Cache storage variables in the stack can save gas](https://github.com/code-423n4/2021-12-perennial-findings/issues/40) 

# Handle

WatchPug


# Vulnerability details

For the storage variables that will be accessed multiple times, cache them in the stack can save ~100 gas from each extra read (`SLOAD` after Berlin).

For example:

- `provider`, `factory()` in `Product#settleInternal()`

    https://github.com/code-423n4/2021-12-perennial/blob/fd7c38823833a51ae0c6ae3856a3d93a7309c0e4/protocol/contracts/product/Product.sol#L72-L100


- `provider`, `_accumulator` in `Product#settleAccountInternal()`

    https://github.com/code-423n4/2021-12-perennial/blob/fd7c38823833a51ae0c6ae3856a3d93a7309c0e4/protocol/contracts/product/Product.sol#L123-L152


- `factory()` in `Collateral#settleProduct()`

    https://github.com/code-423n4/2021-12-perennial/blob/fd7c38823833a51ae0c6ae3856a3d93a7309c0e4/protocol/contracts/collateral/Collateral.sol#L131-L145

