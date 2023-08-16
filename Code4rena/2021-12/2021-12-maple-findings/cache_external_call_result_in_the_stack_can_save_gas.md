## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Cache external call result in the stack can save gas](https://github.com/code-423n4/2021-12-maple-findings/issues/62) 

# Handle

WatchPug


# Vulnerability details

For the result of an external call being written into a storage variable, cache and read from the stack rather than read from the storage variable can save gas.

Instances include:

`IERC20Like(collateralAsset).decimals()` in `DebtLocker.sol#getExpectedAmount()` can be cached to avoid an extra external call.

https://github.com/maple-labs/debt-locker/blob/81f55907db7b23d27e839b9f9f73282184ed4744/contracts/DebtLocker.sol#L237-L253

```solidity=231
function getExpectedAmount(uint256 swapAmount_) external view override whenProtocolNotPaused returns (uint256 returnAmount_) {
    address collateralAsset = IMapleLoanLike(_loan).collateralAsset();
    address fundsAsset      = IMapleLoanLike(_loan).fundsAsset();

    uint256 oracleAmount =
        swapAmount_
            * IMapleGlobalsLike(_getGlobals()).getLatestPrice(collateralAsset)  // Convert from `fromAsset` value.
            * 10 ** IERC20Like(fundsAsset).decimals()                           // Convert to `toAsset` decimal precision.
            * (10_000 - _allowedSlippage)                                       // Multiply by allowed slippage basis points
            / IMapleGlobalsLike(_getGlobals()).getLatestPrice(fundsAsset)       // Convert to `toAsset` value.
            / 10 ** IERC20Like(collateralAsset).decimals()                      // Convert from `fromAsset` decimal precision.
            / 10_000;                                                           // Divide basis points for slippage

    uint256 minRatioAmount = swapAmount_ * _minRatio / 10 ** IERC20Like(collateralAsset).decimals();

    return oracleAmount > minRatioAmount ? oracleAmount : minRatioAmount;
}
```

### Recommendation

Change to:

```solidity=231
function getExpectedAmount(uint256 swapAmount_) external view override whenProtocolNotPaused returns (uint256 returnAmount_) {
    address collateralAsset = IMapleLoanLike(_loan).collateralAsset();
    address fundsAsset      = IMapleLoanLike(_loan).fundsAsset();

    uint256 collateralAssetDecimals = IERC20Like(collateralAsset).decimals();

    uint256 oracleAmount =
        swapAmount_
            * IMapleGlobalsLike(_getGlobals()).getLatestPrice(collateralAsset)  // Convert from `fromAsset` value.
            * 10 ** IERC20Like(fundsAsset).decimals()                           // Convert to `toAsset` decimal precision.
            * (10_000 - _allowedSlippage)                                       // Multiply by allowed slippage basis points
            / IMapleGlobalsLike(_getGlobals()).getLatestPrice(fundsAsset)       // Convert to `toAsset` value.
            / 10 ** collateralAssetDecimals                                     // Convert from `fromAsset` decimal precision.
            / 10_000;                                                           // Divide basis points for slippage

    uint256 minRatioAmount = swapAmount_ * _minRatio / 10 ** collateralAssetDecimals;

    return oracleAmount > minRatioAmount ? oracleAmount : minRatioAmount;
}
```

