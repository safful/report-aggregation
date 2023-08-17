## Tags

- bug
- 3 (High Risk)
- disagree with severity
- resolved
- sponsor confirmed

# [User can forfeit other user rewards](https://github.com/code-423n4/2022-05-aura-findings/issues/50) 

# Lines of code

https://github.com/code-423n4/2022-05-aura/blob/main/contracts/ExtraRewardsDistributor.sol#L127


# Vulnerability details

## Impact
User can forfeit other user rewards by giving a higher _startIndex in getReward function

## Proof of Concept
1. Assume User B has not received any reward yet so that his userClaims[_token][User B]=0
2. User A calls getReward function with _account as User B and _startIndex as 5
3. This eventually calls _allClaimableRewards at ExtraRewardsDistributor.sol#L213 which computes epochIndex =5>0?5:0 = 5
4. Assuming tokenEpochs is 10 and latestEpoch is 8, so reward will computed from epoch 5 till epoch index 7 and _allClaimableRewards will return index as 7
5. _getReward will simply update userClaims[_token][User B] with 7 
6. This is incorrect because as per contract User B has received reward from epoch 0-7 even though he only received reward for epoch 5-7


## Recommended Mitigation Steps
Do not allow users to call getReward function for other users

