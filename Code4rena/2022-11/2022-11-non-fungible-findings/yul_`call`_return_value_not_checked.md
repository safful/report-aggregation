## Tags

- bug
- 2 (Med Risk)
- primary issue
- selected for report
- sponsor confirmed
- M-01

# [Yul `call` return value not checked](https://github.com/code-423n4/2022-11-non-fungible-findings/issues/90) 

# Lines of code

https://github.com/code-423n4/2022-11-non-fungible/blob/323b7cbf607425dd81da96c0777c8b12e800305d/contracts/Exchange.sol#L212-L227


# Vulnerability details

### Author: rotcivegaf

### Impact

The Yul `call` return value on function `_returnDust` is not checked, which could leads to the `sender` lose funds

### Proof of Concept

The caller of the functions `bulkExecute` and `execute` could be a contract who may not implement the `fallback` or `receive` functions or reject the `call`, when a call to it with value sent in the function `_returnDust`, it will revert, thus it would fail to receive the `dust` ether

Proof: 
 - A contract use `bulkExecute`
 - One of the executions fails
 - The `Exchange` contract send the `dust`(Exchange balance) back to the contract 
 - This one for any reason reject the call
 - The `dust` stay in the `Exchange` contract
 - In the next call of `bulkExecute` or `execute` the balance of the `Exchange` contract(including the old `dust`) will send to the new caller
 - The second sender will get the funds of the first contract

### Tools Used

Review

### Recommended Mitigation Steps

```diff
+    error ReturnDustFail();
+
     function _returnDust() private {
         uint256 _remainingETH = remainingETH;
+        bool success;
         assembly {
             if gt(_remainingETH, 0) {
-                let callStatus := call(
+                success := call(
                     gas(),
                     caller(),
                     selfbalance(),
                     0,
                     0,
                     0,
                     0
                 )
             }
         }
+        if (!success) revert ReturnDustFail();
     }
```