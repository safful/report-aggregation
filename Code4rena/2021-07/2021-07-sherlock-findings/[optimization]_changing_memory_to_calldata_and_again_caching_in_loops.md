## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [[Optimization] Changing memory to calldata and again caching in loops](https://github.com/code-423n4/2021-07-sherlock-findings/issues/87) 

# Handle

hrkrshnn


# Vulnerability details

## Change memory to calldata and caching in loop

``` diff
modified   contracts/facets/Manager.sol
@@ -139,16 +139,17 @@ contract Manager is IManager {

   function setProtocolPremiumAndTokenPrice(
     bytes32 _protocol,
-    IERC20[] memory _token,
-    uint256[] memory _premium,
-    uint256[] memory _newUsd
+    IERC20[] calldata _token,
+    uint256[] calldata _premium,
+    uint256[] calldata _newUsd
   ) external override onlyGovMain {
     require(_token.length == _premium.length, 'LENGTH_1');
     require(_token.length == _newUsd.length, 'LENGTH_2');

     (uint256 usdPerBlock, uint256 usdPool) = _getData();

-    for (uint256 i; i < _token.length; i++) {
+    uint length = _token.length;
+    for (uint256 i; i < length; i++) {
       LibPool.payOffDebtAll(_token[i]);
       (usdPerBlock, usdPool) = _setProtocolPremiumAndTokenPrice(
         _protocol,
```

About caching in loop, see my other report on why it's needed.

For the old code, i.e., having an array in memory, there is an
unnecessary copy from `calldata` to `memory`. In the proposed patch,
this unnecessary copy is avoided and values are directly read from
`calldata` by using `calldataload(...)` instead of going via
`calldatacopy(...)`, then `mload(...)`). Saves memory expansion cost,
and cost of copying from `calldata` to `memory`.

There are several other places throughout the codebase where the same
optimization can be used. I've not provided an exhaustive list.


