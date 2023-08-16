## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [[Optimization] Caching in for loops](https://github.com/code-423n4/2021-07-sherlock-findings/issues/86) 

# Handle

hrkrshnn


# Vulnerability details

## Caching in for loops

``` diff
modified   contracts/facets/PoolBase.sol
@@ -128,19 +128,21 @@ contract PoolBase is IPoolBase {
   {
     PoolStorage.Base storage ps = baseData();
     GovStorage.Base storage gs = GovStorage.gs();
-    for (uint256 i = 0; i < ps.unstakeEntries[_staker].length; i++) {
-      if (ps.unstakeEntries[_staker][i].blockInitiated == 0) {
+    PoolStorage.UnstakeEntry[] storage entries = ps.unstakeEntries[_staker];
+    uint length = entries.length;
+    for (uint256 i = 0; i < length; i++) {
+      if (entries[i].blockInitiated == 0) {
         continue;
       }
       if (
-        ps.unstakeEntries[_staker][i].blockInitiated + gs.unstakeCooldown + gs.unstakeWindow <=
+        entries[i].blockInitiated + gs.unstakeCooldown + gs.unstakeWindow <=
         uint40(block.number)
       ) {
         continue;
       }
       return i;
     }
-    return ps.unstakeEntries[_staker].length;
+    return length;
   }
```

Caching expensive state variables would avoid re-reading from storage.
Solidity's optimizer currently will not be able to cache this value (the
IR based codegen and the Yul optimizer can do it; but that is not
activated by default).


