## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed
- selected for report
- M-01

# [LBRouter.removeLiquidity returning wrong values](https://github.com/code-423n4/2022-10-traderjoe-findings/issues/105) 

# Lines of code

https://github.com/code-423n4/2022-10-traderjoe/blob/e81b78ddb7cc17f0ece921fbaef2c2521727094b/src/LBRouter.sol#L291


# Vulnerability details

## Impact
`LBRouter.removeLiquidity` reorders tokens when the user did not pass them in the pair order (ascending order):
```solidity
if (_tokenX != _LBPair.tokenX()) {
            (_tokenX, _tokenY) = (_tokenY, _tokenX);
            (_amountXMin, _amountYMin) = (_amountYMin, _amountXMin);
}
```
However, when returning `amountX` and `amountY`, it is ignored if the order was changed:
```solidity
(amountX, amountY) = _removeLiquidity(_LBPair, _amountXMin, _amountYMin, _ids, _amounts, _to);
```
Therefore, when the order of the tokens is swapped by the function, the return value `amountX` ("Amount of token X returned") in reality is the amount of the user-provided token Y that is returned and vice versa.

Because this is an exposed function that third-party protocols / contracts will use, this can cause them to malfunction. For instance, when integrating with Trader Joe, something natural to do is:
```
(uint256 amountAReceived, uint256 amountBReceived) = LBRouter.removeLiquidity(address(tokenA), address(tokenB), ...);
contractBalanceA += amountAReceived;
contractBalanceB += amountBReceived;
```
This snippet will only be correct when the token addresses are passed in the right order, which should not be the case. When they are not passed in the right order, the accounting of third-party contracts will be messed up, leading to vulnerabilities / lost funds there.

## Proof Of Concept
First consider the following diff, which shows a scenario when `LBRouter` does not switch `tokenX` and `tokenY`, resulting in correct return values:
```diff
--- a/test/LBRouter.Liquidity.t.sol
+++ b/test/LBRouter.Liquidity.t.sol
@@ -57,7 +57,9 @@ contract LiquidityBinRouterTest is TestHelper {
 
         pair.setApprovalForAll(address(router), true);
 
-        router.removeLiquidity(
+        uint256 token6BalBef = token6D.balanceOf(DEV);
+        uint256 token18BalBef = token18D.balanceOf(DEV);
+        (uint256 amountFirstRet, uint256 amountSecondRet) = router.removeLiquidity(
             token6D,
             token18D,
             DEFAULT_BIN_STEP,
@@ -70,7 +72,9 @@ contract LiquidityBinRouterTest is TestHelper {
         );
 
         assertEq(token6D.balanceOf(DEV), amountXIn);
+        assertEq(amountXIn, token6BalBef + amountFirstRet);
         assertEq(token18D.balanceOf(DEV), _amountYIn);
+        assertEq(_amountYIn, token18BalBef + amountSecondRet);
     }
 
     function testRemoveLiquidityReverseOrder() public {
```
This test passes (as it should). Now, consider the following diff, where `LBRouter` switches `tokenX` and `tokenY`:
```diff
--- a/test/LBRouter.Liquidity.t.sol
+++ b/test/LBRouter.Liquidity.t.sol
@@ -57,12 +57,14 @@ contract LiquidityBinRouterTest is TestHelper {
 
         pair.setApprovalForAll(address(router), true);
 
-        router.removeLiquidity(
-            token6D,
+        uint256 token6BalBef = token6D.balanceOf(DEV);
+        uint256 token18BalBef = token18D.balanceOf(DEV);
+        (uint256 amountFirstRet, uint256 amountSecondRet) = router.removeLiquidity(
             token18D,
+            token6D,
             DEFAULT_BIN_STEP,
-            totalXbalance,
             totalYBalance,
+            totalXbalance,
             ids,
             amounts,
             DEV,
@@ -70,7 +72,9 @@ contract LiquidityBinRouterTest is TestHelper {
         );
 
         assertEq(token6D.balanceOf(DEV), amountXIn);
+        assertEq(amountXIn, token6BalBef + amountSecondRet);
         assertEq(token18D.balanceOf(DEV), _amountYIn);
+        assertEq(_amountYIn, token18BalBef + amountFirstRet);
     }
 
     function testRemoveLiquidityReverseOrder() public {
```
This test should also pass (the order of the tokens was only switched), but it does not because the return values are mixed up.

## Recommended Mitigation Steps
Add the following statement in the end:
```solidity
if (_tokenX != _LBPair.tokenX()) {
	return (amountY, amountX);
}
```