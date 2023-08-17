## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Users can claim more fees than expected if governance migrates current rewardToken again by fault.](https://github.com/code-423n4/2022-05-backd-findings/issues/86) 

# Lines of code

https://github.com/code-423n4/2022-05-backd/tree/main/protocol/contracts/BkdLocker.sol#L70-L75
https://github.com/code-423n4/2022-05-backd/tree/main/protocol/contracts/BkdLocker.sol#L302-L322


# Vulnerability details

## Impact
Users can claim more fees than expected if governance migrates current rewardToken again by fault.

## Proof of Concept
In the migrate() function, there is no requirement newRewardToken != rewardToken.
If this function is called with the same "rewardToken" parameter, "_replacedRewardTokens" will contain the current "rewardToken" also.
Then when the user claims fees, "userShares" will be added two times for the same token at L302-L305, L314-L317.
It's because "curRewardTokenData.userFeeIntegrals[user]" is updated at L332 after the "userShares" calculation for past rewardTokens.
So the user can get paid more fees than he should.

## Tools Used
Solidity Visual Developer of VSCode

## Recommended Mitigation Steps
You need to add this require() at L71.

require(newRewardToken != rewardToken, Error.SAME_AS_CURRENT);

