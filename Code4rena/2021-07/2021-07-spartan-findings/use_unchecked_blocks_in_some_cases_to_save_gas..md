## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Use unchecked blocks in some cases to save gas.](https://github.com/code-423n4/2021-07-spartan-findings/issues/74) 

# Handle

hrkrshnn


# Vulnerability details

## Use unchecked blocks when safemath is not required

In some cases, it's unnecessary to use the default checked arithmetic. In such cases, wrapping the block in unchecked
would save gas.

One example is:

``` diff
@@ -271,11 +272,13 @@ contract Pool is iBEP20 {

     // Check the TOKEN amount received by this Pool
     function _getAddedTokenAmount() internal view returns(uint256 _actual){
-        uint _tokenBalance = iBEP20(TOKEN).balanceOf(address(this));
-        if(_tokenBalance > tokenAmount){
-            _actual = _tokenBalance-(tokenAmount);
-        } else {
-            _actual = 0;
+        uint _tokenBalance = iBEP20(TOKEN).balanceOf(address(this));
+       unchecked {
+            if(_tokenBalance > tokenAmount){
+                _actual = _tokenBalance-(tokenAmount);
+            } else {
+                _actual = 0;
+            }

```

For loops, such optimizations would save a lot of gas.

``` diff
@@ -356,9 +359,11 @@ contract Pool is iBEP20 {

     function addFee(uint _rev) internal {
         uint _n = revenueArray.length; // 2
+       require(_n > 0);
+       unchecked {
         for (uint i = _n - 1; i > 0; i--) {
             revenueArray[i] = revenueArray[i - 1];
         }
+       }
         revenueArray[0] = _rev;
     }
 }
```


