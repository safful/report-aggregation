## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Avoid unnecessary arithmetic operations can save gas](https://github.com/code-423n4/2021-12-maple-findings/issues/65) 

# Handle

WatchPug


# Vulnerability details

In `MapleLoanInternals.sol#_getCollateralRequiredFor`, when `principal_ <= drawableFunds_`, `0 / principalRequested_` can be avoid.

https://github.com/maple-labs/loan/blob/9684bcef06481e493d060974b1777a4517c4e792/contracts/MapleLoanInternals.sol#L359-L359

```solidity=359
return (collateralRequired_ * (principal_ > drawableFunds_ ? principal_ - drawableFunds_ : uint256(0))) / principalRequested_;
```

### Recommendation

Change to:

```solidity
if (principal_ <= drawableFunds_) {
    return uint256(0);
}

return (collateralRequired_ * (principal_ - drawableFunds_)) / principalRequested_;
```

