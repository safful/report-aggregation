## Tags

- bug
- 3 (High Risk)
- primary issue
- selected for report
- sponsor confirmed
- H-08

# [Attacker can steal the amount collected so far in the GroupBuy for NFT purchase.](https://github.com/code-423n4/2022-12-tessera-findings/issues/47) 

# Lines of code

https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/modules/GroupBuy.sol#L204


# Vulnerability details

## Description

purchase() in GroupBuy.sol executes the purchase call for the group. After safety checks, the NFT is bought with \_market's execute() function. Supposedly it deploys a vault which owns the NFT. The code makes sure the vault is the new owner of the NFT and exits.
```
// Executes purchase order transaction through market buyer contract and deploys new vault
address vault = IMarketBuyer(_market).execute{value: _price}(_purchaseOrder);
// Checks if NFT contract supports ERC165 and interface ID of ERC721 tokens
if (ERC165Checker.supportsInterface(_nftContract, _INTERFACE_ID_ERC721)) {
    // Verifes vault is owner of ERC-721 token
    if (IERC721(_nftContract).ownerOf(_tokenId) != vault) revert UnsuccessfulPurchase();
} else {
    // Verifies vault is owner of CryptoPunk token
    if (ICryptoPunk(_nftContract).punkIndexToAddress(_tokenId) != vault)
        revert UnsuccessfulPurchase();
}

// Stores mapping value of poolId to newly deployed vault
poolToVault[_poolId] = vault;
// Sets pool state to successful
poolInfo[_poolId].success = true;
// Emits event for purchasing NFT at given price
emit Purchase(_poolId, vault, _nftContract, _tokenId, _price);
```

The issue is that \_market user-supplied variable is not validated at all. Attacker can pass their malicious contract, which uses the passed funds to buy the NFT and store it in attacker's wallet. It will return the NFT-holding wallet so the checks will pass. As a result, attacker has the NFT while they could have contributed nothing to the GroupBuy. Attacker can also just steal the supplied ETH and return the current address which holds the NFT.

## Impact

Attacker can steal the amount collected so far in the GroupBuy for NFT purchase.

## Proof of Concept

1. Group assembles and raises funds to buy NFT X
2. Attacker calls purchase() and supplies their malicious contract in \_market, as described.
3. Attacker receives raised funds totalling  `minReservePrices[_poolId] * filledQuantities[_poolId]`, as checked in line 182.

## Tools Used

Manual audit

## Recommended Mitigation Steps

\_market should be whitelisted, or supplied in createPool stage and able to be scrutinized.