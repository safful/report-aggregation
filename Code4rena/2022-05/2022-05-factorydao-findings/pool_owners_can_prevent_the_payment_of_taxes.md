## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [Pool owners can prevent the payment of taxes](https://github.com/code-423n4/2022-05-factorydao-findings/issues/124) 

# Lines of code

https://github.com/code-423n4/2022-05-factorydao/blob/db415804c06143d8af6880bc4cda7222e5463c0e/contracts/PermissionlessBasicPoolFactory.sol#L258-L272


# Vulnerability details

## Impact
Pool owners can prevent taxes from being paid without impacting any other functionality

## Proof of Concept
By adding a custom reward token that always reverts for transfers to `globalBenericiary`, the owner can prevent taxes from being paid:
```solidity
File: contracts/PermissionlessBasicPoolFactory.sol   #1

258       /// @notice Withdraw taxes from pool
259       /// @dev Anyone may call this, it just moves the taxes from this contract to the globalBeneficiary
260       /// @param poolId which pool are we talking about?
261       function withdrawTaxes(uint poolId) external {
262           Pool storage pool = pools[poolId];
263           require(pool.id == poolId, 'Uninitialized pool');
264   
265           bool success = true;
266           for (uint i = 0; i < pool.rewardTokens.length; i++) {
267               uint tax = taxes[poolId][i];
268               taxes[poolId][i] = 0;
269               success = success && IERC20(pool.rewardTokens[i]).transfer(globalBeneficiary, tax);
270           }
271           require(success, 'Token transfer failed');
272       }
```
https://github.com/code-423n4/2022-05-factorydao/blob/db415804c06143d8af6880bc4cda7222e5463c0e/contracts/PermissionlessBasicPoolFactory.sol#L258-L272

While the sponsor mentions that malicious tokens make the pool malicious, this particular issue has a simple fix outlined below in the mitigation section

## Tools Used
Code inspection

## Recommended Mitigation Steps
Force taxes to be paid during `withdraw()`


