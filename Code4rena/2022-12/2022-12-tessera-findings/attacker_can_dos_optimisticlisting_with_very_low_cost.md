## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- H-05

# [Attacker can DOS OptimisticListing with very low cost](https://github.com/code-423n4/2022-12-tessera-findings/issues/25) 

# Lines of code

https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/modules/OptimisticListingSeaport.sol#L112-L116


# Vulnerability details

## Impact

The only check on a new proposal is that it is priced lower than the existing proposal. It does not constrain on the `_collateral` supplied (except it will revert in _verifyBalance if set to 0). Anyone can block normal proposal creation by creating a proposal with lower price but _collateral == 1. When a high total supply is used, the price of each Rae is negligible and enable an attacker to DOS the protocol.

This violated the `prevent a user from holding a vault hostage and never letting the piece be reasonably bought` requirement.

## Proof of Concept

For any proposal, an attacker can deny it with _collateral = 1 and _price = price - 1

If he do not want the NFT to be sold, he can reject the proposal himself, reseting the contract state.

https://github.com/code-423n4/2022-12-tessera/blob/f37a11407da2af844bbfe868e1422e3665a5f8e4/src/seaport/modules/OptimisticListingSeaport.sol#L112-L116

```solidity
        // Reverts if price per token is not lower than both the proposed and active listings
        if (
            _pricePerToken >= proposedListing.pricePerToken ||
            _pricePerToken >= activeListings[_vault].pricePerToken
        ) revert NotLower();
```

Add this test to OptimisticListingSeaport.t.sol:

```
    function testProposeRevertLowerTotalValue() public {
        uint256 _collateral = 100;
        uint256 _price = 100;
        // setup
        testPropose(_collateral, _price);
        lowerPrice = pricePerToken - 1;
        // execute
        vm.expectRevert();
        _propose(eve, vault, 1, lowerPrice, offer);
        // expect
        _assertListing(eve, 1, lowerPrice, block.timestamp);
        _assertTokenBalance(eve, token, tokenId, eveTokenBalance - 1);
    }
```
[FAIL. Reason: Call did not revert as expected]


## Tools Used

Foundry

## Recommended Mitigation Steps

Require the total value of the new collateral be greater than the previous.
This however still allow a Rae holder with sufficiently large holding to block proposal by creating a new proposal and immediately reject it himself. 