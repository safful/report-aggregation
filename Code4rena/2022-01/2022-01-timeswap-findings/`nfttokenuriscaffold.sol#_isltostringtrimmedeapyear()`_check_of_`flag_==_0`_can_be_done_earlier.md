## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [`NFTTokenURIScaffold.sol#_isLtoStringTrimmedeapYear()` Check of `flag == 0` can be done earlier](https://github.com/code-423n4/2022-01-timeswap-findings/issues/157) 

# Handle

WatchPug


# Vulnerability details

`flag == 0` is cheaper than `temp % 10 == 0`. 

Therefore, checking `flag == 0 `first can save some gas.

https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Convenience/contracts/libraries/NFTTokenURIScaffold.sol#L162-L162

```solidity
if (temp % 10 == 0 && flag == 0)
```

### Recommendation

Change to:

```solidity
    if (flag == 0 && temp % 10 == 0) 
```

Other instances include:

https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Convenience/contracts/libraries/NFTTokenURIScaffold.sol#L180-L180

```solidity
else if (value % 10 != 0 && flag == 0) 
```

