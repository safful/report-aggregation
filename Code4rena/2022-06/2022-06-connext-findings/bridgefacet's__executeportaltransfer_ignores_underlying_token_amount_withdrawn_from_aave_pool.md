## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [BridgeFacet's _executePortalTransfer ignores underlying token amount withdrawn from Aave pool](https://github.com/code-423n4/2022-06-connext-findings/issues/181) 

# Lines of code

https://github.com/code-423n4/2022-06-connext/blob/b4532655071566b33c41eac46e75be29b4a381ed/contracts/contracts/core/connext/facets/BridgeFacet.sol#L882-L900


# Vulnerability details

_executePortalTransfer can introduce underlying token deficit by accounting for full underlying amount received from Aave unconditionally on what was actually withdrawn from Aave pool. Actual amount withdrawn is returned by `IAavePool(s.aavePool).withdraw()`, but currently is not used.

Setting the severity to medium as this can end up with a situation of partial insolvency, when where are a surplus of atokens, but deficit of underlying tokens in the bridge, so bridge functionality can become unavailable as there will be not enough underlying tokens, which were used up in the previous operations when atokens wasn't converted to underlying fully and underlying tokens from other operations were used up instead without accounting. I.e. the system in this situation supposes that all atokens are in the form of underlying tokens while there will be some atokens left unconverted due to withdrawal being only partial.

## Proof of Concept

Call sequence here is execute() -> _handleExecuteLiquidity() -> _executePortalTransfer(). 

BridgeFacet._executePortalTransfer() mints the atokens needed, then withdraws them from Aave pool, always accounting for the full withdrawal:

https://github.com/code-423n4/2022-06-connext/blob/b4532655071566b33c41eac46e75be29b4a381ed/contracts/contracts/core/connext/facets/BridgeFacet.sol#L882-L900

```solidity
  /**
   * @notice Uses Aave Portals to provide fast liquidity
   */
  function _executePortalTransfer(
    bytes32 _transferId,
    uint256 _fastTransferAmount,
    address _local,
    address _router
  ) internal returns (uint256, address) {
    // Calculate local to adopted swap output if needed
    (uint256 userAmount, address adopted) = AssetLogic.calculateSwapFromLocalAssetIfNeeded(_local, _fastTransferAmount);

    IAavePool(s.aavePool).mintUnbacked(adopted, userAmount, address(this), AAVE_REFERRAL_CODE);

    // Improvement: Instead of withdrawing to address(this), withdraw directly to the user or executor to save 1 transfer
    IAavePool(s.aavePool).withdraw(adopted, userAmount, address(this));

    // Store principle debt
    s.portalDebt[_transferId] = userAmount;
```


Aave pool's withdraw() returns the amount of underlying asset that was actually withdrawn:

https://github.com/aave/aave-v3-core/blob/master/contracts/protocol/pool/Pool.sol#L196-L217

https://github.com/aave/aave-v3-core/blob/master/contracts/protocol/libraries/logic/SupplyLogic.sol#L93-L111

If a particular lending pool has liquidity shortage at the moment, say all underlying is lent out, full withdrawal of the requested underlying token amount will not be possible.

## Recommended Mitigation Steps

Consider adjusting for the amount actually withdrawn. Also the buffer that stores minted but not yet used atoken amount, say aAmountStored, can be introduced.

For example: 

``` 
+   uint256 amountNeeded = userAmount < aAmountStored ? 0 : userAmount - aAmountStored;

-   IAavePool(s.aavePool).mintUnbacked(adopted, userAmount, address(this), AAVE_REFERRAL_CODE);
+   if (amountNeeded > 0) {
+       IAavePool(s.aavePool).mintUnbacked(adopted, amountNeeded, address(this), AAVE_REFERRAL_CODE);
+   }

    // Improvement: Instead of withdrawing to address(this), withdraw directly to the user or executor to save 1 transfer
-   IAavePool(s.aavePool).withdraw(adopted, userAmount, address(this));
+   uint256 amountWithdrawn = IAavePool(s.aavePool).withdraw(adopted, userAmount, address(this));

    // Store principle debt
-   s.portalDebt[_transferId] = userAmount;
+   s.portalDebt[_transferId] = amountWithdrawn; // can't exceed userAmount
+   aAmountStored = (userAmount < aAmountStored ? aAmountStored : userAmount) - amountWithdrawn; // we used amountWithdrawn

```

