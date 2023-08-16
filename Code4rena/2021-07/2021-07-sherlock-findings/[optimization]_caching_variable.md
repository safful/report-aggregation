## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [[Optimization] Caching variable](https://github.com/code-423n4/2021-07-sherlock-findings/issues/81) 

# Handle

hrkrshnn


# Vulnerability details

## Caching variable

``` diff
modified   contracts/facets/SherXERC20.sol
@@ -66,8 +66,9 @@ contract SherXERC20 is IERC20, ISherXERC20 {
     require(_spender != address(0), 'SPENDER');
     require(_amount != 0, 'AMOUNT');
     SherXERC20Storage.Base storage sx20 = SherXERC20Storage.sx20();
-    sx20.allowances[msg.sender][_spender] = sx20.allowances[msg.sender][_spender].add(_amount);
-    emit Approval(msg.sender, _spender, sx20.allowances[msg.sender][_spender]);
+    uint256 newAllowance = sx20.allowances[msg.sender][_spender].add(_amount);
+    sx20.allowances[msg.sender][_spender] = newAllowance;
+    emit Approval(msg.sender, _spender, newAllowance);
     return true;
   }
```

The above change would avoid a `sload`, and will instead use `dupX`,
saving \`100\` gas.


