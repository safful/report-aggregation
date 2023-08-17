## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Inconsistency in view functions can lead to users believing they’re due for more BKD rewards](https://github.com/code-423n4/2022-05-backd-findings/issues/150) 

# Lines of code

https://github.com/code-423n4/2022-05-backd/blob/2a5664d35cde5b036074edef3c1369b984d10010/protocol/contracts/tokenomics/AmmConvexGauge.sol#L107-L111
https://github.com/code-423n4/2022-05-backd/blob/2a5664d35cde5b036074edef3c1369b984d10010/protocol/contracts/tokenomics/AmmConvexGauge.sol#L129-L134


# Vulnerability details

## Impact
The view functions used for a user to check their claimable rewards vary in their implementation. This can cause users to believe they are due X amount but will receive Y.

## Proof of Concept
If the `inflationRecipient` is set, then `poolStakedIntegral` will be incremented in `claimableRewards()` but not in any other function like `allClaimableRewards()` or `poolCheckpoint()`.

If a user calls `claimableRewards()` after the `inflationRepient` has been set, `claimableRewards()` will return a larger value than `allClaimableRewards()` or the amount actually returned by `claimRewards()`.

## Tools Used
Manual review

## Recommended Mitigation Steps
To make the logic consistent, `claimableRewards()` needs `if (inflationRecipient == address(0))` added to it.

