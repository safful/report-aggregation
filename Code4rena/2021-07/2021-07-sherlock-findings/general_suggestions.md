## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [General suggestions](https://github.com/code-423n4/2021-07-sherlock-findings/issues/89) 

# Handle

hrkrshnn


# Vulnerability details

## Use `type(uintX).max` instead of `uintX(-1)`

``` diff
modified   contracts/facets/Gov.sol
@@ -55,7 +55,7 @@ contract Gov is IGov {
     GovStorage.Base storage gs = GovStorage.gs();
     SherXStorage.Base storage sx = SherXStorage.sx();

-    return sx.sherXPerBlock.mul(gs.watsonsSherxWeight).div(uint16(-1));
+    return sx.sherXPerBlock.mul(gs.watsonsSherxWeight).div(type(uint16).max);
   }

   function getWatsonsUnmintedSherX() external view override returns (uint256) {
```

Use `type(integerType).max` for such cases. There are also other places
that could use this.

## Have NatSpec comments for all functions
## Avoid the diamond standard

The most significant optimization that can be done in the contract is to
get rid of the diamond standard, because, proxy architectures are
inherently expensive. Unless there are specific reasons, such as
contract size limits, a diamond makes the contract unnecessary complex.
Also, try to avoid upgradability if you can afford it.


