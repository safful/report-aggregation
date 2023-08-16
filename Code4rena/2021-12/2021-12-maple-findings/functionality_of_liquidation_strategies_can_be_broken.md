## Tags

- bug
- 1 (Low Risk)
- disagree with severity
- sponsor confirmed

# [Functionality of liquidation strategies can be broken](https://github.com/code-423n4/2021-12-maple-findings/issues/35) 

# Handle

cmichel


# Vulnerability details

The liquidations strategies (`code-423n4/evm-league/56-maple/repo/liquidations-1.0.0-beta.1/contracts/SushiswapStrategy.sol/UniswapStrategy.sol`) check that the current contract balance in the `swap` callback exactly equals the `swapAmount_` parameter from `flashBorrowLiquidation`.
(The `swap` is called as a callback from `flashBorrowLiquidation`'s `liquidatePortion`).

```solidity
function swap(
    uint256 swapAmount_,
    uint256 minReturnAmount_,
    address collateralAsset_,
    address middleAsset_,
    address fundsAsset_,
    address profitDestination_
)
    external override
{
    // @audit grifer can send 1 wei. should >=
    require(IERC20Like(collateralAsset_).balanceOf(address(this)) == swapAmount_, "SushiswapStrategy:WRONG_COLLATERAL_AMT");
}
```

There's a griefing attacker where a keeper tries to liquidate and calls `flashBorrowLiquidation`  but an attacker frontruns this transaction and sends the smallest unit of the `collateralAsset_` to the contract, making this `require` call fail.

## Impact
The important automated liquidation strategies that Keepers might use do not work anymore, no liquidations are done in time, and bad debt can occur.

I'd rate this as high severity as the impact is big and it's also very easy to break this contract entirely with a single transfer:
- there's only one strategy contract for many liquidation contracts which means it's important that it's reliable
- it's enough to send a few tokens of collateral assets to the contract _once_ to break the `flashBorrowLiquidation/swap` functionality. Because when calling `flashBorrowLiquidation(swapAmount)`, the liquidation contract will always send exactly this `swapAmount` to the strategy, meaning the `IERC20Like(collateralAsset_).balanceOf(address(this)) == swapAmount_` comparison will always fail if there already were tokens in the contract.

## Recommended Mitigation Steps
Use a `IERC20Like(collateralAsset_).balanceOf(address(this)) >= swapAmount_` comparison instead.


