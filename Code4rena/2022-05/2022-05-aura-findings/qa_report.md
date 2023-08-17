## Tags

- bug
- QA (Quality Assurance)
- resolved
- sponsor confirmed

# [QA Report](https://github.com/code-423n4/2022-05-aura-findings/issues/5) 

The `amount should be greater than 0` is not checked by the `_addReward` function.

## Permalink:
https://github.com/code-423n4/2022-05-aura/blob/c332f8c23e12b3bb678f018682c7609df9c77ed9/contracts/ExtraRewardsDistributor.sol#L87

## Recommendation:
Can Add Require Statement to check _amount is greater than 0.