## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Cache the return value from rewardPerToken()](https://github.com/code-423n4/2021-11-streaming-findings/issues/44) 

# Handle

GiveMeTestEther


# Vulnerability details

## Impact
Save gas by caching the return value from rewardPerToken() in a local variable and use the local variable L220 and L222. This saves us two storage reads (1 cold = 800 gas, and 1 warm= 100 gas). It is way cheaper to read from a local variable (push/pop operations 2-3 gas each + cheap others)

Note: same for the claimReward() function on L555

## Proof of Concept
https://github.com/code-423n4/2021-11-streaming/blob/56d81204a00fc949d29ddd277169690318b36821/Streaming/src/Locke.sol#L203

https://github.com/code-423n4/2021-11-streaming/blob/56d81204a00fc949d29ddd277169690318b36821/Streaming/src/Locke.sol#L555
## Tools Used

## Recommended Mitigation Steps
- cache in a  local variable: uint256 _rewardPerToken = rewardPerToken();
- write the value to the storage variable: cumulativeRewardPerToken =  _rewardPerToken;
- replace the occurrences of cumulativeRewardPerToken on L220/222 with _rewardPerToken 

