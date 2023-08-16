## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Gas Optimization: Unnecessary variables](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/299) 

# Handle

gzeon


# Vulnerability details

## Impact
The 3 variable defined in L365-367 are used only once
https://github.com/code-423n4/2021-12-yetifinance/blob/5f5bf61209b722ba568623d8446111b1ea5cb61c/packages/contracts/contracts/Dependencies/Whitelist.sol#L365-367
```
        uint256 price = getPrice(_collateral);
        uint256 decimals = collateralParams[_collateral].decimals;
        uint256 ratio = collateralParams[_collateral].ratio;
```

We can skip them and do everything inline:
```
return (getPrice(_collateral).mul(_amount).mul(collateralParams[_collateral].ratio).div(10**(18 + collateralParams[_collateral].decimals)));
```

Similarly, L352-354
```
        return getPrice(_collateral).mul(_amount).div(10**collateralParams[_collateral].decimals);
```

