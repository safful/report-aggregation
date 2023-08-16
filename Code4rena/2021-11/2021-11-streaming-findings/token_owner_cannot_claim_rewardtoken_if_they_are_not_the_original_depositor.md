## Tags

- bug
- 1 (Low Risk)
- disagree with severity
- sponsor confirmed

# [Token owner cannot claim rewardToken if they are not the original depositor](https://github.com/code-423n4/2021-11-streaming-findings/issues/204) 

# Handle

gzeon


# Vulnerability details

## Impact
The comment in https://github.com/code-423n4/2021-11-streaming/blob/56d81204a00fc949d29ddd277169690318b36821/Streaming/src/Locke.sol#L553 stated that: 
> Allows a receipt token holder (or original depositor in case of a sale) to claim their rewardTokens
but the reward is only tracked to the original depositor in both case, see
https://github.com/code-423n4/2021-11-streaming/blob/56d81204a00fc949d29ddd277169690318b36821/Streaming/src/Locke.sol#L558
```
        TokenStream storage ts = tokensNotYetStreamed[msg.sender];
```
Transferring the LockeERC20 token does not transfer the TokenStream state.


