## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [`uint64(block.timestamp % 2**64)` can be simpler and save some gas](https://github.com/code-423n4/2021-11-malt-findings/issues/273) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/MovingAverage.sol#L154-L154

```solidity=154
uint64 blockTimestamp = uint64(block.timestamp % 2**64); 
```

Use `uint64(n)` can cut off higher-order bits already, `n % 2**64` is redundant.

See: https://docs.soliditylang.org/en/v0.8.10/types.html#explicit-conversions

### Recommendation

Change to:

```solidity=154
uint64 blockTimestamp = uint64(block.timestamp); 
```

