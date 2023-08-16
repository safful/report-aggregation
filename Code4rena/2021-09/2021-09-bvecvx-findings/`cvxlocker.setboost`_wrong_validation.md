## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- sponsor acknowledged

# [`CvxLocker.setBoost` wrong validation](https://github.com/code-423n4/2021-09-bvecvx-findings/issues/51) 

# Handle

cmichel


# Vulnerability details

## Vulnerability Details
The `CvxLocker.setBoost` function does not validate the `_max, _rate` parameters, instead it validates the already set **storage** variables.

```solidity
// @audit this is checking the already-set storage variables, not the parameters
require(maximumBoostPayment < 1500, "over max payment"); //max 15%
require(boostRate < 30000, "over max rate"); //max 3x
```

## Impact
Once wrong boost values are set (which are not validated when they are set), they cannot be set to new values anymore, breaking core contract functionality.

## Recommended Mitigation Steps
Implement these two checks instead:

```solidity
require(_max < 1500, "over max payment"); //max 15%
require(_rate < 30000, "over max rate"); //max 3x
```


