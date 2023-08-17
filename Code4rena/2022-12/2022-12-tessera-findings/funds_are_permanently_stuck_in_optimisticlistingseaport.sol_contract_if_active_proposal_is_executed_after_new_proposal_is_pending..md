## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- H-06

# [Funds are permanently stuck in OptimisticListingSeaport.sol contract if active proposal is executed after new proposal is pending.](https://github.com/code-423n4/2022-12-tessera-findings/issues/43) 

# Lines of code

https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/modules/OptimisticListingSeaport.sol#L428


# Vulnerability details

## Description

\_constructOrder is called in propose(), OptimisticListingSeaport.sol. It fills the order params stored in proposedListings[_vault]. 
```
{
    orderParams.offerer = _vault;
    orderParams.startTime = block.timestamp;
    // order doesn't expire in human time scales and needs explicit cancellations
    orderParams.endTime = type(uint256).max;
    orderParams.zone = zone;
    // 0: no partial fills, anyone can execute
    orderParams.orderType = OrderType.FULL_OPEN;
    orderParams.conduitKey = conduitKey;
    // 1 Consideration for the listing itself + 1 consideration for the fees
    orderParams.totalOriginalConsiderationItems = 3;
}
```

Importantly, it updates the order hash associated with the vault:
`vaultOrderHash[_vault] = _getOrderHash(orderParams, counter);`

There is only one other use of `vaultOrderHash`, in \_verifySale(). 
```
function _verifySale(address _vault) internal view returns (bool status) {
    (bool isValidated, bool isCancelled, uint256 totalFilled, uint256 totalSize) = ISeaport(
        seaport
    ).getOrderStatus(vaultOrderHash[_vault]);
    if (isValidated && !isCancelled && totalFilled > 0 && totalFilled == totalSize) {
        status = true;
    }
}
```
This function gets order information from the order hash, and makes sure the order is completely fulfilled.

After NFT sell has completed, cash() is used to distribute income ETH:
```
function cash(address _vault, bytes32[] calldata _burnProof) external {
    // Reverts if vault is not registered
    (address token, uint256 id) = _verifyVault(_vault);
    // Reverts if active listing has not been settled
    Listing storage activeListing = activeListings[_vault];
    // Reverts if listing has not been sold
	  // -------------- _verifySale MUST BE TRUE ---------
    if (!_verifySale(_vault)) {
        revert NotSold();
    } else if (activeListing.collateral != 0) {
        uint256 collateral = activeListing.collateral;
        activeListing.collateral = 0;
        // Sets collateral amount to pending balances for withdrawal
        pendingBalances[_vault][activeListing.proposer] = collateral;
    }
```

As long as sale is not complete, cash() can't be called as highlighted. The issue is that `vaultOrderHash[_vault]` is not protected during the lifetime of an active proposal. If another proposal is proposed and then the sell using active proposal takes place, cash() will keep reverting. Funds are stuck in listing contract.

We can try to be clever and call propose() again with the same parameters to create an identical orderID, which will make `vaultOrderHash[_vault]` fine again and allow cash() to go through. But order params contain block.timestamp which will certainly be different which will make the hash different.


## Impact

Funds are permanently stuck in OptimisticListingSeaport.sol contract if active proposal is executed after new proposal is pending.

## Proof of Concept

1. User A calls propose(), setting proposedListing. vaultOrderHash=X
2. PROPOSAL_PERIOD passes , list is called promoting the listing to activeListing.
3. Another user, malicious or innocent, proposes another proposal. vaultOrderHash=Y
4. Sell goes down due to OpenSea validation confirmed on activeListing.
5. \_verifySale will never return true because we can never got vaultOrderHash to be X
6. cash() is bricked. Money is stuck in contract.

## Tools Used

Manual audit

## Recommended Mitigation Steps

Keep the order hash in the Listing structure rather than a single one per vault.