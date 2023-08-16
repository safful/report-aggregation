## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [[Optimization] A branchless version of an if else statement](https://github.com/code-423n4/2021-07-sherlock-findings/issues/82) 

# Handle

hrkrshnn


# Vulnerability details

## Writing a branch less version

``` diff
@@ -76,11 +77,11 @@ contract SherXERC20 is IERC20, ISherXERC20 {
     require(_amount != 0, 'AMOUNT');
     SherXERC20Storage.Base storage sx20 = SherXERC20Storage.sx20();
     uint256 oldValue = sx20.allowances[msg.sender][_spender];
-    if (_amount > oldValue) {
-      sx20.allowances[msg.sender][_spender] = 0;
-    } else {
-      sx20.allowances[msg.sender][_spender] = oldValue.sub(_amount);
-    }
+    uint256 newValue;
+    assembly {
+        newValue := mul(gt(oldValue, _amount), sub(oldValue, _amount))
+    }
+    sx20.allowances[msg.sender][_spender] = newValue;
     emit Approval(msg.sender, _spender, sx20.allowances[msg.sender][_spender]);
     return true;
   }
```

The branch-less version avoids at least two `jumpi`, i.e., at least 20
gas and some additional stack operations, along with deploy costs.

Here's a SMT proof that the transformation is equivalent:

``` python
from z3 import *

# A SMT proof that
#
# if (_amount > oldValue) {
#   sx20.allowances[msg.sender][_spender] = 0;
# } else {
#   sx20.allowances[msg.sender][_spender] = oldValue.sub(_amount);
# }
#
# is same as
#
# assembly {
#     newValue := mul(gt(oldValue, _amount), sub(oldValue, _amount))
# }
# sx20.allowances[msg.sender][_spender] = newValue;
#

n_bits = 256
amount = BitVec('amount', n_bits)
oldValue = BitVec('oldValue', n_bits)
allowance = BitVec('oldValue', n_bits)

old_allowance_computation = If(UGT(amount, oldValue), 0, oldValue - amount)

def GT(x, y):
    return If(UGT(x, y), BitVecVal(1, n_bits), BitVecVal(0, n_bits))
def MUL(x, y):
    return x * y
def SUB(x, y):
    return x - y

new_allowance_computation = MUL(GT(oldValue, amount), SUB(oldValue, amount))

solver = Solver()
solver.add(old_allowance_computation != new_allowance_computation)

result = solver.check()
print(result)
# unsat
```


