## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Reward token not correctly recovered](https://github.com/code-423n4/2021-11-streaming-findings/issues/214) 

# Handle

cmichel


# Vulnerability details

The `Streaming` contract allows recovering the reward token by calling `recoverTokens(rewardToken, recipient)`.

However, the excess amount is computed incorrectly as `ERC20(token).balanceOf(address(this)) - (rewardTokenAmount + rewardTokenFeeAmount)`:

```solidity
function recoverTokens(address token, address recipient) public lock {
    if (token == rewardToken) {
        require(block.timestamp > endRewardLock, "time");

        // check what isnt claimable by depositors and governance
        // @audit-issue rewardTokenAmount increased on fundStream, but never decreased! this excess underflows
        uint256 excess = ERC20(token).balanceOf(address(this)) - (rewardTokenAmount + rewardTokenFeeAmount);
        ERC20(token).safeTransfer(recipient, excess);

        emit RecoveredTokens(token, recipient, excess);
        return;
    }
    // ...
```

Note that `rewardTokenAmount` only ever _increases_ (when calling `fundStream`) but it never decreases when claiming the rewards through `claimReward`.
However, `claimReward` transfers out the reward token.

Therefore, the `rewardTokenAmount` never tracks the contract's reward balance and the excess cannot be computed that way.

#### POC
Assume no reward fees for simplicity and only a single user staking.

- Someone funds `1000` reward tokens through `fundStream(1000)`. Then `rewardTokenAmount = 1000` 
- The stream and reward lock period is over, i.e. `block.timestamp > endRewardLock`
- The user claims their full reward and receives `1000` reward tokens by calling `claimReward()`. The reward contract balance is now `0` but `rewardTokenAmount = 1000`
- Some fool sends 1000 reward tokens to the contract by accident. These cannot be recovered as the `excess = balance - rewardTokenAmount = 0`

## Impact
Reward token recovery does not work.

## Recommended Mitigation Steps
The claimed rewards need to be tracked as well, just like the claimed deposits are tracked.
I think you can even decrease `rewardTokenAmount` in `claimReward` because at this point `rewardTokenAmount` is not used to update the `cumulativeRewardPerToken` anymore.

