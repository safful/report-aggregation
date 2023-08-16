## Tags

- bug
- 1 (Low Risk)
- resolved
- sponsor confirmed

# [Only passing in one depositedBalance in _checkpointAndClaim()](https://github.com/code-423n4/2022-01-yield-findings/issues/88) 

# Handle

GeekyLumberjack


# Vulnerability details

`uint256[2] memory depositedBalance;` is defined at the beginning of [_checkpointAndClaim](https://github.com/code-423n4/2022-01-yield/blob/main/contracts/ConvexStakingWrapper.sol#L279-L291) only one `depositedBalance` slot is being filed and then the entire array gets passed into `_calcRewardIntegral()` and `_calcCvxIntegral()` along with an array of two `_accounts`. Having only one of the `depositedBalance` and two `_accounts` may cause loss in rewards for the second account. This function is currently only used in `GetReward()` which is passing in a zero address as the second address.


