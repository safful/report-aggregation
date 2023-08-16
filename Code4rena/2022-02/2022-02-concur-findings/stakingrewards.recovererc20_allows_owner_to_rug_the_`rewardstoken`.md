## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [StakingRewards.recoverERC20 allows owner to rug the `rewardsToken`](https://github.com/code-423n4/2022-02-concur-findings/issues/69) 

# Lines of code

https://github.com/code-423n4/2022-02-concur/blob/main/contracts/StakingRewards.sol#L166


# Vulnerability details

## Impact
`StakingRewards.recoverERC20` rightfully checks against the `stakingToken` being sweeped away.
However there's no check against the `rewardsToken` which over time will sit in this contract.

This is the case of an admin privilege, which allows the owner to sweep the rewards tokens, perhaps as a way to rug depositors

## Proof of Concept
calling `StakingRewards.recoverERC20(rewardsToken, rewardsToken.balanceOf(this))` enables the `owner` to sweep the token

## Recommended Mitigation Steps
Add an additional check
```
        require(
            tokenAddress != address(rewardsToken),
            "Cannot withdraw the rewards token"
        );
```

