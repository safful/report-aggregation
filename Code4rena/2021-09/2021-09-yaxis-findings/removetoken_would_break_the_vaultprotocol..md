## Tags

- bug
- 3 (High Risk)
- sponsor acknowledged
- sponsor confirmed

# [removeToken would break the vault/protocol.](https://github.com/code-423n4/2021-09-yaxis-findings/issues/4) 

# Handle

jonah1005


# Vulnerability details

## removeToken would break the vault.


## Impact
There's no safety check in Manager.sol's removeToken. [Manager.sol#L454-L487](https://github.com/code-423n4/2021-09-yaxis/blob/main/contracts/v3/Manager.sol#L454-L487)
1. The token would be locked in the original vault. Given the current design, the vault would keep a ratio of total amount to save the gas. Once the token is removed at manager contract, these token would lost.
2. Controller's balanceOf would no longer reflects the real value. [Controller.sol#L488-L495](https://github.com/code-423n4/2021-09-yaxis/blob/main/contracts/v3/controllers/Controller.sol#L488-L495) While `_vaultDetails[msg.sender].balance;` remains the same, user can nolonger withdraw those amount.
3. Share price in the vault would decrease drastically. The share price is calculated as `totalValue / totalSupply` [Vault.sol#L217](https://github.com/code-423n4/2021-09-yaxis/blob/main/contracts/v3/Vault.sol#L217). While the `totalSupply` of the share remains the same, the total balance has drastically decreased.

Calling removeToken way would almost break the whole protocol if the vault has already started. I consider this is a high-risk issue.


## Proof of Concept

We can see how the vault would be affected with below web3.py script.
```python
print(vault.functions.balanceOfThis().call())
print(vault.functions.totalSupply().call())
manager.functions.removeToken(vault.address, dai.address).transact()
print(vault.functions.balanceOfThis().call())
print(vault.functions.totalSupply().call())
```

output
```
100000000000000000000000
100000000000000000000000
0
100000000000000000000000
```
## Tools Used
Hardhat

## Recommended Mitigation Steps
Remove tokens from a vault would be a really critical job. I recommend the team cover all possible cases and check all components' states (all vault/ strategy/ controller's state) in the test.

 Some steps that I try to come up with that is required to remove TokenA from a vault.
 1. Withdraw all tokenA from all strategies (and handle it correctly in the controller).
 2. Withdraw all tokenA from the vault.
 3. Convert all tokenA that's collected in the previous step into tokenB.
 4. Transfer tokenB to the vault and compensate the transaction fee/slippage cost to the vault.
 

