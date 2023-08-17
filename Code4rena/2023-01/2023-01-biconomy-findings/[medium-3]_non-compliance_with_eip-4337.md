## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-02

# [[Medium-3] Non-compliance with EIP-4337](https://github.com/code-423n4/2023-01-biconomy-findings/issues/498) 

# Lines of code

https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/BaseSmartAccount.sol#L60-L68
https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/aa-4337/core/EntryPoint.sol#L319-L329
https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/BaseSmartAccount.sol#L60-L68


# Vulnerability details

## Impact

Some parts of the codebase are not compliant with the EIP-4337 from the [EIP-4337 specifications](https://eips.ethereum.org/EIPS/eip-4337#specification), at multiple degrees of severity.

## Proof of Concept

### Sender existence

```text
Create the account if it does not yet exist, using the initcode provided in the UserOperation. If the account does not exist, and the initcode is empty, or does not deploy a contract at the “sender” address, the call must fail.
```

If we take a look at the [`_createSenderIfNeeded()`]() function, we can see that it's not properly implemented:
```solidity
function _createSenderIfNeeded(uint256 opIndex, UserOpInfo memory opInfo, bytes calldata initCode) internal {
	if (initCode.length != 0) {
		address sender = opInfo.mUserOp.sender;
    	if (sender.code.length != 0) revert FailedOp(opIndex, address(0), "AA10 sender already constructed");
      	address sender1 = senderCreator.createSender{gas: opInfo.mUserOp.verificationGasLimit}(initCode);
        if (sender1 == address(0)) revert FailedOp(opIndex, address(0), "AA13 initCode failed or OOG");
        if (sender1 != sender) revert FailedOp(opIndex, address(0), "AA14 initCode must return sender");
        if (sender1.code.length == 0) revert FailedOp(opIndex, address(0), "AA15 initCode must create sender");
        address factory = address(bytes20(initCode[0:20]));
      	emit AccountDeployed(opInfo.userOpHash, sender, factory, opInfo.mUserOp.paymaster);
	}
}
```

The statement in the EIP implies that if the account does not exist, the initcode **must** be used.
In this case, it first check if the initcode exists, but this condition should be checked later.

This could be rewritten to:

```solidity
function _createSenderIfNeeded(uint256 opIndex, UserOpInfo memory opInfo, bytes calldata initCode) internal {
	address sender = opInfo.mUserOp.sender;
	if (sender.code.length == 0) {
		require(initCode.length != 0, "empty initcode");
		address sender1 = senderCreator.createSender{gas: opInfo.mUserOp.verificationGasLimit}(initCode);
        if (sender1 == address(0)) revert FailedOp(opIndex, address(0), "AA13 initCode failed or OOG");
        if (sender1 != sender) revert FailedOp(opIndex, address(0), "AA14 initCode must return sender");
        if (sender1.code.length == 0) revert FailedOp(opIndex, address(0), "AA15 initCode must create sender");
        address factory = address(bytes20(initCode[0:20]));
      	emit AccountDeployed(opInfo.userOpHash, sender, factory, opInfo.mUserOp.paymaster);
	}
}
```

### Account

The third specification of the [`validateUserOp()`](https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/BaseSmartAccount.sol#L60-L68) is the following:

```text
If the account does not support signature aggregation, it MUST validate the signature is a valid signature of the userOpHash, and SHOULD return SIG_VALIDATION_FAILED (and not revert) on signature mismatch. Any other error should revert.
```

This is currently not the case, as the case when the account does not support signature aggregation is not supported right now in the code. The `validateUserOp()` [reverts everytime](https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/aa-4337/core/EntryPoint.sol#L319-L329) if the recovered signature does not match.

Additionally, the `validateUserOp()` should return a time range, as per the EIP specifications:

```text
The return value is packed of sigFailure, validUntil and validAfter timestamps.
		- sigFailure is 1 byte value of “1” the signature check failed (should not revert on signature failure, to support estimate)
		- validUntil is 8-byte timestamp value, or zero for “infinite”. The UserOp is valid only up to this time.
		- validAfter is 8-byte timestamp. The UserOp is valid only after this time.
```

This isn't the case. It just returns a signature deadline validity, which would probably be here the `validUntil` value.

### Aggregator

This part deals with the aggregator interfacing:

```text
validateUserOp() (inherited from IAccount interface) MUST verify the aggregator parameter is valid and the same as getAggregator

...

The account should also support aggregator-specific getter (e.g. getAggregationInfo()). This method should export the account’s public-key to the aggregator, and possibly more info (note that it is not called directly by the entryPoint)

...

If an account uses an aggregator (returns it with getAggregator()), then its address is returned by simulateValidation() reverting with ValidationResultWithAggregator instead of ValidationResult
```

This aggregator address validation is not [done](https://github.com/code-423n4/2023-01-biconomy/blob/main/scw-contracts/contracts/smart-contract-wallet/BaseSmartAccount.sol#L60-L68).

## Tools Used

Manual inspection

## Recommended Mitigation Steps

Refactor the code that is not compliant with the EIP