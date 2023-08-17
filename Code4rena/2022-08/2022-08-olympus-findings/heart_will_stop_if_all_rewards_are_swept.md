## Tags

- bug
- 2 (Med Risk)
- high quality report
- sponsor confirmed

# [Heart will stop if all rewards are swept](https://github.com/code-423n4/2022-08-olympus-findings/issues/378) 

# Lines of code

https://github.com/code-423n4/2022-08-olympus/blob/b5e139d732eb4c07102f149fb9426d356af617aa/src/policies/Heart.sol#L110-L115


# Vulnerability details

Rewards for Heart `beat` are sent via `_issueReward`

https://github.com/code-423n4/2022-08-olympus/blob/b5e139d732eb4c07102f149fb9426d356af617aa/src/policies/Heart.sol#L110-L115

```solidity

    function _issueReward(address to_) internal {
        rewardToken.safeTransfer(to_, reward);
        emit RewardIssued(to_, reward);
    }

```

The function doesn't check for available tokens e.g.
`min(reward, rewardToken.balanceOf(address(this)));`


In case of calling `withdrawUnspentRewards`

https://github.com/code-423n4/2022-08-olympus/blob/b5e139d732eb4c07102f149fb9426d356af617aa/src/policies/Heart.sol#L149-L152

```solidity
    /// @inheritdoc IHeart
    function withdrawUnspentRewards(ERC20 token_) external onlyRole("heart_admin") {
        token_.safeTransfer(msg.sender, token_.balanceOf(address(this)));
    }
```

Because the function withdraws the entire amount, the heart will stop until a caller incentive is deposited again.

While a profitable searches will stop calling the Heart without an incentive, allowing the heart to beat when no rewards are available is preferable to having it self-DOS until a DAO aligned caller donates `rewardToken` or the DAO deals with the lack of tokens.

## Remediation

Add a check for available tokens
`min(reward, rewardToken.balanceOf(address(this)));`