## Tags

- bug
- disagree with severity
- G (Gas Optimization)
- sponsor confirmed

# [Redundant type casting](https://github.com/code-423n4/2022-01-trader-joe-findings/issues/235) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2022-01-trader-joe/blob/a1579f6453bc4bf9fb0db9c627beaa41135438ed/contracts/LaunchEvent.sol#L82-L82

```solidity
IJoeFactory private factory;
```

https://github.com/code-423n4/2022-01-trader-joe/blob/a1579f6453bc4bf9fb0db9c627beaa41135438ed/contracts/LaunchEvent.sol#L385-L385

```solidity
IJoeFactory(factory).getPair(wavaxAddress, tokenAddress)
```

`factory` is defined as `IJoeFactory` already, the type casting is redundant.

