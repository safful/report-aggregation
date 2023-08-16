## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [AuctionBurnReserveSkew - remove `for` loop from initializer](https://github.com/code-423n4/2021-11-malt-findings/issues/326) 

# Handle

ScopeLift


# Vulnerability details

## Estimated risk level
Gas Optimization

## Impact
Instantiating an array of length n is better than `push(0)`  n times and saves 20k gas in tests.

## Proof of Concept

## Tools Used


## Recommended Mitigation Steps
change the initializer

```diff
## Saves ~20,000 gas on initialize

diff --git a/src/contracts/AuctionBurnReserveSkew.sol b/src/contracts/AuctionBurnReserveSkew.sol
index 4ed6fa6..87d5959 100644
--- a/src/contracts/AuctionBurnReserveSkew.sol
+++ b/src/contracts/AuctionBurnReserveSkew.sol
@@ -51,9 +51,7 @@ contract AuctionBurnReserveSkew is Initializable, Permissions {
     auction = IAuction(_auction);
     auctionAverageLookback = _period;
 
-    for (uint i = 0; i < _period; i++) {
-      pegObservations.push(0);
-    }
+    pegObservations = new uint256[](_period);
   }
 
   function consult(uint256 excess) public view returns (uint256) {
```

