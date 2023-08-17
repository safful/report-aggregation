## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [Pool owners can prevent withdrawals of specific receipts](https://github.com/code-423n4/2022-05-factorydao-findings/issues/125) 

# Lines of code

https://github.com/code-423n4/2022-05-factorydao/blob/db415804c06143d8af6880bc4cda7222e5463c0e/contracts/PermissionlessBasicPoolFactory.sol#L230-L234


# Vulnerability details

## Impact
Pool owners can prevent withdrawals of specific receipts without impacting any other functionality

## Proof of Concept
Reciepts are non-transferrable, so a malicious owner can monitor the blockchain for receipt creations, and inspect which account holds the receiptId. Next, by changing settings in a custom reward token that reverts for specific addresses, the owner can prevent that specific receipt owner from withdrawing:
```solidity
File: contracts/PermissionlessBasicPoolFactory.sol   #1

230               success = success && IERC20(pool.rewardTokens[i]).transfer(receipt.owner, transferAmount);
231           }
232   
233           success = success && IERC20(pool.depositToken).transfer(receipt.owner, receipt.amountDepositedWei);
234           require(success, 'Token transfer failed');
```
https://github.com/code-423n4/2022-05-factorydao/blob/db415804c06143d8af6880bc4cda7222e5463c0e/contracts/PermissionlessBasicPoolFactory.sol#L230-L234

While the sponsor mentions that malicious tokens make the pool malicious, this particular issue has a straight forward fix outlined below in the mitigation section

## Tools Used
Code inspection

## Recommended Mitigation Steps
Rather than reverting the whole withdrawal if only one transfer fails, return a boolean of whether all withdrawals were successful, and allow `withdraw()` to be called multiple times, keeping track of what has been transferred and what hasn't


