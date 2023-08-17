## Tags

- bug
- 3 (High Risk)
- primary issue
- selected for report
- sponsor confirmed
- H-07

# [User loses collateral converted to pendingBalance when cash() or list() is called](https://github.com/code-423n4/2022-12-tessera-findings/issues/44) 

# Lines of code

https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/modules/OptimisticListingSeaport.sol#L295
https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/modules/OptimisticListingSeaport.sol#L232


# Vulnerability details

## Description

In OptimisticListingOpensea, there are several functions which update pendingBalances of a proposer:
1. list()
2. cash()
3. propose()

Unfortunately, in list() and cash() the = operator is used instead of += when writing the new pendingBalances. For example:
```
function cash(address _vault, bytes32[] calldata _burnProof) external {
    // Reverts if vault is not registered
    (address token, uint256 id) = _verifyVault(_vault);
    // Reverts if active listing has not been settled
    Listing storage activeListing = activeListings[_vault];
    // Reverts if listing has not been sold
    if (!_verifySale(_vault)) {
        revert NotSold();
    } else if (activeListing.collateral != 0) {
        uint256 collateral = activeListing.collateral;
        activeListing.collateral = 0;
        // Sets collateral amount to pending balances for withdrawal
        pendingBalances[_vault][activeListing.proposer] = collateral;
    }
	...
```

pendingBalances is not guaranteed to be zero. There could be funds from previous proposals which are not yet collected. Propose updates pendingBalance correctly:
```
// Sets collateral amount to pending balances for withdrawal
pendingBalances[_vault][proposedListing.proposer] += proposedListing.collateral;
```

So, when propose is followed by another propose(), the pendingBalance is updated correctly, but in cash and list we don't account for pre-existing balance. This issue would manifest even after the fix suggested in the issue "User can send a proposal and instantly take back their collateral" because reject functions would increment the pendingBalance and then it would be overriden.

## Impact

User loses collateral converted to pendingBalance when cash() or list() is called

## Proof of Concept

1. User calls propose() and gets pendingBalance = x
2. User calls propose() with an improved proposal and gets pendingBalance = 1.5x
3. proposal is successfull and the listing purchased the NFT
4. cash() is called to convert the Raes to ETH amount from the sell. pendingBalance is overridden by the current "collateral"  value. pendingBalance = 0.5x
5. User loses x collateral value which is stuck in the contract

## Tools Used

Manual audit

## Recommended Mitigation Steps

Change the = operator to += in list() and cash().