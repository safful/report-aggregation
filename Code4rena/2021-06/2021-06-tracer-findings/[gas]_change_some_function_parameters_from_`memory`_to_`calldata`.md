## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [[Gas] Change some function parameters from `memory` to `calldata`](https://github.com/code-423n4/2021-06-tracer-findings/issues/36) 

# Handle

hrkrshnn


# Vulnerability details

## Impact

Gas optimization.

## For function arguments, change `memory` to `calldata`

There are several places where this is applicable, however, will point
out one such occasion:

``` diff
modified   src/contracts/Trader.sol
@@ -64,7 +64,7 @@ contract Trader is ITrader {
      * @param makers An array of signed make orders
      * @param takers An array of signed take orders
      */
-    function executeTrade(Types.SignedLimitOrder[] memory makers, Types.SignedLimitOrder[] memory takers)
+    function executeTrade(Types.SignedLimitOrder[] calldata makers, Types.SignedLimitOrder[] calldata takers)
         external
         override
     {
@@ -144,7 +144,7 @@ contract Trader is ITrader {
      * @dev Should only be called with a verified signedOrder and with index
      *      < signedOrders.length
      */
-    function grabOrder(Types.SignedLimitOrder[] memory signedOrders, uint256 index)
+    function grabOrder(Types.SignedLimitOrder[] calldata signedOrders, uint256 index)
         internal
         returns (Perpetuals.Order memory)
     {
```

Reason: when you specify `memory` for a (non value type)
function-parameter for an external function, the following happens: the
compiler would copy elements from `calldata` to `memory` (using the
opcode `calldatacopy`.) Then later on, the internal call (here
`grabOrder`) would pass a memory reference. However, this is a great
example of where copying to memory is unnecessary. Note that there is
also the opcode `calldataload` to read an offset from `calldata`. By
changing the location from `memory` to `calldata`, you avoid this
expensive copy from `calldata` to `memory`, while managing to do exactly
what's needed.

You would only have to use `memory` if the function has to modify the
parameter, in which case a copy is really needed as `calldata` cannot be
modified.



