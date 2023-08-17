## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- edited-by-warden
- H-02

# [Anyone can steal CryptoPunk during the deposit flow to WPunkGateway](https://github.com/code-423n4/2022-11-paraspace-findings/issues/137) 

# Lines of code

https://github.com/code-423n4/2022-11-paraspace/blob/c6820a279c64a299a783955749fdc977de8f0449/paraspace-core/contracts/ui/WPunkGateway.sol#L77-L95
https://github.com/code-423n4/2022-11-paraspace/blob/c6820a279c64a299a783955749fdc977de8f0449/paraspace-core/contracts/ui/WPunkGateway.sol#L129-L155
https://github.com/code-423n4/2022-11-paraspace/blob/c6820a279c64a299a783955749fdc977de8f0449/paraspace-core/contracts/ui/WPunkGateway.sol#L167-L193


# Vulnerability details

## Impact

All CryptoPunk deposits can be stolen

## Proof of Concept

CryptoPunks were created before the ERC721 standard. A consequence of this is that they do not possess the `transferFrom` method. To approximate this a user can `offerPunkForSaleToAddress` for a price of 0 to effectively approve the contract to `transferFrom`. 


[WPunkGateway.sol#L77-L95](https://github.com/code-423n4/2022-11-paraspace/blob/c6820a279c64a299a783955749fdc977de8f0449/paraspace-core/contracts/ui/WPunkGateway.sol#L77-L95)

    function supplyPunk(
        DataTypes.ERC721SupplyParams[] calldata punkIndexes,
        address onBehalfOf,
        uint16 referralCode
    ) external nonReentrant {
        for (uint256 i = 0; i < punkIndexes.length; i++) {
            Punk.buyPunk(punkIndexes[i].tokenId);
            Punk.transferPunk(proxy, punkIndexes[i].tokenId);
            // gatewayProxy is the sender of this function, not the original gateway
            WPunk.mint(punkIndexes[i].tokenId);
        }
        Pool.supplyERC721(
            address(WPunk),
            punkIndexes,
            onBehalfOf,
            referralCode
        );
    }

The current implementation of `WPunkGateway#supplyPunk` allows anyone to execute and determine where the nTokens are minted to. To complete the flow supply flow a user would need to `offerPunkForSaleToAddress` for a price of 0 to `WPunkGateway`. After they have done this, anyone can call the function to deposit the punk and mint the nTokens to themselves, effectively stealing it.

Example:
`User A` owns `tokenID` of 1. They want to deposit it so they call `offerPunkForSaleToAddress` with an amount of 0, effectively approving the `WPunkGateway` to transfer their CryptoPunk. `User B` monitors the transactions and immediately calls `supplyPunk` with themselves as `onBehalfOf`. This completes the transfer of the CryptoPunk and deposits it into the pool but mints the `nTokens` to `User B`, allowing them to effectively steal the CryptoPunk 

The same fundamental issue exists with `acceptBidWithCredit` and `batchAcceptBidWithCredit`

## Tools Used

Manual Review

## Recommended Mitigation Steps

Query the punkIndexToAddress to find the owner and only allow owner to deposit:

        for (uint256 i = 0; i < punkIndexes.length; i++) {
    +       address owner = Punk.punkIndexToAddress(punkIndexes[i].tokenId);
    +       require(owner == msg.sender);

            Punk.buyPunk(punkIndexes[i].tokenId);
            Punk.transferPunk(proxy, punkIndexes[i].tokenId);
            // gatewayProxy is the sender of this function, not the original gateway
            WPunk.mint(punkIndexes[i].tokenId);
        }