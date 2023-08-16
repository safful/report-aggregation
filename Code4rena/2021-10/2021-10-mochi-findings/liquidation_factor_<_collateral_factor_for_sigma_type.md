## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [liquidation factor < collateral factor for Sigma type](https://github.com/code-423n4/2021-10-mochi-findings/issues/126) 

# Handle

cmichel


# Vulnerability details

The `MochiProfileV0` defines liquidation and collateral factors for different asset types.
For the `AssetClass.Sigma` type, the liquidation factor is _less_ than the collateral factor:

```solidity
function liquidationFactor(address _asset)
    public
    view
    override
    returns (float memory)
{
    AssetClass class = assetClass(_asset);
    if (class == AssetClass.Sigma) { // } else if (class == AssetClass.Sigma) {
        return float({numerator: 40, denominator: 100});
    }
}

function maxCollateralFactor(address _asset)
    public
    view
    override
    returns (float memory)
{
    AssetClass class = assetClass(_asset);
    if (class == AssetClass.Sigma) {
        return float({numerator: 45, denominator: 100});
    }
}
```

This means that one can take a loan of up to 45% of their collateral but then immediately gets liquidated as the liquidation factor is only 40%.
There should always be a buffer between these such that taking the max loan does not immediately lead to liquidations:

> A safety buffer is maintained between max CF and LF to protect users against liquidations due to normal volatility. [Docs](https://hackmd.io/@az-/mochi-whitepaper#Collateral-Factor-CF)


## Recommended Mitigation Steps
The max collateral factor for the Sigma type should be higher than its liquidation factor.


