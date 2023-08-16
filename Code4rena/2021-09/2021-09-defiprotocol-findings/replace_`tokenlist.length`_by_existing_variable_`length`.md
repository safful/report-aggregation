## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Replace `tokenList.length` by existing variable `length`](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/230) 

# Handle

hrkrshnn


# Vulnerability details

## Replace `tokenList.length` by existing variable `length`

``` diff
modified   contracts/contracts/Basket.sol
@@ -61,7 +61,7 @@ contract Basket is IBasket, ERC20Upgradeable {
             require(_tokens[i] != address(0));
             require(_weights[i] > 0);

-            for (uint256 x = 0; x < tokenList.length; x++) {
+            for (uint256 x = 0; x < length; x++) {
                 require(_tokens[i] != tokenList[x]);
             }
```

Context:
<https://github.com/code-423n4/2021-09-defiProtocol/blob/main/contracts/contracts/Basket.sol#L64>

The value `tokenList.length` is read from memory and therefore requires
a `mload(...)` (6 gas for `push memory_offset` + `mload`). On the other
hand, this value is already available in the stack as `length` and could
just be `dup-ed` (3 gas). Saves 3 gas for each loop iteration of the
interior loop.


