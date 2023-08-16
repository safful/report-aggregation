## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [`LpIssuer.sol#_chargeFees()` Check `if (performanceFee > 0)` can be done earlier to save gas](https://github.com/code-423n4/2021-12-mellow-findings/issues/90) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-12-mellow/blob/6679e2dd118b33481ee81ad013ece4ea723327b5/mellow-vaults/contracts/LpIssuer.sol#L249-L252

```solidity=249
uint256 performanceFee = strategyParams.performanceFee;
uint256[] memory hwms = _lpPriceHighWaterMarks;
if (performanceFee > 0) {
    uint256 minLpPriceFactor = type(uint256).max;
    ...
```

Check `if (performanceFee > 0)` at L251 can be done earlier to avoid unnecessary code execution (read `_lpPriceHighWaterMarks` and copy to memory) at L250 and save some gas when performanceFee == 0.

