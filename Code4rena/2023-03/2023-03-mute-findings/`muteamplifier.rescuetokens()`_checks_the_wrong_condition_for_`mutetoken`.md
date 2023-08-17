## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-03

# [`MuteAmplifier.rescueTokens()` checks the wrong condition for `muteToken`](https://github.com/code-423n4/2023-03-mute-findings/issues/32) 

# Lines of code

https://github.com/code-423n4/2023-03-mute/blob/4d8b13add2907b17ac14627cfa04e0c3cc9a2bed/contracts/amplifier/MuteAmplifier.sol#L185-L191


# Vulnerability details

## Impact
There will be 2 impacts.

- The reward system would be broken as the rewards can be withdrawn before starting staking.
- Some rewards would be locked inside the contract forever as it doesn't check `totalReclaimed`

## Proof of Concept
`rescueTokens()` checks the below condition to rescue `muteToken`.

```solidity
else if (tokenToRescue == muteToken) {
    if (totalStakers > 0) {
        require(amount <= IERC20(muteToken).balanceOf(address(this)).sub(totalRewards.sub(totalClaimedRewards)),
            "MuteAmplifier::rescueTokens: that muteToken belongs to stakers"
        );
    }
}
```

But there are 2 problems.
1. Currently, it doesn't check anything when `totalStakers == 0`. So some parts(or 100%) of rewards can be withdrawn before the staking period. In this case, the reward system won't work properly due to the lack of rewards.
2. It checks the wrong condition when `totalStakers > 0` as well. As we can see [here](https://github.com/code-423n4/2023-03-mute/blob/4d8b13add2907b17ac14627cfa04e0c3cc9a2bed/contracts/amplifier/MuteAmplifier.sol#L241), some remaining rewards are tracked using `totalReclaimed` and transferred to treasury directly. So we should consider this amount as well.

## Tools Used
Manual Review

## Recommended Mitigation Steps
It should be modified like the below.

```solidity
else if (tokenToRescue == muteToken) {
    if (totalStakers > 0) { //should check totalReclaimed as well
        require(amount <= IERC20(muteToken).balanceOf(address(this)).sub(totalRewards.sub(totalClaimedRewards).sub(totalReclaimed)),
            "MuteAmplifier::rescueTokens: that muteToken belongs to stakers"
        );
    }
    else if(block.timestamp <= endTime) { //no stakers but staking is still active, should maintain totalRewards
        require(amount <= IERC20(muteToken).balanceOf(address(this)).sub(totalRewards),
            "MuteAmplifier::rescueTokens: that muteToken belongs to stakers"
        );
    }
}
```