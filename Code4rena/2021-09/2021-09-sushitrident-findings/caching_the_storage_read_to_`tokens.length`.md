## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Caching the storage read to `tokens.length`](https://github.com/code-423n4/2021-09-sushitrident-findings/issues/115) 

# Handle

hrkrshnn


# Vulnerability details

## Caching the storage read to `tokens.length`

Ignoring the caching for the for-loop condition (see my other issue
"Caching the length in for loops"), this would save an additional
`sload`, around `100` gas.

[Context](https://github.com/sushiswap/trident/blob/9130b10efaf9c653d74dc7a65bde788ec4b354b5/contracts/pool/IndexPool.sol#L129).

``` diff
modified   contracts/pool/IndexPool.sol
@@ -121,12 +121,12 @@ contract IndexPool is IPool, TridentERC20 {
         (address recipient, bool unwrapBento, uint256 toBurn) = abi.decode(data, (address, bool, uint256));

         uint256 ratio = _div(toBurn, totalSupply);
-        withdrawnAmounts = new TokenAmount[](tokens.length);
+        uint length = tokens.length;
+        withdrawnAmounts = new TokenAmount[](length);

         _burn(address(this), toBurn);

-        for (uint256 i = 0; i < tokens.length; i++) {
+        for (uint256 i = 0; i < length; i++) {
             address tokenOut = tokens[i];
             uint256 balance = records[tokenOut].reserve;
             uint120 amountOut = uint120(_mul(ratio, balance));
```


