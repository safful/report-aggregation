## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [`UToken.sol` should inherits and complies with `IUToken.sol`](https://github.com/code-423n4/2021-10-union-findings/issues/77) 

# Handle

WatchPug


# Vulnerability details

In the current implementation, `UToken.sol` does not inherit and comply with `IUToken.sol`. This is against the best practices and inconsistent with other contracts in the codebase that do inherit and comply with their interfaces.

For example, the `repay()` function defined in `IUToken.sol` is implementated as `repayBorrowBehalf()` and `repayBorrow()`.

It makes the `IUToken.sol` unable to be used and misleading.

### Recommendation

Make `UToken.sol` inherits and complies with `IUToken.sol`.

