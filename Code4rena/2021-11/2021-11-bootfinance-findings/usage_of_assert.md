## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [Usage of assert](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/279) 

# Handle

pauliax


# Vulnerability details

## Impact
Contracts use assert() instead of require() in multiple places. Assert is recommended to be used to check for internal errors, or to check invariants. 
In your case, I think these validations could better use 'require' as they are likely to be triggered:
assert(claimable > 0);
assert(airdrop[msg.sender].amount - claimable != 0);
assert(block.timestamp - startEpochTime <= RATE_TIME);
assert(block.timestamp - initTime >= YEAR * 5);

A similar issue was submitted in a previous contest and was assigned a severity of low: https://github.com/code-423n4/2021-06-realitycards-findings/issues/83

## Recommended Mitigation Steps
Consider replacing 'assert' with 'require' in the cases mentioned above.

