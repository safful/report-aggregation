## Tags

- bug
- sponsor confirmed
- G (Gas Optimization)
- resolved

# [`public` functions can be `external`](https://github.com/code-423n4/2021-10-tempus-findings/issues/45) 

# Handle

pants


# Vulnerability details

These `public` functions are never called by their contract:
- `TempusAMM.getSwapAmountToEndWithEqualShares()`
- `TempusAMM.getRate()`
- `AaveTempusPool.currentInterestRate()`
- `AaveTempusPool.numAssetsPerYieldToken()`
- `AaveTempusPool.numYieldTokensPerAsset()`
- `CompoundTempusPool.currentInterestRate()`
- `CompoundTempusPool.numAssetsPerYieldToken()`
- `CompoundTempusPool.numYieldTokensPerAsset()`
- `LidoTempusPool.currentInterestRate()`
- `LidoTempusPool.numAssetsPerYieldToken()`
- `LidoTempusPool.numYieldTokensPerAsset()`
- `ERC20FixedSupply.decimals()`
- `ERC20OwnerMintableToken.burn()`
- `ERC20OwnerMintableToken.burnFrom()`
- `PoolShare.decimals()`
- `PermanentlyOwnable.renounceOwnership()`
- `TempusController.depositYieldBearing()`
- `TempusController.depositBacking()`
- `TempusController.redeemToYieldBearing()`
- `TempusController.redeemToBacking()`
- `TempusPool.estimatedMintedShares()`
- `TempusPool.estimatedRedeem()`

Therefore, their visibility can be reduced to `external`.

## Impact
`external` functions are cheaper than `public` functions.

## Proof of Concept
https://gus-tavo-guim.medium.com/public-vs-external-functions-in-solidity-b46bcf0ba3ac

## Tool Used
Manual code review.

## Recommended Mitigation Steps
Define these functions as `external`.

