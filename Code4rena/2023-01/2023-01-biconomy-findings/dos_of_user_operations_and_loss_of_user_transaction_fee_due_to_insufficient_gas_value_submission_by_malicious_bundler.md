## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- downgraded by judge
- primary issue
- selected for report
- sponsor confirmed
- M-05

# [DoS of user operations and loss of user transaction fee due to insufficient gas value submission by malicious bundler](https://github.com/code-423n4/2023-01-biconomy-findings/issues/303) 

# Lines of code

https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/aa-4337/core/EntryPoint.sol#L68-L86
https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/aa-4337/core/EntryPoint.sol#L168-L190


# Vulnerability details

## Impact
An attacker (e.g. a malicious bundler) could submit a bundle of high gas usage user operations with insufficient gas value, causing the bundle to fail even when the users calculated the gas limits correctly. This will result in a DoS for the user and the user/paymaster still have to pay for the execution, potentially draining their funds. This attack is possible as user operations are included by bundlers from the UserOperation mempool into the Ethereum block (see post on ERC-4337 https://medium.com/infinitism/erc-4337-account-abstraction-without-ethereum-protocol-changes-d75c9d94dc4a).

Reference for this issue: https://github.com/eth-infinitism/account-abstraction/commit/4fef857019dc2efbc415ac9fc549b222b07131ef

## Proof of Concept
In innerHandleOp(), a call was made to handle the operation with the specified mUserOp.callGasLimit. However, a malicious bundler could call the innerHandleOp() via handleOps() with a gas value that is insufficient for the transactions, resulting in the call to fail. 

The remaining gas amount (e.g. gasLeft()) at this point was not verified to ensure that it is more than enough to fulfill the specified mUserOp.callGasLimit for the user operation. Even though the operation failed, the user/payment will still pay for the transactions due to the post operation logic. 

https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/aa-4337/core/EntryPoint.sol#L176

	(bool success,bytes memory result) = address(mUserOp.sender).call{gas : mUserOp.callGasLimit}(callData);

## Tools Used

## Recommended Mitigation Steps
Update the Account Abstraction implementation to the latest version. This will update the innerHandleOp() to verify that remaining gas is more than sufficient to cover the specified mUserOp.callGasLimit and mUserOp.verificationGasLimit. 

Reference: https://github.com/eth-infinitism/account-abstraction/commit/4fef857019dc2efbc415ac9fc549b222b07131ef