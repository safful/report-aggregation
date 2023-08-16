## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Adding unchecked directive can save gas](https://github.com/code-423n4/2021-10-covalent-findings/issues/52) 

# Handle

WatchPug


# Vulnerability details

For the arithmetic operations that will never over/underflow, using the unchecked directive (Solidity v0.8 has default overflow/underflow checks) can save some gas from the unnecessary internal over/underflow checks.

For example:

https://github.com/code-423n4/2021-10-covalent/blob/ded3aeb2476da553e8bb1fe43358b73334434737/contracts/DelegatedStaking.sol#L111-L111

```solidity
rewardsLocked -= amount;
```

https://github.com/code-423n4/2021-10-covalent/blob/ded3aeb2476da553e8bb1fe43358b73334434737/contracts/DelegatedStaking.sol#L201-L202

```solidity
totalGlobalShares += globalSharesToAdd;
v.globalShares += globalSharesToAdd;
```


https://github.com/code-423n4/2021-10-covalent/blob/ded3aeb2476da553e8bb1fe43358b73334434737/contracts/DelegatedStaking.sol#L261-L261

```solidity
us.amount -= amount;
```

https://github.com/code-423n4/2021-10-covalent/blob/ded3aeb2476da553e8bb1fe43358b73334434737/contracts/DelegatedStaking.sol#L350-L350

```solidity
validatorsN +=1;
```

https://github.com/code-423n4/2021-10-covalent/blob/ded3aeb2476da553e8bb1fe43358b73334434737/contracts/DelegatedStaking.sol#L415-L415

```solidity
us.amount -= amount;
```

https://github.com/code-423n4/2021-10-covalent/blob/ded3aeb2476da553e8bb1fe43358b73334434737/contracts/DelegatedStaking.sol#L428-L428

```solidity
us.amount -= amount;
```

https://github.com/code-423n4/2021-10-covalent/blob/ded3aeb2476da553e8bb1fe43358b73334434737/contracts/DelegatedStaking.sol#L316-L319

```solidity
uint128 commissionLeftOver = amount < v.commissionAvailableToRedeem ? v.commissionAvailableToRedeem - amount : 0;
// if there is more, redeem  it from regular rewards
if (commissionLeftOver == 0){
    uint128 validatorSharesRemove = tokensToShares(amount - v.commissionAvailableToRedeem, v.exchangeRate);
```

