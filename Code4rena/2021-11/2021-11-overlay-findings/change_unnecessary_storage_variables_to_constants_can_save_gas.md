## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Change unnecessary storage variables to constants can save gas](https://github.com/code-423n4/2021-11-overlay-findings/issues/85) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-11-overlay/blob/1833b792caf3eb8756b1ba5f50f9c2ce085e54d0/contracts/OverlayV1UniswapV3Market.sol#L14-L14

```solidity=14
uint256 internal X96 = 0x1000000000000000000000000;
```

Some storage variables include `X96` will not never be changed and they should not be.

Changing them to `constant` can save gas.

