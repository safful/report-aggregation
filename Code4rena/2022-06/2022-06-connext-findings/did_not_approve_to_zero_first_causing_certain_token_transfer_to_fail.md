## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [Did Not Approve To Zero First Causing Certain Token Transfer To Fail](https://github.com/code-423n4/2022-06-connext-findings/issues/154) 

# Lines of code

https://github.com/code-423n4/2022-06-connext/blob/4dd6149748b635f95460d4c3924c7e3fb6716967/contracts/contracts/core/connext/facets/BridgeFacet.sol#L984
https://github.com/code-423n4/2022-06-connext/blob/4dd6149748b635f95460d4c3924c7e3fb6716967/contracts/contracts/core/connext/libraries/AssetLogic.sol#L347


# Vulnerability details

## Proof-of-Concept

Some tokens (like USDT) do not work when changing the allowance from an existing non-zero allowance value. For example Tether (USDT)'s `approve()` function will revert if the current approval is not zero, to protect against front-running changes of approvals.

#### Instance 1 - `BridgeFacet._reconcileProcessPortal`

The following function must be approved by zero first, and then the ` SafeERC20.safeIncreaseAllowance` function can be called. Otherwise, the `_reconcileProcessPortal` function will revert everytime it handles such kind of tokens. Understood from the [comment](https://github.com/code-423n4/2022-06-connext/blob/4dd6149748b635f95460d4c3924c7e3fb6716967/contracts/contracts/core/connext/facets/BridgeFacet.sol#L1025) that after the backUnbacked call there could be a remaining allowance.

[https://github.com/code-423n4/2022-06-connext/blob/4dd6149748b635f95460d4c3924c7e3fb6716967/contracts/contracts/core/connext/facets/BridgeFacet.sol#L984](https://github.com/code-423n4/2022-06-connext/blob/4dd6149748b635f95460d4c3924c7e3fb6716967/contracts/contracts/core/connext/facets/BridgeFacet.sol#L984)

```solidity
  function _reconcileProcessPortal(
    uint256 _amount,
    address _local,
    address _router,
    bytes32 _transferId
  ) private returns (uint256) {
  	..SNIP..
    SafeERC20.safeIncreaseAllowance(IERC20(adopted), s.aavePool, totalRepayAmount);

    (bool success, ) = s.aavePool.call(
      abi.encodeWithSelector(IAavePool.backUnbacked.selector, adopted, backUnbackedAmount, portalFee)
    );
  	..SNIP..
  }
```

#### Instance 2 - `BridgeFacet_swapAssetOut`

The following fucntion must first be approved by zero, follow by the actual allowance to be approved. Otherwise, the `_swapAssetOut` function will revert everytime it handles such kind of tokens.

[https://github.com/code-423n4/2022-06-connext/blob/4dd6149748b635f95460d4c3924c7e3fb6716967/contracts/contracts/core/connext/libraries/AssetLogic.sol#L347](https://github.com/code-423n4/2022-06-connext/blob/4dd6149748b635f95460d4c3924c7e3fb6716967/contracts/contracts/core/connext/libraries/AssetLogic.sol#L347)

```solidity
  function _swapAssetOut(
    bytes32 _canonicalId,
    address _assetIn,
    address _assetOut,
    uint256 _amountOut,
    uint256 _maxIn
  )
    internal
    returns (
      bool,
      uint256,
      address
    )
  {
    AppStorage storage s = LibConnextStorage.connextStorage();

    bool success;
    uint256 amountIn;

    // Swap the asset to the proper local asset
    if (stableSwapPoolExist(_canonicalId)) {
      // get internal swap pool
      SwapUtils.Swap storage ipool = s.swapStorages[_canonicalId];
      // if internal swap pool exists
      uint8 tokenIndexIn = getTokenIndexFromStableSwapPool(_canonicalId, _assetIn);
      uint8 tokenIndexOut = getTokenIndexFromStableSwapPool(_canonicalId, _assetOut);
      // calculate slippage before performing swap
      // NOTE: this is less efficient then relying on the `swapInternalOut` revert, but makes it easier
      // to handle slippage failures (this can be called during reconcile, so must not fail)
      if (_maxIn >= ipool.calculateSwapInv(tokenIndexIn, tokenIndexOut, _amountOut)) {
        success = true;
        amountIn = ipool.swapInternalOut(tokenIndexIn, tokenIndexOut, _amountOut, _maxIn);
      }
      // slippage is too high to perform swap: success = false, amountIn = 0
    } else {
      // Otherwise, swap via stable swap pool
      IStableSwap pool = s.adoptedToLocalPools[_canonicalId];
      uint256 _amountIn = pool.calculateSwapOutFromAddress(_assetIn, _assetOut, _amountOut);
      if (_amountIn <= _maxIn) {
        // set the success
        success = true;

        // perform the swap
        SafeERC20.safeApprove(IERC20(_assetIn), address(pool), _amountIn);
        amountIn = pool.swapExactOut(_amountOut, _assetIn, _assetOut, _maxIn);
      }
      // slippage is too high to perform swap: success = false, amountIn = 0
    }

    return (success, amountIn, _assetOut);
  }
```

## Impact

Both the`_reconcileProcessPortal` and `_swapAssetOut` functions are called during repayment to Aave Portal if the fast-transfer was executed using portal liquidity. Thus, it is core part of the token transfer process within Connext, and failure of any of these functions would disrupt the AAVE repayment process.

Since both functions affect the AAVE repayment process, I'm grouping them as one issue.

## Recommended Mitigation Steps

As Connext bridges/routers deal with all sort of tokens existed in various domains/chains, the protocol should try to implement measure to ensure that it is compatible with as much tokens as possible for future growth and availability of the protocol.

#### Instance 1 - `BridgeFacet._reconcileProcessPortal`

It is recommended to set the allowance to zero before increasing the allowance

```solidity
SafeERC20.safeApprove(IERC20(_assetIn), address(pool), 0);
SafeERC20.safeIncreaseAllowance(IERC20(adopted), s.aavePool, totalRepayAmount);
```

#### Instance 2 - `BridgeFacet_swapAssetOut`

It is recommended to set the allowance to zero before each approve call.

```solidity
SafeERC20.safeApprove(IERC20(_assetIn), address(pool), 0);
SafeERC20.safeApprove(IERC20(_assetIn), address(pool), _amountIn);
```

