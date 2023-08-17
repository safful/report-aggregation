## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Unbounded loop in `withdraw()` may cause rewards to be locked in the contract](https://github.com/code-423n4/2022-05-factorydao-findings/issues/122) 

# Lines of code

https://github.com/code-423n4/2022-05-factorydao/blob/db415804c06143d8af6880bc4cda7222e5463c0e/contracts/PermissionlessBasicPoolFactory.sol#L224-L231


# Vulnerability details

## Impact
The `withdraw()` has an unbounded loop with external calls. If the gas costs of functions change between when deposits are made and when rewards are withdrawn, or if the gas cost of the deposit (`transferFrom()`) is less than the gas cost of the withdrawal (`transfer()`), then the `withdraw()` function may revert due to exceeding the block size gas limit.

## Proof of Concept
`transfer()` is an external call, and `rewards.length` has no maximum size:
```solidity
File: contracts/PermissionlessBasicPoolFactory.sol   #1

224           for (uint i = 0; i < rewards.length; i++) {
225               pool.rewardsWeiClaimed[i] += rewards[i];
226               pool.rewardFunding[i] -= rewards[i];
227               uint tax = (pool.taxPerCapita * rewards[i]) / 1000;
228               uint transferAmount = rewards[i] - tax;
229               taxes[poolId][i] += tax;
230               success = success && IERC20(pool.rewardTokens[i]).transfer(receipt.owner, transferAmount);
231           }
```
https://github.com/code-423n4/2022-05-factorydao/blob/db415804c06143d8af6880bc4cda7222e5463c0e/contracts/PermissionlessBasicPoolFactory.sol#L224-L231

## Tools Used
Code inspection

## Recommended Mitigation Steps
Allow the specification of an offset and length to the `withdraw()` function, so that withdrawals can be broken up into smaller batches if required


