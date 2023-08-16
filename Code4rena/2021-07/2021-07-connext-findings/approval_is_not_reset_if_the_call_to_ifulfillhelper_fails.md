## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Approval is not reset if the call to IFulfillHelper fails](https://github.com/code-423n4/2021-07-connext-findings/issues/31) 

# Handle

pauliax


# Vulnerability details

## Impact
Function fulfill first approves the callTo to transfer an amount of toSend tokens and tries to call IFulfillHelper but if the call fails it transfers these assets directly. However, in such case the approval is not reset so a malicous callTo can pull these tokens later:
    // First, approve the funds to the helper if needed
        if (!LibAsset.isEther(txData.receivingAssetId) && toSend > 0) {
          require(LibERC20.approve(txData.receivingAssetId, txData.callTo, toSend), "fulfill: APPROVAL_FAILED");
        }

        // Next, call `addFunds` on the helper. Helpers should internally
        // track funds to make sure no one user is able to take all funds
        // for tx
        if (toSend > 0) {
          try
            IFulfillHelper(txData.callTo).addFunds{ value: LibAsset.isEther(txData.receivingAssetId) ? toSend : 0}(
              txData.user,
              txData.transactionId,
              txData.receivingAssetId,
              toSend
            )
          {} catch {
            // Regardless of error within the callData execution, send funds
            // to the predetermined fallback address
            require(
              LibAsset.transferAsset(txData.receivingAssetId, payable(txData.receivingAddress), toSend),
              "fulfill: TRANSFER_FAILED"
            );
          }
        }

## Recommended Mitigation Steps
Approve should be placed inside the try/catch block or approval needs to be reset if the call fails.

