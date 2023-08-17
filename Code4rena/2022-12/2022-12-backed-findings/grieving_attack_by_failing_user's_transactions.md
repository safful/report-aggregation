## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- edited-by-warden
- M-03

# [Grieving attack by failing user's transactions](https://github.com/code-423n4/2022-12-backed-findings/issues/92) 

# Lines of code

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L486


# Vulnerability details

## Impact

An attacker can apply grieving attack by preventing users from interacting with some of the protocol functions. In other words whenever a user is going to reduce his debt, or buy and reduce his debt in one tx, it can be failed by the attacker.

## Proof of Concept
In the following scenario, I am explaining how it is possible to fail user's transaction to reduce their debt fully. Failing other transaction (buy and reduce the debt in one tx) can be done similarly.

 - Suppose Alice (an honest user) has debt of 1000 `PaprToken` and she intends to repay her debt fully: 
 - So, she calls the function `reduceDebt` with the following parameters:
   - `account`: Alice's address
   - `asset`: The NFT which was used as collateral
   - `amount`: 1000 * 10**18 (decimal of `PaprToken` is 18).
```
function reduceDebt(address account, ERC721 asset, uint256 amount) external override {
        _reduceDebt({account: account, asset: asset, burnFrom: msg.sender, amount: amount});
    }
```
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L148
 - Bob (a malicious user who owns a small amount of `PaprToken`) notices Alice's transaction in the Mempool. So, Bob applies front-run attack and calls the function `reduceDebt` with the following parameters:
   - `account`: Alice's address
   - `asset`: The NFT which was used as collateral
   - `amount`: 1
 - By doing so, Bob repays only **1** `PaprToken` on behalf of Alice, so Alice's debt becomes `1000 * 10**18 - 1`.
```
function _reduceDebt(address account, ERC721 asset, address burnFrom, uint256 amount) internal {
        _reduceDebtWithoutBurn(account, asset, amount);
        PaprToken(address(papr)).burn(burnFrom, amount);
    }
```
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L481
```
function _reduceDebtWithoutBurn(address account, ERC721 asset, uint256 amount) internal {
        _vaultInfo[account][asset].debt = uint200(_vaultInfo[account][asset].debt - amount);
        emit ReduceDebt(account, asset, amount);
    }
```
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L486
 - Then, when Alice's transaction is going to be executed, it fails because of `Underflow Error`. Since Alice's debt is `1000 * 10**18 - 1` while Alice's transaction was going to repay `1000 * 10**18`.
```
_vaultInfo[account][asset].debt = uint200(_vaultInfo[account][asset].debt - amount);
```
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L487
 - Bob only pays a very small value of 1 `PaprToken` (consider that the decimal is 18) to apply this grieving attack.
 - Bob can repeat this attack for Alice, if Alice is going to call this function again with correct parameter.

***In summary, Bob could prevent the user from paying her debt fully by just repaying a very small amount of the user's debt in advance and as a result causing underflow error. Bob can apply this attack for all other users who are going to repay their debt fully. Please note that if a user is going to repay her debt partially, the attack can be expensive and not financially reasonable, but in case of full repayment of debt, it is very cheap to apply this grieving attack.***

***This attack can be applied on the transactions that are going to interact with the function `_reduceDebt`. The transactions interacting with this specific function are:***
 - `buyAndReduceDebt(...)`
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L229
 - `reduceDebt(...)`
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L149

***It means that the attacker can prevent users from calling the functions above.***

## Tools Used

## Recommended Mitigation Steps
The following condition should be added to the function `_reduceDebtWithoutBurn`:
```
function _reduceDebtWithoutBurn(address account, ERC721 asset, uint256 amount) internal {
        if(amount > _vaultInfo[account][asset].debt){
            amount = _vaultInfo[account][asset].debt;
        }
        _vaultInfo[account][asset].debt = uint200(_vaultInfo[account][asset].debt - amount);
        emit ReduceDebt(account, asset, amount);
    }
```
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L486