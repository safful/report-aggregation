## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Unused imports](https://github.com/code-423n4/2022-01-timeswap-findings/issues/159) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Core/contracts/libraries/Callback.sol#L5-L5

```solidity
import {IPair} from '../interfaces/IPair.sol';
```

`IPair` is unused in `Callback.sol`.

