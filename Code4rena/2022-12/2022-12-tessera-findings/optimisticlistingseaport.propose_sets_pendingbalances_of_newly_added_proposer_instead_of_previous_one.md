## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- H-04

# [OptimisticListingSeaport.propose sets pendingBalances of newly added proposer instead of previous one](https://github.com/code-423n4/2022-12-tessera-findings/issues/12) 

# Lines of code

https://github.com/code-423n4/2022-12-tessera/blob/1e408ebc1c4fdcc72678ea7f21a94d38855ccc0b/src/seaport/modules/OptimisticListingSeaport.sol#L126


# Vulnerability details

## Impact
In `OptimisticListingSeaport.propose`, `pendingBalances` is set to the collateral. The purpose of this is that the proposer of a previous proposal can withdraw his collateral afterwards. However, this is done on the storage variable `proposedListing` after the new listing is already set:
```solidity
_setListing(proposedListing, msg.sender, _collateral, _pricePerToken, block.timestamp);

// Sets collateral amount to pending balances for withdrawal
pendingBalances[_vault][proposedListing.proposer] += proposedListing.collateral;
```
Because of that, it will actually set `pendingBalances` of the new proposer. Therefore, the old proposer loses his collateral and the new one can make proposals for free.

## Proof Of Concept
```diff
--- a/test/seaport/OptimisticListingSeaport.t.sol
+++ b/test/seaport/OptimisticListingSeaport.t.sol
@@ -379,8 +379,11 @@ contract OptimisticListingSeaportTest is SeaportTestUtil {
     /// ===== LIST =====
     /// ================
     function testList(uint256 _collateral, uint256 _price) public {
         // setup
         testPropose(_collateral, _price);
+        assertEq(optimistic.pendingBalances(vault, bob), 0);
         _increaseTime(PROPOSAL_PERIOD);
         _collateral = _boundCollateral(_collateral, bobTokenBalance);
         _price = _boundPrice(_price);
```
This test fails and `optimistic.pendingBalances(vault, bob)` is equal to `_collateral`.

## Recommended Mitigation Steps
Run `pendingBalances[_vault][proposedListing.proposer] += proposedListing.collateral;` before the `_setListing` call, in which case the above PoC no longer works.