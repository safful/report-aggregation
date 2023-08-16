## Tags

- bug
- sponsor confirmed
- 0 (Non-critical)

# [Consider avoiding low level calls to MasterDeployer](https://github.com/code-423n4/2021-09-sushitrident-findings/issues/121) 

# Handle

hrkrshnn


# Vulnerability details

## Consider avoiding low level calls to MasterDeployer

[Context](https://github.com/sushiswap/trident/blob/9130b10efaf9c653d74dc7a65bde788ec4b354b5/contracts/pool/ConstantProductPool.sol#L64)

The constructor uses low-level calls to the master deployer. It is more
idiomatic to rely on high level solidity to deal with the calls. The
difference would be additional checks on whether there is code at the
specified address (additional `100` gas) and automatically performing
the ABI decoding (may actually be more efficient than the manual
implementation.) (Note that this call will still be `staticcall`, since
`barFee` is a view function in the interface.)

Example:

``` diff
modified   contracts/pool/ConstantProductPool.sol
@@ -61,7 +61,7 @@ contract ConstantProductPool is IPool, TridentERC20 {
         require(_token1 != address(this), "INVALID_TOKEN");
         require(_swapFee <= MAX_FEE, "INVALID_SWAP_FEE");

-        (, bytes memory _barFee) = _masterDeployer.staticcall(abi.encodeWithSelector(IMasterDeployer.barFee.selector));
+        barFee = IMasterDeployer(_masterDeployer).barFee();
         (, bytes memory _barFeeTo) = _masterDeployer.staticcall(abi.encodeWithSelector(IMasterDeployer.barFeeTo.selector));
         (, bytes memory _bento) = _masterDeployer.staticcall(abi.encodeWithSelector(IMasterDeployer.bento.selector));

@@ -72,7 +72,6 @@ contract ConstantProductPool is IPool, TridentERC20 {
         unchecked {
             MAX_FEE_MINUS_SWAP_FEE = MAX_FEE - _swapFee;
         }
-        barFee = abi.decode(_barFee, (uint256));
         barFeeTo = abi.decode(_barFeeTo, (address));
         bento = abi.decode(_bento, (address));
         masterDeployer = _masterDeployer;
```


