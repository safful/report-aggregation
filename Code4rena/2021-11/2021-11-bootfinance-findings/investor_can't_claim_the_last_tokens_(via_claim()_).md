## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [Investor can't claim the last tokens (via claim() )](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/131) 

# Handle

gpersoon


# Vulnerability details

## Impact
Suppose you are an investor and want to claim the last part of your claimable tokens (or your entire set of claimable tokens if you haven't claimed anything yet).
Then you call the function claim() of InvestorDistribution.sol, which has the following statement:
"require(investors[msg.sender].amount - claimable != 0);"
This statement will prevent you from claiming your tokens because it will stop execution.

Note: with the function claimExact() it is possible to claim the last part.

## Proof of Concept
// https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/vesting/contracts/InvestorDistribution.sol#L113-L128

function claim() external nonReentrant {
...
        require(investors[msg.sender].amount - claimable != 0);
        investors[msg.sender].amount -= claimable;

## Tools Used

## Recommended Mitigation Steps
Remove the require statement.


