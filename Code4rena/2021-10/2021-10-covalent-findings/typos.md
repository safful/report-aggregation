## Tags

- bug
- 0 (Non-critical)
- resolved
- sponsor confirmed

# [Typos](https://github.com/code-423n4/2021-10-covalent-findings/issues/42) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-10-covalent/blob/ded3aeb2476da553e8bb1fe43358b73334434737/contracts/DelegatedStaking.sol#L57

```solidity=57
event TransferedUnstake(uint128 indexed oldValidatorId, uint128 indexed newValidatorId, address indexed delegator, uint128 amount, uint128 unstakingId);
```

https://github.com/code-423n4/2021-10-covalent/blob/ded3aeb2476da553e8bb1fe43358b73334434737/contracts/DelegatedStaking.sol#L419

```solidity=419
emit TransferedUnstake(oldValidatorId, newValidatorId, msg.sender, amount, unstakingId);
```

`Transfered` should be `Transferred`.

https://github.com/code-423n4/2021-10-covalent/blob/ded3aeb2476da553e8bb1fe43358b73334434737/contracts/DelegatedStaking.sol#L303-L304

```solidity
// if validator calls redeem rewards, first tokens paid from comissions will be redeemed and then regular rewards
    function redeemRewards( uint128 validatorId, address beneficiary, uint128 amount) public {
```

`comissions` should be `commissions`.

https://github.com/code-423n4/2021-10-covalent/blob/ded3aeb2476da553e8bb1fe43358b73334434737/contracts/DelegatedStaking.sol#L392-L393

```solidity
// only owner can change comission rate
    function setValidatorCommissionRate(uint128 amount, uint128 validatorId) public onlyOwner {
```

`comission` should be `commission`.

https://github.com/code-423n4/2021-10-covalent/blob/ded3aeb2476da553e8bb1fe43358b73334434737/contracts/DelegatedStaking.sol#L374-L375

```solidity
// calclate how much rewards would be distribited with the old emission rate
uint128 futureRewards = allocatedTokensPerEpoch * epochs;
```

`calculate` should be `calculate`. `distribited` should be `distributed`.

