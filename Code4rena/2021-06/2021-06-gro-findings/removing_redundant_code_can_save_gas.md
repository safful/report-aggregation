## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Removing redundant code can save gas](https://github.com/code-423n4/2021-06-gro-findings/issues/42) 

# Handle

0xRajeev


# Vulnerability details

## Impact

In LG setDependencies(), the code to approve withdrawHandler to pull from lifeguard is repeated twice, once to set it to 0 allowance if the withdrawHandler is != 0 and then unconditionally to set it MAX. Given that this is the only function that sets withdrawHandler, the first set of 0 approvals seem to be redundant given the unconditional approvals that follow. Removing this can save some gas although we don’t expect this to be called often.

The redundant logic could be for the case where the withdrawHandler is updated and the old handler is given an approval of 0 and the new one MAX.


## Proof of Concept

https://github.com/code-423n4/2021-06-gro/blob/091660467fc8d13741f8aafcec80f1e8cf129a33/contracts/pools/LifeGuard3Pool.sol#L78-L89


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Evaluate code and remove logic if redundant. If this is present to handle withdrawHandler updates then ignore this recommendation.

