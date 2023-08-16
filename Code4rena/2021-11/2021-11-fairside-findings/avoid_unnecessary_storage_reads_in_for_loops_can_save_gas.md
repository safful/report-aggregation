## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Avoid unnecessary storage reads in for loops can save gas](https://github.com/code-423n4/2021-11-fairside-findings/issues/66) 

# Handle

WatchPug


# Vulnerability details

For the storage variables that will be accessed multiple times, especially in for loops, cache and read from the stack can save ~100 gas from each extra read (`SLOAD` after Berlin).

For example:

https://github.com/code-423n4/2021-11-fairside/blob/20c68793f48ee2678508b9d3a1bae917c007b712/contracts/dependencies/TributeAccrual.sol#L77-L88

```solidity=77
    function totalAvailableTribute(uint256 offset)
        external
        view
        override
        returns (uint256 total)
    {
        for (uint256 i = offset; i < totalTributes; i++)
            total = total.add(availableTribute(i));

        for (uint256 i = offset; i < totalGovernanceTributes; i++)
            total = total.add(availableGovernanceTribute(i));
    }
```

`totalTributes` and `totalGovernanceTributes` can be cached.

