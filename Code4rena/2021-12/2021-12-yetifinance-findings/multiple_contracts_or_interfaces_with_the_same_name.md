## Tags

- bug
- 0 (Non-critical)
- disagree with severity
- sponsor confirmed

# [Multiple contracts or interfaces with the same name](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/180) 

# Handle

heiho1


# Vulnerability details

## Impact
Got an error attempting Slither analysis due to Pool2Unipool and Unipool declaring the same contract named LPTokenWrapper.  It is confusing and error prone to have such similarly named contracts and there is no clear benefit to re-using the name.

## Proof of Concept

https://github.com/code-423n4/2021-12-yetifinance/blob/5f5bf61209b722ba568623d8446111b1ea5cb61c/packages/contracts/contracts/LPRewards/Pool2Unipool.sol#L23

https://github.com/code-423n4/2021-12-yetifinance/blob/5f5bf61209b722ba568623d8446111b1ea5cb61c/packages/contracts/contracts/LPRewards/Pool2Unipool.sol#L23

## Tools Used

Slither

## Recommended Mitigation Steps

Rename LPTokenWrapper in Pool2Unipool to 'Pool2LPTokenWrapper' and correct any related imports.

