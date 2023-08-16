## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [getRewardsAmount doesn't check epochs haven't been claimed](https://github.com/code-423n4/2021-12-pooltogether-findings/issues/50) 

# Handle

harleythedog


# Vulnerability details

## Impact
In ITwabRewards.sol, it is claimed that `getRewardsAmount` should account for epochs that have already been claimed, and not include these epochs in the total amount (indeed, there is a line that says `@dev Will be 0 if user has already claimed rewards for the epoch.`)

However, no such check is done in the implementation of `getRewardsAmount`. This means that users will be shown rewardAmounts that are higher than they should be, and users will be confused when they are transferred fewer tokens than they are told they will. This would cause confusion, and people may begin to mistrust the contract since they think they are being transferred fewer tokens than they are owed.

## Proof of Concept
See the implementation of `getRewardsAmount` here: https://github.com/pooltogether/v4-periphery/blob/b520faea26bcf60371012f6cb246aa149abd3c7d/contracts/TwabRewards.sol#L209

Notice that there are no checks that the epochs have not already been claimed. Compare this to `claimRewards` which *does* check for epochs that have already been claimed with the following require statement:
```
require(!_isClaimedEpoch(_userClaimedEpochs, _epochId), "TwabRewards/rewards-already-claimed");
```
A similar check should be added `getRewardsAmount` so that previously claimed epochs are not included in the sum.

## Tools Used
Inspection

## Recommended Mitigation Steps
Add a similar check for previously claimed epochs as described above.

