## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Expired transfers will lock user funds on the sending chain](https://github.com/code-423n4/2021-07-connext-findings/issues/47) 

# Handle

0xRajeev


# Vulnerability details

## Impact

The cancelling relayer is being paid in receivingAssetId on the sendingChain instead of in sendingAssetID. If the user relies on a relayer to cancel transactions and that receivingAssetId asset does not exist on the sending chain (assuming only sendingAssetID on the sending chain and receivingAssetId on the receiving chain are assured to be valid and present) then the cancel transaction from the relayer will always revert and user’s funds will remain locked on the sending chain.

Impact: Expired transfers can never be cancelled and user funds will be locked forever if user relies on a relayer.

## Proof of Concept

https://github.com/code-423n4/2021-07-connext/blob/8e1a7ea396d508ed2ebeba4d1898a748255a48d2/contracts/TransactionManager.sol#L510-L517


## Tools Used

Manual Analysis

## Recommended Mitigation Steps

Change receivingAssetId to sendingAssetId in transferAsset() on L514.

