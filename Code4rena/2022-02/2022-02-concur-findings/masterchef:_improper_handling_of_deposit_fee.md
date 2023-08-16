## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Masterchef: Improper handling of deposit fee](https://github.com/code-423n4/2022-02-concur-findings/issues/138) 

# Lines of code

https://github.com/code-423n4/2022-02-concur/blob/main/contracts/MasterChef.sol#L170-L172


# Vulnerability details

## Impact

If a pool’s deposit fee is non-zero, it is subtracted from the amount to be credited to the user.

```jsx
if (pool.depositFeeBP > 0) {
  uint depositFee = _amount.mul(pool.depositFeeBP).div(_perMille);
  user.amount = SafeCast.toUint128(user.amount + _amount - depositFee);
}
```

However, the deposit fee is not credited to anyone, leading to permanent lockups of deposit fees in the relevant depositor contracts (StakingRewards and ConvexStakingWrapper for now).

## Proof of Concept

### Example 1: ConvexStakingWrapper

Assume the following

- The [curve cDai / cUSDC / cUSDT LP token](https://etherscan.io/address/0x9fC689CCaDa600B6DF723D9E47D84d76664a1F23) corresponds to `pid = 1` in the convex booster contract.
- Pool is added in Masterchef with `depositFeeBP = 100 (10%)`.
1. Alice deposits 1000 LP tokens via the ConvexStakingWrapper contract. A deposit fee of 100 LP tokens is charged. Note that the `deposits` mapping of the ConvexStakingWrapper contract credits 1000 LP tokens to her.
2. However, Alice will only be able to withdraw 900 LP tokens. The 100 LP tokens is not credited to any party, and is therefore locked up permanently (essentially becomes protocol-owned liquidity). While she is able to do `requestWithdraw()` for 1000 LP tokens, attempts to execute `withdraw()` with amount = 1000 will revert because she is only credited 900 LP tokens in the Masterchef contract.

### Example 2: StakingRewards

- CRV pool is added in Masterchef with `depositFeeBP = 100 (10%)`.
1. Alice deposits 1000 CRV into the StakingRewards contract. A deposit fee of 100 CRV is charged.
2. Alice is only able to withdraw 900 CRV tokens, while the 100 CRV is not credited to any party, and is therefore locked up permanently.

These examples are non-exhaustive as more depositors can be added / removed from the Masterchef contract.

## Recommended Mitigation Steps

I recommend shifting the deposit fee logic out of the masterchef contract into the depositor contracts themselves, as additional logic would have to be added in the masterchef to update the fee recipient’s state (rewardDebt, send pending concur rewards, update amount), which further complicates matters. As the fee recipient is likely to be the treasury, it is also not desirable for it to accrue concur rewards.

```jsx
if (pool.depositFeeBP > 0) {
  uint depositFee = _amount.mul(pool.depositFeeBP).div(_perMille);
  user.amount = SafeCast.toUint128(user.amount + _amount - depositFee);
  UserInfo storage feeRecipient = userInfo[_pid][feeRecipient];
  // TODO: update and send feeRecipient pending concur rewards
  feeRecipient.amount = SafeCast.toUint128(feeRecipient.amount + depositFee);
  // TODO: update fee recipient's rewardDebt
}
```

