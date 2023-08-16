## Tags

- bug
- 0 (Non-critical)
- mStableYieldSource
- sponsor confirmed

# [Incorrect comment about memory](https://github.com/code-423n4/2021-07-pooltogether-findings/issues/69) 

# Handle

hrkrshnn


# Vulnerability details

## Incorrect comment

[Context](https://github.com/pooltogether/pooltogether-mstable/blob/0bcbd363936fadf5830e9c48392415695896ddb5/contracts/yield-source/MStableYieldSource.sol#L47)

``` diff
modified   contracts/MStableYieldSource.sol
@@ -44,7 +44,7 @@ contract MStableYieldSource is IYieldSource, ReentrancyGuard {

     constructor(ISavingsContractV2 _savings) ReentrancyGuard() {
         // As immutable storage variables can not be accessed in the constructor,
-        // create in-memory variables that can be used instead.
+        // create in-stack variables that can be used instead.
         IERC20 mAssetMemory = IERC20(_savings.underlying());

         // infinite approve Savings Contract to transfer mAssets from this contract
```

The comment and therefore the variable name aren't accurate. The value
would be in stack, and not memory.

However, this doesn't affect the code in any way.


