## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [Inconsistent check of token balance](https://github.com/code-423n4/2021-11-streaming-findings/issues/249) 

# Handle

WatchPug


# Vulnerability details

`require(newBal <= type(uint112).max ...)` vs `require(newBal < type(uint112).max...)`.

https://github.com/code-423n4/2021-11-streaming/blob/56d81204a00fc949d29ddd277169690318b36821/Streaming/src/Locke.sol#L386-L386
```solidity=386
require(newBal < type(uint112).max && newBal > prevBal, "erc");
```

https://github.com/code-423n4/2021-11-streaming/blob/56d81204a00fc949d29ddd277169690318b36821/Streaming/src/Locke.sol#L427-L427
```solidity=427
require(newBal <= type(uint112).max && newBal > prevBal, "erc");
```

https://github.com/code-423n4/2021-11-streaming/blob/56d81204a00fc949d29ddd277169690318b36821/Streaming/src/Locke.sol#L506-L506
```solidity=506
require(newBal <= type(uint112).max && newBal > prevBal, "erc");
```

