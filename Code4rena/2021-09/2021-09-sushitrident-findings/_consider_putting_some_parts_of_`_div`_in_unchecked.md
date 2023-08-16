## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [ Consider putting some parts of `_div` in unchecked](https://github.com/code-423n4/2021-09-sushitrident-findings/issues/118) 

# Handle

hrkrshnn


# Vulnerability details

## Consider putting some parts of `_div` in unchecked

[Context](https://github.com/sushiswap/trident/blob/9130b10efaf9c653d74dc7a65bde788ec4b354b5/contracts/pool/IndexPool.sol#L335)

``` diff
modified   contracts/pool/IndexPool.sol
@@ -332,7 +332,8 @@ contract IndexPool is IPool, TridentERC20 {

     function _div(uint256 a, uint256 b) internal pure returns (uint256 c2) {
         uint256 c0 = a * BASE;
-        uint256 c1 = c0 + (b / 2);
+        unchecked { uint256 tmp =  b / 2; }
+        uint256 c1 = c0 + tmp;
         c2 = c1 / b;
     }
```

Looking at the optimized assembly generated, the unchecked version
doesn't seem to have the additional check for division by zero. I'm not
entirely sure why, but my guess is because of inlining. Also consider
replacing the division by inline assembly.

``` solidity
uint tmp;
assembly {
    tmp := div(b, 2)
}
uint256 c1 = c0 + tmp;
```

The change avoids an `if` condition which checks if the divisor is zero,
which the optimizer is currently unable to optimize out. The gas savings
would be around 16 (reduces a `jumpi`, `push 0` and `dupN`). Since the
`_div` function is called from throughout the code (also present in
other contracts), this may be worth considering, although I admit this
might be too much of a micro-optimization.


