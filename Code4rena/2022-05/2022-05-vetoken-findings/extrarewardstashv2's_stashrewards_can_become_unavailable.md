## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [ExtraRewardStashV2's stashRewards can become unavailable](https://github.com/code-423n4/2022-05-vetoken-findings/issues/36) 

# Lines of code

https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/ExtraRewardStashV2.sol#L193-L203


# Vulnerability details

There is no check for the reward token amount to be transferred out in stashRewards(). As reward token list is external (controlled with `IGauge(gauge).reward_tokens`), and an arbitrary token can end up there, in the case when such token doesn't allow for zero amount transfers, the stashRewards() managed extra rewards retrieval can become unavailable.

I.e. stashRewards() can be blocked for even an extended period of time, so all other extra rewards gathering will not be possible. This cannot be controlled by the system as pool reward token list is external.

Setting the severity to medium as reward gathering is a base functionality of the system and its availability is affected.

## Proof of Concept

stashRewards() attempts to send the `amount` to rewardArbitrator() without checking:

https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/ExtraRewardStashV2.sol#L193-L203

```solidity
    if (activeCount > 1) {
        //take difference of before/after(only send new tokens)
        uint256 amount = IERC20(token).balanceOf(address(this));
        amount = amount.sub(before);

        //send to arbitrator
        address arb = IDeposit(operator).rewardArbitrator();
        if (arb != address(0)) {
            IERC20(token).safeTransfer(arb, amount);
        }
    }
```

If `IStaker(staker).withdraw()` produced no new tokens for any reason, the `amount = amount.sub(before)` above can be zero:

https://github.com/code-423n4/2022-05-vetoken/blob/2d7cd1f6780a9bcc8387dea8fecfbd758462c152/contracts/ExtraRewardStashV2.sol#L188-L189

```solidity
    uint256 before = IERC20(token).balanceOf(address(this));
    IStaker(staker).withdraw(token);
```

As reward `token` can be arbitrary, it can also be reverting on an attempt to transfer zero amounts:

https://github.com/d-xo/weird-erc20#revert-on-zero-value-transfers

If this be the case then the whole stashRewards() call will be failing until `IStaker(staker).withdraw()` manage to withdraw some `tokens` or such `token` be removed from gauge's reward token list. Both events aren’t directly controllable by the system.

## Recommended Mitigation Steps

Consider running the transfer only when amount is positive:

```solidity
-   if (activeCount > 1) {
+   if (amount > 0 && activeCount > 1) {
        //take difference of before/after(only send new tokens)
        uint256 amount = IERC20(token).balanceOf(address(this));
        amount = amount.sub(before);

        //send to arbitrator
        address arb = IDeposit(operator).rewardArbitrator();
        if (arb != address(0)) {
            IERC20(token).safeTransfer(arb, amount);
        }
    }
```

