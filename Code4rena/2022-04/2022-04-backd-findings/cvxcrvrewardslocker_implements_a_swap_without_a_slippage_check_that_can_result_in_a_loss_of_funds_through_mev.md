## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- reviewed

# [CvxCrvRewardsLocker implements a swap without a slippage check that can result in a loss of funds through MEV](https://github.com/code-423n4/2022-04-backd-findings/issues/161) 

# Lines of code

https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/CvxCrvRewardsLocker.sol#L247-L252


# Vulnerability details

## Impact
The CvxCrvRewardsLocker contract swaps tokens through the CRV cvxCRV pool. But, it doesn't use any slippage checks. The swap is at risk of being frontrun / sandwiched which will result in a loss of funds.

Since MEV is very prominent I think the chance of that happening is pretty high.

## Proof of Concept
Here's the swap: https://github.com/code-423n4/2022-04-backd/blob/c856714a50437cb33240a5964b63687c9876275b/backd/contracts/CvxCrvRewardsLocker.sol#L247-L252

## Tools Used
none

## Recommended Mitigation Steps
Use a proper value for `minOut` instead of `0`.

