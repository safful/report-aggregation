## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- fixed-in-upstream-repo

# [[Optimization] Cache length in the loop](https://github.com/code-423n4/2021-08-floatcapital-findings/issues/97) 

# Handle

hrkrshnn


# Vulnerability details


## Cache length in the for loop

``` solidity
modified   contracts/contracts/LongShort.sol
@@ -1059,7 +1059,8 @@ contract LongShort is ILongShort, Initializable {
   /// @param user The address of the user for whom to execute the function for.
   /// @param marketIndexes An array of int32s which each uniquely identify a market.
   function executeOutstandingNextPriceSettlementsUserMulti(address user, uint32[] memory marketIndexes) external {
-    for (uint256 i = 0; i < marketIndexes.length; i++) {
+    uint length = marketIndexes.length;
+    for (uint256 i = 0; i < length; i++) {
       _executeOutstandingNextPriceSettlements(user, marketIndexes[i]);
     }
   }
```

In the previous case, at each iteration of the loop, length is read from
memory. something like `mload(memory_offset)`. It takes `6` gas (3 for
`mload` and 3 to place `memory_offset`) in the stack.

In the replacement, the value is placed in the stack only once and each
iteration involves a `dupN` (3 gas). Saves around 3 gas per iteration.

Here are other places that can use this.

``` text
./contracts/contracts/LongShort.sol:776:    for (uint256 i = 0; i < marketIndexes.length; i++) {
./contracts/contracts/LongShort.sol:1063:    for (uint256 i = 0; i < length; i++) {
./contracts/contracts/Staker.sol:790:    for (uint256 i = 0; i < marketIndexes.length; i++) {
./contracts/contracts/mocks/BandOracleMock.sol:84:    for (uint256 i = 0; i < _bases.length; i++) {
./contracts/contracts/testing/LongShortInternalStateSetters.sol:34:    for (uint256 i = 0; i < marketIndexes.length; i++) {
```


