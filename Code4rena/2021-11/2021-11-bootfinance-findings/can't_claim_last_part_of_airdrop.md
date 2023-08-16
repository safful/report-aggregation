## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [Can't claim last part of airdrop](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/130) 

# Handle

gpersoon


# Vulnerability details

## Impact
Suppose you are eligible for the last part of your airdrop (or your entire airdrop if you haven't claimed anything yet).
Then you call the function claim() of AirdropDistribution.sol, which has the following statement:
"assert(airdrop[msg.sender].amount - claimable != 0);"
This statement will prevent you from claiming your airdrop because it will stop execution.

Note: with the function claimExact() it is possible to claim the last part.

## Proof of Concept
// https://github.com/code-423n4/2021-11-bootfinance/blob/7c457b2b5ba6b2c887dafdf7428fd577e405d652/vesting/contracts/AirdropDistribution.sol#L522-L536

function claim() external nonReentrant {
       ..
        assert(airdrop[msg.sender].amount - claimable != 0);
        airdrop[msg.sender].amount -= claimable;

## Tools Used

## Recommended Mitigation Steps
Remove the assert statement.
Also add the following to validate() , to prevent claiming the airdrop again:
      require(validated[msg.sender]== 0, "Already validated.");

