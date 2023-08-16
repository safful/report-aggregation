## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Caching length in for loops](https://github.com/code-423n4/2021-08-notional-findings/issues/88) 

# Handle

hrkrshnn


# Vulnerability details

## Caching length in for loops

Consider a generic example of an array `arr` and the following loop:

``` solidity
for (uint i = 0; i < arr.length; i++) {
    // do something that doesn't change arr.length
}
```

In the above case, the solidity compiler will always read the length of
the array during each iteration. That is, if it is a storage array, this
is an extra `sload` operation (100 additional extra gas for each
iteration except for the first) and if it is a memory array, this is an
extra `mload` operation (3 additional gas for each iteration except for
the first).

This extra costs can be avoided by caching the array length (in stack):

``` solidity
uint length = arr.length;
for (uint i = 0; i < length; i++) {
    // do something that doesn't change arr.length
}
```

In the above example, the `sload` or `mload` operation is only done once
and subsequently replaced by a cheap `dupN` instruction.

This optimization is especially important if it is a storage array or if
it is a lengthy for loop.

Note that the Yul based optimizer (not enabled by default; only relevant
if you are using `--experimental-via-ir` or the equivalent in standard
JSON) can sometimes do this caching automatically. However, this is
likely not the case in your project.

### Examples

Here are some examples where this can be applied (found using a simple
grep)

``` txt
./contracts/external/Views.sol:187:        for (uint256 i = 0; i < cashGroup.maxMarketIndex; i++) {
./contracts/external/actions/BatchAction.sol:42:        for (uint256 i; i < actions.length; i++) {
./contracts/external/actions/BatchAction.sol:120:        for (uint256 i; i < actions.length; i++) {
./contracts/external/actions/ERC1155Action.sol:62:        for (uint256 i; i < accounts.length; i++) {
./contracts/external/actions/ERC1155Action.sol:91:        for (uint256 i; i < portfolio.length; i++) {
./contracts/external/actions/ERC1155Action.sol:240:        for (uint256 i; i < ids.length; i++) {
./contracts/external/actions/InitializeMarketsAction.sol:121:        for (uint256 i = 1; i < nToken.portfolioState.storedAssets.length; i++) {
./contracts/external/actions/InitializeMarketsAction.sol:146:        for (uint256 i = 1; i < nToken.portfolioState.storedAssets.length; i++) {
./contracts/external/actions/InitializeMarketsAction.sol:537:        for (uint256 i; i < nToken.cashGroup.maxMarketIndex; i++) {
./contracts/external/actions/TradingAction.sol:81:        for (uint256 i; i < trades.length; i++) {
./contracts/external/actions/TradingAction.sol:123:        for (uint256 i; i < trades.length; i++) {
./contracts/external/actions/nTokenMintAction.sol:110:        for (uint256 marketIndex = nToken.cashGroup.maxMarketIndex; marketIndex > 0; marketIndex--) {
./contracts/external/actions/nTokenRedeemAction.sol:138:        for (uint256 i; i < markets.length; i++) {
./contracts/external/actions/nTokenRedeemAction.sol:222:        for (uint256 i; i < nToken.portfolioState.storedAssets.length; i++) {
./contracts/external/actions/nTokenRedeemAction.sol:265:        for (uint256 i; i < markets.length; i++) {
./contracts/external/adapters/CompoundToNotionalV2.sol:81:        for (uint256 i; i < notionalV2CollateralIds.length; i++) {
./contracts/external/governance/NoteERC20.sol:98:        for (uint256 i = 0; i < initialGrantAmount.length; i++) {
./contracts/internal/balances/BalanceHandler.sol:300:        for (uint256 i; i < settleAmounts.length; i++) {
./contracts/internal/liquidation/LiquidateCurrency.sol:31:        for (uint256 i; i < portfolio.length; i++) {
./contracts/internal/liquidation/LiquidateCurrency.sol:345:        for (uint256 i; i < portfolioState.storedAssets.length; i++) {
./contracts/internal/liquidation/LiquidateCurrency.sol:449:        for (uint256 i; i < portfolioState.storedAssets.length; i++) {
./contracts/internal/liquidation/LiquidateCurrency.sol:528:            for (uint256 i; i < markets.length; i++) {
./contracts/internal/liquidation/LiquidatefCash.sol:77:        for (uint256 i; i < portfolio.length; i++) {
./contracts/internal/liquidation/LiquidatefCash.sol:131:        for (uint256 i; i < fCashMaturities.length; i++) {
./contracts/internal/liquidation/LiquidatefCash.sol:232:        for (uint256 i; i < fCashMaturities.length; i++) {
./contracts/internal/liquidation/LiquidatefCash.sol:495:        for (uint256 i; i < assets.length; i++) {
./contracts/internal/markets/CashGroup.sol:297:        for (uint256 i; i < cashGroup.liquidityTokenHaircuts.length; i++) {
./contracts/internal/markets/CashGroup.sol:309:        for (uint256 i; i < cashGroup.rateScalars.length; i++) {
./contracts/internal/nTokenHandler.sol:279:        for (uint256 i; i < depositShares.length; i++) {
./contracts/internal/nTokenHandler.sol:306:        for (uint256 i; i < proportions.length; i++) {
./contracts/internal/nTokenHandler.sol:371:        for (; i < array1.length; i++) {
./contracts/internal/nTokenHandler.sol:494:            for (uint256 i; i < nToken.portfolioState.storedAssets.length; i++) {
./contracts/internal/portfolio/BitmapAssetsHandler.sol:96:        for (uint256 i; i < assets.length; i++) {
./contracts/internal/portfolio/PortfolioHandler.sol:20:        for (uint256 i; i < assets.length; i++) {
./contracts/internal/portfolio/PortfolioHandler.sol:40:        for (uint256 i; i < assetArray.length; i++) {
./contracts/internal/portfolio/PortfolioHandler.sol:129:        for (uint256 i; i < newAssets.length; i++) {
./contracts/internal/portfolio/PortfolioHandler.sol:161:        for (uint256 i; i < portfolioState.storedAssets.length; i++) {
./contracts/internal/portfolio/PortfolioHandler.sol:171:        for (uint256 i; i < portfolioState.storedAssets.length; i++) {
./contracts/internal/portfolio/PortfolioHandler.sol:202:        for (uint256 i; i < portfolioState.newAssets.length; i++) {
./contracts/internal/portfolio/PortfolioHandler.sol:283:        for (uint256 i; i < portfolioState.storedAssets.length; i++) {
./contracts/internal/portfolio/TransferAssets.sol:47:        for (uint256 i; i < assets.length; i++) {
./contracts/internal/settlement/SettlePortfolioAssets.sol:32:        for (uint256 i = portfolioState.storedAssets.length; (i--) > 0;) {
./contracts/internal/settlement/SettlePortfolioAssets.sol:137:        for (uint256 i; i < portfolioState.storedAssets.length; i++) {
./contracts/internal/valuation/AssetHandler.sol:231:        for (uint256 i = portfolioIndex; i < assets.length; i++) {
./contracts/internal/valuation/AssetHandler.sol:250:        for (; j < assets.length; j++) {
./contracts/mocks/BaseMockLiquidation.sol:52:        for (uint256 i = 0; i < cashGroup.maxMarketIndex; i++) {
./contracts/mocks/BaseMockLiquidation.sol:72:        for (uint256 i; i < portfolioState.storedAssets.length; i++) {
./contracts/mocks/MockFlashLender.sol:30:        for (uint256 i; i < assets.length; i++) {
./contracts/mocks/MockFlashLender.sol:39:        for (uint256 i; i < assets.length; i++) {
```


