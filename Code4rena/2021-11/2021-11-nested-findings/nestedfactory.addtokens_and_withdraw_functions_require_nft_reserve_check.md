## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [NestedFactory.addTokens and withdraw functions require NFT reserve check](https://github.com/code-423n4/2021-11-nested-findings/issues/199) 

# Handle

hyh


# Vulnerability details

## Impact

NFT token operations will fail if wrong reserve is used.

## Proof of Concept

```NestedFactory``` ```reserve``` is used in ```addtokens``` and ```withdraw``` function for a given NFT, but the NFT to reserve contract correspondence isn't checked.

addtokens:
https://github.com/code-423n4/2021-11-nested/blob/main/contracts/NestedFactory.sol#L119

withdraw:
https://github.com/code-423n4/2021-11-nested/blob/main/contracts/NestedFactory.sol#L241

## Recommended Mitigation Steps

Add the ```require(nestedRecords.getAssetReserve(_nftId) == address(reserve), "...")``` check in the beginning of the functions.

