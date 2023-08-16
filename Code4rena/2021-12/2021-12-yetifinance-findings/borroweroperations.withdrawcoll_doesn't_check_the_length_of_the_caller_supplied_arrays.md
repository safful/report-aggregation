## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [BorrowerOperations.withdrawColl doesn't check the length of the caller supplied arrays](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/96) 

# Handle

hyh


# Vulnerability details

# Impact

On calling with arrays of different lengths various malfunctions are possible as the arrays are used as given.
`withdrawColl` outcome will not be as expected by a caller.

## Proof of Concept

`_adjustTrove` doesn't check for array lengths and all other array providing usages of this function do check them before usage.

BorrowerOperations.withdrawColl doesn't:
https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/BorrowerOperations.sol#L373

## Recommended Mitigation Steps

Add the check:

Now:
```
params._collsOut = _collsOut;
params._amountsOut = _amountsOut;
```

To be:
```
require(_collsOut.length == _amountsOut.length);
params._collsOut = _collsOut;
params._amountsOut = _amountsOut;
```


