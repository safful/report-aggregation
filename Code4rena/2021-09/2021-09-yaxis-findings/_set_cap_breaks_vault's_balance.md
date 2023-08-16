## Tags

- bug
- 3 (High Risk)
- sponsor acknowledged
- sponsor confirmed

# [ set cap breaks vault's Balance](https://github.com/code-423n4/2021-09-yaxis-findings/issues/1) 

# Handle

jonah1005


# Vulnerability details

## Impact
In controller.sol's function `setCap`, the contract wrongly handles `_vaultDetails[_vault].balance`. While the balance should be decreased by the difference of strategies balance, it subtracts the remaining balance of the strategy.
[Controller.sol#L262-L278](https://github.com/code-423n4/2021-09-yaxis/blob/main/contracts/v3/controllers/Controller.sol#L262-L278)
 `_vaultDetails[_vault].balance = _vaultDetails[_vault].balance.sub(_balance);`

This would result in `vaultDetails[_vault].balance` being far smaller than the strategy's value. A user would trigger the assertion at [Contreller.sol#475](https://github.com/code-423n4/2021-09-yaxis/blob/main/contracts/v3/controllers/Controller.sol#L475) and the fund would be locked in the strategy.

Though `setCap` is a permission function that only the operator can call, it's likely to be called and the fund would be locked in the contract. I consider this a high severity issue.

## Proof of Concept
We can trigger the issue by setting the cap 1 wei smaller than the strategy's balance.


```python
strategy_balance = strategy.functions.balanceOf().call()
controller.functions.setCap(vault.address, strategy.address, strategy_balance - 1, dai.address).transact()

## this would be reverted
vault.functions.withdrawAll(dai.address).transact()
```


[Controller.sol#L262-L278](https://github.com/code-423n4/2021-09-yaxis/blob/main/contracts/v3/controllers/Controller.sol#L262-L278)

## Tools Used
Hardhat

## Recommended Mitigation Steps

I believe the dev would spot the issue in the test if `_vaultDetails[_vault].balance` is a public variable.

One possile fix is to subtract the difference of the balance.
```solidity
uint previousBalance = IStrategy(_strategy).balanceOf();
_vaultDetails[_vault].balance.sub(previousBalance.sub(_amount));
```


