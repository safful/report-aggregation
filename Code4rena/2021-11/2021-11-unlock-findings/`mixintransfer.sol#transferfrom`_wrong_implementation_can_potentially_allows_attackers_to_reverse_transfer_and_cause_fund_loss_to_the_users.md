## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [`MixinTransfer.sol#transferFrom` Wrong implementation can potentially allows attackers to reverse transfer and cause fund loss to the users](https://github.com/code-423n4/2021-11-unlock-findings/issues/182) 

# Handle

WatchPug


# Vulnerability details

https://github.com/code-423n4/2021-11-unlock/blob/ec41eada1dd116bcccc5603ce342257584bec783/smart-contracts/contracts/mixins/MixinTransfer.sol#L131-L152

```solidity
    if (toKey.tokenId == 0) {
      toKey.tokenId = _tokenId;
      _recordOwner(_recipient, _tokenId);
      // Clear any previous approvals
      _clearApproval(_tokenId);
    }

    if (previousExpiration <= block.timestamp) {
      // The recipient did not have a key, or had a key but it expired. The new expiration is the sender's key expiration
      // An expired key is no longer a valid key, so the new tokenID is the sender's tokenID
      toKey.expirationTimestamp = fromKey.expirationTimestamp;
      toKey.tokenId = _tokenId;

      // Reset the key Manager to the key owner
      _setKeyManagerOf(_tokenId, address(0));

      _recordOwner(_recipient, _tokenId);
    } else {
      // The recipient has a non expired key. We just add them the corresponding remaining time
      // SafeSub is not required since the if confirms `previousExpiration - block.timestamp` cannot underflow
      toKey.expirationTimestamp = fromKey.expirationTimestamp + previousExpiration - block.timestamp;
    }
```

Based on the context, L131-136 seems to be the logic of handling the case of the recipient with no key, and L138-148 is handing the case of the recipient's key expired.

However, in L131-136, the key manager is not being reset.

This allows attackers to keep the role of key manager after the transfer, and transfer the key back or to another recipient.

### PoC

Given:

- Alice owns a key that is valid until 1 year later.

1. Alice calls `setKeyManagerOf()`, making herself the keyManager;
2. Alice calls `transferFrom()`, transferring the key to Bob; Bob might have paid a certain amount of money to Alice upon receive of the key;
3. Alice calls `transferFrom()` again, transferring the key back from Bob.

### Recommendation

Consider resetting the key manager regardless of the status of the recipient's key.

