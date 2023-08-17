## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [Undercollateralized loans possible](https://github.com/code-423n4/2022-04-dualityfocus-findings/issues/12) 

# Lines of code

https://github.com/code-423n4/2022-04-dualityfocus/blob/f21ef7708c9335ee1996142e2581cb8714a525c9/contracts/compound_rari_fork/Comptroller.sol#L1491


# Vulnerability details

## Impact
The `_setPoolCollateralFactors` function does not check that the collateral factor is < 100%.
It's possible that it's set to 200% and then borrows more than the collateral is worth, stealing from the pool.

## Recommended Mitigation Steps
Disable the possibility of ever having a collateral factor > 100% by checking:

```diff
for (uint256 i = 0; i < pools.length; i++) {
+   require(collateralFactorsMantissa[i] <= 1e18, "CF > 100%");
    poolCollateralFactors[pools[i]] = collateralFactorsMantissa[i];
}
```


