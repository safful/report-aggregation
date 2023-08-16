## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Missing checks on debt max/min limits could cause pour to revert](https://github.com/code-423n4/2021-05-yield-findings/issues/41) 

# Handle

0xRajeev


# Vulnerability details

## Impact

setDebtLimits() is used to set the maximum and minimum debt for an underlying and ilk pair. The assumption is that max will be greater than min while setting them because otherwise the debt checks in _pour() for line/dust will fail and revert.

While max and min debt limits can be reset, it is safer to perform input validation on them in setDebtLimits().

## Proof of Concept

A recipe incorrectly interchanges the values of min and max debt which leads to exceptions in pouring into the vaults.

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/Cauldron.sol#L91-L92

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/Cauldron.sol#L319-L322

https://github.com/code-423n4/2021-05-yield/blob/e4c8491cd7bfa5dc1b59eb1b257161cd5bf8c6b0/contracts/Wand.sol#L79


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Add a check to ensure max > mix.

