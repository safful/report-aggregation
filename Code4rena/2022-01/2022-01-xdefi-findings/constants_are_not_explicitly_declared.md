## Tags

- bug
- 0 (Non-critical)
- resolved
- sponsor confirmed

# [Constants are not explicitly declared](https://github.com/code-423n4/2022-01-xdefi-findings/issues/115) 

# Handle

WatchPug


# Vulnerability details

It's a best practice to use constant variables rather than literal values to make the code easier to understand and maintain.

Consider defining a constant variable for the literal value used and giving it a clear and self-explanatory name.

Instances include:

https://github.com/XDeFi-tech/xdefi-distribution/blob/3856a42df295183b40c6eee89307308f196612fe/contracts/XDEFIDistribution.sol#L82-L82

```solidity
require(duration <= uint256(18250 days), "INVALID_DURATION");
```

Consider changing `uint256(18250 days)` to `MAX_DURATION` constant.


