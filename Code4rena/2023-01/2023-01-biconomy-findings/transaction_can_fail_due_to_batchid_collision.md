## Tags

- bug
- 2 (Med Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-08

# [Transaction can fail due to batchId collision](https://github.com/code-423n4/2023-01-biconomy-findings/issues/246) 

# Lines of code

https://github.com/code-423n4/2023-01-biconomy/blob/53c8c3823175aeb26dee5529eeefa81240a406ba/scw-contracts/contracts/smart-contract-wallet/SmartAccount.sol#L216


# Vulnerability details

## Impact

The protocol supports 2D nonces through a `batchId` mechanism. 
Due to different ways to execute transaction on the wallet there could be a collision between `batchIds` being used.

This can result in unexpected failing of transactions

## Proof of Concept

There are two main ways to execute transaction from the smart wallet 
1. Via EntryPoint - calls `execFromEntryPoint`/`execute`
2. Via `execTransaction`

`SmartAccount` has locked the `batchId` #0  to be used by the `EntryPoint`.
When an `EntryPoint` calls `validateUserOp` before execution, the hardcoded nonce of `batchId` #0 will be incremented and validated,
https://github.com/code-423n4/2023-01-biconomy/blob/53c8c3823175aeb26dee5529eeefa81240a406ba/scw-contracts/contracts/smart-contract-wallet/SmartAccount.sol#L501
```
    // @notice Nonce space is locked to 0 for AA transactions
    // userOp could have batchId as well
    function _validateAndUpdateNonce(UserOperation calldata userOp) internal override {
        require(nonces[0]++ == userOp.nonce, "account: invalid nonce");
    }
```

Calls to `execTransaction` are more immediate and are likely to be executed before a `UserOp` through `EntryPoint`.
There is no limitation in `execTransaction` to use `batchId` #0 although it should be called only by `EntryPoint`.

If there is a call to `execTransaction` with `batchId` set to `0`. It will increment the nonce and `EntryPoint` transactions will revert.
https://github.com/code-423n4/2023-01-biconomy/blob/53c8c3823175aeb26dee5529eeefa81240a406ba/scw-contracts/contracts/smart-contract-wallet/SmartAccount.sol#L216
```
    function execTransaction(
        Transaction memory _tx,
        uint256 batchId,
        FeeRefund memory refundInfo,
        bytes memory signatures
    ) public payable virtual override returns (bool success) {
-------
            nonces[batchId]++;
-------
        }
    }
```

## Tools Used

VS Code

## Recommended Mitigation Steps

Add a requirement that `batchId` is not `0` in `execTransaction`:

`require(batchId != 0, "batchId 0 is used only by EntryPoint")`