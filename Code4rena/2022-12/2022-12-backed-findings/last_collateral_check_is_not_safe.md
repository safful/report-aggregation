## Tags

- bug
- 2 (Med Risk)
- downgraded by judge
- satisfactory
- selected for report
- sponsor confirmed
- M-07

# [Last collateral check is not safe](https://github.com/code-423n4/2022-12-backed-findings/issues/216) 

# Lines of code

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L273


# Vulnerability details

## Impact

Liquidation might work incorrectly

## Proof of Concept

There is a function `purchaseLiquidationAuctionNFT()` to allow liquidators to purchase NFTs on auction.
In the line 273, the protocol checks if the current NFT is the last collateral using the `collateralValueCached`.
But it might be possible for Reservoir Oracle to return zero (for whatever reason) and in that case `collateralValueCached` will be zero even when the `_vaultInfo[auction.nftOwner][auction.auctionAssetContract].count!=0`.
One might argue that it is impossible for the Reservoir oracle to return zero output but I think it is safe not to rely on it.

```solidity
PaprController.sol
264:     function purchaseLiquidationAuctionNFT(
265:         Auction calldata auction,
266:         uint256 maxPrice,
267:         address sendTo,
268:         ReservoirOracleUnderwriter.OracleInfo calldata oracleInfo
269:     ) external override {
270:         uint256 collateralValueCached = underwritePriceForCollateral(
271:             auction.auctionAssetContract, ReservoirOracleUnderwriter.PriceKind.TWAP, oracleInfo
272:         ) * _vaultInfo[auction.nftOwner][auction.auctionAssetContract].count;
273:         bool isLastCollateral = collateralValueCached == 0;//@audit not safe
274:
275:         uint256 debtCached = _vaultInfo[auction.nftOwner][auction.auctionAssetContract].debt;
276:         uint256 maxDebtCached = isLastCollateral ? debtCached : _maxDebt(collateralValueCached, updateTarget());
277:         /// anything above what is needed to bring this vault under maxDebt is considered excess
278:         uint256 neededToSaveVault = maxDebtCached > debtCached ? 0 : debtCached - maxDebtCached;
279:         uint256 price = _purchaseNFTAndUpdateVaultIfNeeded(auction, maxPrice, sendTo);time
280:         uint256 excess = price > neededToSaveVault ? price - neededToSaveVault : 0;
281:         uint256 remaining;
282:
283:         if (excess > 0) {
284:             remaining = _handleExcess(excess, neededToSaveVault, debtCached, auction);
285:         } else {
286:             _reduceDebt(auction.nftOwner, auction.auctionAssetContract, address(this), price);
287:             remaining = debtCached - price;
288:         }
289:
290:         if (isLastCollateral && remaining != 0) {
291:             /// there will be debt left with no NFTs, set it to 0
292:             _reduceDebtWithoutBurn(auction.nftOwner, auction.auctionAssetContract, remaining);
293:         }
294:     }
295:

```

## Tools Used

Manual Review

## Recommended Mitigation Steps

Change the line 273 as below.

```soliditiy
bool isLastCollateral = _vaultInfo[auction.nftOwner][auction.auctionAssetContract].count == 0;
```