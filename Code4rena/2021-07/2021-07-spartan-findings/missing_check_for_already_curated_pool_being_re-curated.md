## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Missing check for already curated pool being re-curated](https://github.com/code-423n4/2021-07-spartan-findings/issues/137) 

# Handle

0xRajeev


# Vulnerability details

## Impact

addCuratedPool() is missing a require(isCuratedPool[_pool] == false) check, similar to the one in removeCuratedPool to ensure that the DAO is not trying to curate an already curated pool which indicates a mismatch of assumption/accounting compared to the contract state.

## Proof of Concept

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/poolFactory.sol#L79-L87

https://github.com/code-423n4/2021-07-spartan/blob/e2555aab44d9760fdd640df9095b7235b70f035e/contracts/poolFactory.sol#L93


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Add require(isCuratedPool[_pool] == false) before setting isCuratedPool[_pool] = true.

