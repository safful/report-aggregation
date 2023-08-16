## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Unused storage variables](https://github.com/code-423n4/2021-11-malt-findings/issues/287) 

# Handle

WatchPug


# Vulnerability details

Unused storage variables in contracts use up storage slots and increase contract size and gas usage at deployment and initialization.

Instances include:

https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/StabilizerNode.sol#L57-L57
```solidity=57
address public uniswapV2Factory;
```

