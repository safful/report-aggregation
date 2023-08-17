## Tags

- bug
- 3 (High Risk)
- primary issue
- selected for report
- sponsor confirmed
- H-01

# [Any user being the first to claim rewards from GiantMevAndFeesPool can unexepectedly collect them all](https://github.com/code-423n4/2022-11-stakehouse-findings/issues/32) 

# Lines of code

https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/SyndicateRewardsProcessor.sol#L85
https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/SyndicateRewardsProcessor.sol#L61
https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/liquid-staking/GiantMevAndFeesPool.sol#L203


# Vulnerability details

## Impact
Any user being the first to claim rewards from GiantMevAndFeesPool, can get all the previously generated rewards whatever the amount and even if he did not participate to generate those rewards...

## Proof of Concept

https://gist.github.com/clems4ever/c9fe06ce454ff6c4124f4bd29d3598de

Copy paste it in the test suite and run it.

## Tools Used

forge test

## Recommended Mitigation Steps

Rework the way `accumulatedETHPerLPShare` and `claimed` is used. There are multiple bugs due to the interaction between those variables as you will see in my other reports.