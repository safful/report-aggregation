## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [a single user can become owner of multiple token ids](https://github.com/code-423n4/2021-11-unlock-findings/issues/120) 

# Handle

GiveMeTestEther


# Vulnerability details

## Impact / POC

A single user can become the owner of multiple token ids and break the assumption of the comment [https://github.com/code-423n4/2021-11-unlock/blob/ec41eada1dd116bcccc5603ce342257584bec783/smart-contracts/contracts/mixins/MixinKeys.sol#L181](https://github.com/code-423n4/2021-11-unlock/blob/ec41eada1dd116bcccc5603ce342257584bec783/smart-contracts/contracts/mixins/MixinKeys.sol#L181) of the function numberOfOwners() that it returns "total number of unique owners" 

If a key manager/approved transfers a key with transferFrom() [https://github.com/code-423n4/2021-11-unlock/blob/ec41eada1dd116bcccc5603ce342257584bec783/smart-contracts/contracts/mixins/MixinTransfer.sol#L109](https://github.com/code-423n4/2021-11-unlock/blob/ec41eada1dd116bcccc5603ce342257584bec783/smart-contracts/contracts/mixins/MixinTransfer.sol#L109) to a recipient that also owns a valid key then we don't go into the "if block" L131 and also not into the "if block" L138 (this is important s.t. no key owner change happens) and go into the "else block" L148 (not really important). 

We end with: fromKey.expirationTimestamp = block.timestamp; and fromKey.tokenId = 0;

If the key owner or someone else buys for this key owner again a "key" [https://github.com/code-423n4/2021-11-unlock/blob/ec41eada1dd116bcccc5603ce342257584bec783/smart-contracts/contracts/mixins/MixinPurchase.sol#L51](https://github.com/code-423n4/2021-11-unlock/blob/ec41eada1dd116bcccc5603ce342257584bec783/smart-contracts/contracts/mixins/MixinPurchase.sol#L51) satisfies the condition idTo ==0 (bcs of tokenId = 0) and in _assignNewTokenId(toKey); the key gets a new token id and the owner gets also registered as the new owner of the new token id in _recordOwner(_recipient, idTo);

The "old" key got overwritten but we are now the owner of two token ids.

This breaks the comment of numberOfOwners() that it returns "total number of unique owners" but for this the key owner that owns now two token ids, we executed "_recordOwner" twice and therefore added the same address twice to the owner array  [https://github.com/code-423n4/2021-11-unlock/blob/ec41eada1dd116bcccc5603ce342257584bec783/smart-contracts/contracts/mixins/MixinKeys.sol#L327](https://github.com/code-423n4/2021-11-unlock/blob/ec41eada1dd116bcccc5603ce342257584bec783/smart-contracts/contracts/mixins/MixinKeys.sol#L327) 

## Tools Used

Manual Analysis

## Recommended Mitigation Steps

- need to also implement the removal of ownership of a tokenId when it is set 0 zero to be congruent with the state of the key, and also adapt the other logic depending on it

