## Tags

- bug
- disagree with severity
- 2 (Med Risk)
- sponsor confirmed
- filed

# [Interest debt is capped after a year](https://github.com/code-423n4/2021-04-vader-findings/issues/219) 

# Handle

@cmichelio


# Vulnerability details


## Vulnerability Details

The `Utils.getInterestOwed` function computes the `_interestPayment` as:

```solidity
uint256 _interestPayment =
  calcShare(
      timeElapsed,
      _year,
      getInterestPayment(collateralAsset, debtAsset)
  ); // Share of the payment over 1 year
```

However, `calcShare` caps `timeElpased` to `_year` and therefore the owed interest does not grow after a year has elapsed.

## Impact

The impact is probably small because the only call so far computes the elapsed time as `block.timestamp - mapCollateralAsset_NextEra[collateralAsset][debtAsset];` which most likely will never go beyond a year.

It's still recommended to fix the logic bug in case more functions will be added that use the broken function.

## Recommended Mitigation Steps

Use a different function than `calcShare` that does not cap.


