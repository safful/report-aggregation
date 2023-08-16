## Tags

- bug
- 0 (Non-critical)
- resolved
- sponsor confirmed

# [Misleading parameter name](https://github.com/code-423n4/2021-10-covalent-findings/issues/60) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-10-covalent/blob/ded3aeb2476da553e8bb1fe43358b73334434737/contracts/DelegatedStaking.sol#L423-L433

```solidity
function transferUnstakedOut(uint128 amount, uint128 validatorId, uint128 stakingId) public {
    Unstaking storage us = validators[validatorId].unstakings[msg.sender][stakingId];
    require( uint128(block.number) > us.coolDownEnd, "Cooldown period has not ended" );
    require(us.amount >= amount, "Amount is too high");
    transferFromContract(msg.sender, amount);
    us.amount -= amount;
    // set cool down end to 0 to release gas if new unstaking amount is 0
    if (us.amount == 0)
        us.coolDownEnd = 0;
    emit UnstakeRedeemed(validatorId, msg.sender, amount);
}
```

The last parameter of `transferUnstakedOut()` is named `stakingId`, while other functions is using `unstakingId`. This is inconsistent and can be misleading.

### Recommendation

Change from `stakingId` to `unstakingId`.

