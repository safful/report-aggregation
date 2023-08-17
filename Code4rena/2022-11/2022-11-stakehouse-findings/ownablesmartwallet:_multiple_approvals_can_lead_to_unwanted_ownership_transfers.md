## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-07

# [OwnableSmartWallet: Multiple approvals can lead to unwanted ownership transfers](https://github.com/code-423n4/2022-11-stakehouse-findings/issues/99) 

# Lines of code

https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/smart-wallet/OwnableSmartWallet.sol#L94
https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/smart-wallet/OwnableSmartWallet.sol#L105-L106


# Vulnerability details

## Impact
The `OwnableSmartWallet` contract employs a mechanism for the owner to approve addresses that can then claim ownership ([https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/smart-wallet/OwnableSmartWallet.sol#L94](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/smart-wallet/OwnableSmartWallet.sol#L94)) of the contract.  

The source code has a comment included which states that "Approval is revoked, in order to avoid unintended transfer allowance if this wallet ever returns to the previous owner" ([https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/smart-wallet/OwnableSmartWallet.sol#L105-L106](https://github.com/code-423n4/2022-11-stakehouse/blob/4b6828e9c807f2f7c569e6d721ca1289f7cf7112/contracts/smart-wallet/OwnableSmartWallet.sol#L105-L106)).  

This means that when ownership is transferred from User A to User B, the approvals that User A has given should be revoked.  

The existing code does not however revoke all approvals that User A has given. It only revokes one approval.  

This can lead to unwanted transfers of ownership.  

## Proof of Concept
1. User A approves User B and User C to claim ownership
2. User B claims ownership first
3. Only User A's approval for User B is revoked, not however User A's approval for User C
4. User B transfers ownerhsip back to User A
5. Now User C can claim ownership even though this time User A has not approved User C

## Tools Used
VSCode

## Recommended Mitigation Steps
You should invalidate all approvals User A has given when another User becomes the owner of the OwnableSmartWallet.  

Unfortunately you cannot use a statement like `delete _isTransferApproved[owner()]`.  

So you would need an array that keeps track of approvals as pointed out in this StackExchange question: [https://ethereum.stackexchange.com/questions/15553/how-to-delete-a-mapping](https://ethereum.stackexchange.com/questions/15553/how-to-delete-a-mapping)  
