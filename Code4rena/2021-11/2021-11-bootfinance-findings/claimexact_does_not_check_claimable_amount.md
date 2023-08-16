## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [claimExact does not check claimable amount](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/126) 

# Handle

nathaniel


# Vulnerability details

## Impact
Inconsistency between the `claim()` function and `claimExact()` function, in that `claimExact` does not check the claimable amount. In the scenario where claimable = 0, and `investors[msg.sender].claimed != 0` then it will attempt to underflow.
If `_amount` is 0, then it could potentially reach the `vestLock.vest()` function, where it will then revert with the inaccurate message "amount must be positive" which doesn't reflect the underlying issue.

## Proof of Concept
https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/InvestorDistribution.sol#L145
https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/InvestorDistribution.sol#L121
https://github.com/code-423n4/2021-11-bootfinance/blob/main/vesting/contracts/Vesting.sol#L75

## Tools Used
manual review 

## Recommended Mitigation Steps
Add `require(claimable > 0)`

