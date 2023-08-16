## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [IdleYieldSource doesn't use mantissa calculations](https://github.com/code-423n4/2021-06-pooltogether-findings/issues/103) 

# Handle

tensors


# Vulnerability details

## Impact
Because mantissa calculations are not used in this case to account for decimals, the arithmetic
can zero out the number of shares or tokens that should be given.

For example, say I deposit 1 token, expecting 1 share in return.
On L95, if the totalunderlying assets is increased to be larger than the number of total shares, then the division would output 0 and I wouldn't get any shares.  

## Proof of Concept
https://github.com/sunnyRK/IdleYieldSource-PoolTogether/blob/6dcc419e881a4f0f205c07c58f4db87520b6046d/contracts/IdleYieldSource.sol#L95

https://github.com/sunnyRK/IdleYieldSource-PoolTogether/blob/6dcc419e881a4f0f205c07c58f4db87520b6046d/contracts/IdleYieldSource.sol#L106

## Recommended Mitigation Steps
Implement mantissa calculations like in the contract for the AAVE  yield.

