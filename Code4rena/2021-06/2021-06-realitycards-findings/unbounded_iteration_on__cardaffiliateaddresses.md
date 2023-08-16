## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed
- resolved

# [Unbounded iteration on _cardAffiliateAddresses](https://github.com/code-423n4/2021-06-realitycards-findings/issues/154) 

# Handle

cmichel


# Vulnerability details

## Vulnerability Details

The `Factory.createMarket` iterates over all `_cardAffiliateAddresses`.

## Impact

The transactions can fail if the arrays get too big and the transaction would consume more gas than the block limit.
This will then result in a denial of service for the desired functionality and break core functionality.


## Recommended Mitigation Steps

Perform a `_cardAffiliateAddresses.length == 0 || _cardAffiliateAddresses.length == tokenUris.length` check in `createMarket` instead of silently skipping card affiliate cuts in `Market.initialize`.
This would restrict the `_cardAffiliateAddresses` length to the `nftMintingLimit` as well.

