## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Moving logic to where required will save >=6800 gas on deposit/withdraw flows](https://github.com/code-423n4/2021-06-gro-findings/issues/36) 

# Handle

0xRajeev


# Vulnerability details

## Impact

In isValidBigFish(), the calculation of gvt and pard assets by making an external call to PnL.calcPnL() is required only if the amount is >= bigFishAbsoluteThreshold. 

Impact: Moving this logic for calculation of `assets` to the else part where it is required will save gas due to the external pnl call (2600 call + 2*2100 SLOADs for state variable reads in calcPnL()) for the sardine flow, where this is not required.

## Proof of Concept

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/Controller.sol#L250-L258

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/pnl/PnL.sol#L144-L146


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Move logic to else part instead of doing it before the conditional as shown below:
```
        if (amount < bigFishAbsoluteThreshold) {
            return false;
        } else if (amount > assets) {
            return true;
        } else {
            (uint256 gvtAssets, uint256 pwrdAssets) = IPnL(pnl).calcPnL();
            uint256 assets = pwrdAssets.add(gvtAssets);
            return amount > assets.mul(bigFishThreshold).div(PERCENTAGE_DECIMAL_FACTOR);
        }
```

