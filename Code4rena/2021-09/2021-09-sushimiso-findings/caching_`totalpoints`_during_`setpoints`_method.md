## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Caching `totalPoints` during `setPoints` method](https://github.com/code-423n4/2021-09-sushimiso-findings/issues/135) 

# Handle

hrkrshnn


# Vulnerability details

## Caching `totalPoints` during `setPoints` method

Instead of constantly writing to the same slot in a for loop, write it
once at the end. This would save `100` gas for each iteration of the for
loop. (Since [EIP-2929](https://eips.ethereum.org/EIPS/eip-2929), the
cost of writing to a dirty storage slot is 100 gas)

``` diff
modified   contracts/Access/PointList.sol
@@ -65,6 +65,7 @@ contract PointList is IPointList, MISOAccessControls {
    function setPoints(address[] memory _accounts, uint256[] memory _amounts) external override {
         require(hasAdminRole(msg.sender) || hasOperatorRole(msg.sender), "PointList.setPoints: Sender must be operator");
         require(_accounts.length != 0, "PointList.setPoints: empty array");
         require(_accounts.length == _amounts.length, "PointList.setPoints: incorrect array length");
+        uint totalPointsCache = totalPoints;
         for (uint i = 0; i < _accounts.length; i++) {
             address account = _accounts[i];
             uint256 amount = _amounts[i];
@@ -72,9 +73,10 @@ contract PointList is IPointList, MISOAccessControls {

             if (amount != previousPoints) {
                 points[account] = amount;
-                totalPoints = totalPoints.sub(previousPoints).add(amount);
+                totalPointsCache = totalPointsCache.sub(previousPoints).add(amount);
                 emit PointsUpdated(account, previousPoints, amount);
             }
         }
+        totalPoints = totalPointsCache;
     }
 }
```


