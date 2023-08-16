## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Inline unnecessary function can make the code simpler and save some gas](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/278) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-12-yetifinance/blob/5f5bf61209b722ba568623d8446111b1ea5cb61c/packages/contracts/contracts/YUSDToken.sol#L297-L301

```solidity=297
    function _requireCallerIsTMLorSP() internal view {
        require(
            msg.sender == stabilityPoolAddress || msg.sender == troveManagerLiquidationsAddress,
            "YUSD: Caller is neither TroveManagerLiquidator nor StabilityPool");
    }
```

https://github.com/code-423n4/2021-12-yetifinance/blob/5f5bf61209b722ba568623d8446111b1ea5cb61c/packages/contracts/contracts/YUSDToken.sol#L128-L131

```solidity=128
    function returnFromPool(address _poolAddress, address _receiver, uint256 _amount) external override {
        _requireCallerIsTMLorSP();
        _transfer(_poolAddress, _receiver, _amount);
    }
```


`_requireCallerIsTMLorSP()` is unnecessary as it's being used only once. Therefore it can be inlined in `returnFromPool()` to make the code simpler and save gas.

## Recommendation

Change to:

```solidity=128
    function returnFromPool(address _poolAddress, address _receiver, uint256 _amount) external override {
        require(
            msg.sender == stabilityPoolAddress || msg.sender == troveManagerLiquidationsAddress,
            "YUSD: Caller is neither TroveManagerLiquidator nor StabilityPool");
        _transfer(_poolAddress, _receiver, _amount);
    }
```

Other examples include:

-   `TroveManagerRedemptions.sol#_getRedemptionFee()` can be inlined in `TroveManagerRedemptions.sol#redeemCollateral()`

