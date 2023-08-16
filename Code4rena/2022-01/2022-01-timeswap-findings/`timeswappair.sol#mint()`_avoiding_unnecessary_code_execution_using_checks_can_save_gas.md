## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [`TimeswapPair.sol#mint()` Avoiding unnecessary code execution using checks can save gas](https://github.com/code-423n4/2022-01-timeswap-findings/issues/155) 

# Handle

WatchPug


# Vulnerability details

Move storage writes to inside the code block of `if (tokensOut.asset > 0) {...}` can avoid unnecessary code execution when this check doesn't pass and save gas.

https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Core/contracts/TimeswapPair.sol#L214-L218

```solidity
        pool.state.reserves.asset -= tokensOut.asset;
        pool.state.reserves.collateral -= tokensOut.collateral;

        if (tokensOut.asset > 0) asset.safeTransfer(assetTo, tokensOut.asset);
        if (tokensOut.collateral > 0) collateral.safeTransfer(collateralTo, tokensOut.collateral);
```
### Recommendation

Change to:

```solidity
        if (tokensOut.asset > 0) {
                pool.state.reserves.asset -= tokensOut.asset;
                asset.safeTransfer(assetTo, tokensOut.asset);
        }
        if (tokensOut.collateral > 0) {
                pool.state.reserves.collateral -= tokensOut.collateral;
                collateral.safeTransfer(collateralTo, tokensOut.collateral);
        }
```



