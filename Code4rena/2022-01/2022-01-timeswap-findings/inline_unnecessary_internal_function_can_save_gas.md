## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Inline unnecessary internal function can save gas](https://github.com/code-423n4/2022-01-timeswap-findings/issues/163) 

# Handle

WatchPug


# Vulnerability details

`checkProportional()` is a rather simple one line function, making it inline instead of an internal function call can make the code simpler and save some gas.

https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Core/contracts/TimeswapPair.sol#L359-L368

```solidity
for (uint256 i; i < ids.length; i++) {
    Due storage due = dues[ids[i]];
    require(due.startBlock != BlockNumber.get(), 'E207');
    if (owner != msg.sender) require(collateralsOut[i] == 0, 'E213');
    PayMath.checkProportional(assetsIn[i], collateralsOut[i], due);
    due.debt -= assetsIn[i];
    due.collateral -= collateralsOut[i];
    assetIn += assetsIn[i];
    collateralOut += collateralsOut[i];
}
```

https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Core/contracts/libraries/PayMath.sol#L7-L14

```solidity
function checkProportional(
    uint112 assetIn,
    uint112 collateralOut,
    IPair.Due memory due
) internal pure {
    require(uint256(assetIn) * due.collateral >= uint256(collateralOut) * due.debt, 'E303');
}
}
```

Can be changed to:

```solidity
for (uint256 i; i < ids.length; i++) {
    Due storage due = dues[ids[i]];
    require(due.startBlock != BlockNumber.get(), 'E207');
    if (owner != msg.sender) require(collateralsOut[i] == 0, 'E213');
    require(uint256(assetIn[i]) * due.collateral >= uint256(collateralOut[i]) * due.debt, 'E303');
    due.debt -= assetsIn[i];
    due.collateral -= collateralsOut[i];
    assetIn += assetsIn[i];
    collateralOut += collateralsOut[i];
}
```


