## Tags

- bug
- 2 (Med Risk)
- downgraded by judge
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- M-08

# [user fund loss because function purchaseLiquidationAuctionNFT() take extra liquidation penalty when user's last collateral is liquidated, (set wrong value for maxDebtCached when isLastCollateral is true)](https://github.com/code-423n4/2022-12-backed-findings/issues/255) 

# Lines of code

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L264-L294


# Vulnerability details

## Impact
Function `purchaseLiquidationAuctionNFT()` purchases a liquidation auction with the controller's papr token. the liquidator pays the papr amount which is equal to price of the auction and receives the auctioned  NFT. contract would transfer paid papr and  pay borrower debt and if there is extra papr left it would be transferred to the user. for extra papr that is not required for brining user debt under max debt, contract gets liquidation penalty but in some cases (when the auctioned NFT is user's last collateral) contract take penalty from all of the transferred papr and not just the extra. so users would lose funds in those situations because of this and the fund could be big because the penalty is 10% of the price of the auction and in most cases user would lose 10% of his debt (the value of the NFT).

## Proof of Concept
This is `purchaseLiquidationAuctionNFT()` code:
```
    function purchaseLiquidationAuctionNFT(
        Auction calldata auction,
        uint256 maxPrice,
        address sendTo,
        ReservoirOracleUnderwriter.OracleInfo calldata oracleInfo
    ) external override {
        uint256 collateralValueCached = underwritePriceForCollateral(
            auction.auctionAssetContract, ReservoirOracleUnderwriter.PriceKind.TWAP, oracleInfo
        ) * _vaultInfo[auction.nftOwner][auction.auctionAssetContract].count;
        bool isLastCollateral = collateralValueCached == 0;

        uint256 debtCached = _vaultInfo[auction.nftOwner][auction.auctionAssetContract].debt;
        uint256 maxDebtCached = isLastCollateral ? debtCached : _maxDebt(collateralValueCached, updateTarget());
        /// anything above what is needed to bring this vault under maxDebt is considered excess
        uint256 neededToSaveVault = maxDebtCached > debtCached ? 0 : debtCached - maxDebtCached;
        uint256 price = _purchaseNFTAndUpdateVaultIfNeeded(auction, maxPrice, sendTo);
        uint256 excess = price > neededToSaveVault ? price - neededToSaveVault : 0;
        uint256 remaining;

        if (excess > 0) {
            remaining = _handleExcess(excess, neededToSaveVault, debtCached, auction);
        } else {
            _reduceDebt(auction.nftOwner, auction.auctionAssetContract, address(this), price);
            remaining = debtCached - price;
        }

        if (isLastCollateral && remaining != 0) {
            /// there will be debt left with no NFTs, set it to 0
            _reduceDebtWithoutBurn(auction.nftOwner, auction.auctionAssetContract, remaining);
        }
    }
```
As you can see when `collateralValueCached` is 0 and user has no more collaterals left then the value of `isLastCollateral` set as true. and when `isLastCollateral` is true the value of `maxDebtCached` set as `debtCached` (line `maxDebtCached = isLastCollateral ? debtCached : _maxDebt(collateralValueCached, updateTarget());`) and the value of the `neededToSaveVault` would be 0 (line `neededToSaveVault = maxDebtCached > debtCached ? 0 : debtCached - maxDebtCached`) and the `excess` would be equal to `price` (in the line `excess = price > neededToSaveVault ? price - neededToSaveVault : 0`) so all the papr paid by liquidator would considered as excess and contract would get liquidation penalty out of that. so: in current implementation in last collateral liquidation all of the paid papr by liquidator would be considered excess:
1. user has no NFT left.
2. debtCached is 100.
3. collateralValueCached  is 0 and isLastCollateral is true.
4. maxDebtCached would be as debtCached which is 100.
5. neededToSaveVault would be debtCached - maxDebtCached which is 0.
6. excess would equal to price and code would take penalty out of all the price amount.

code wants to take penalty from what borrower is going to receive(other than the required amount for extra debt), but in the current implementation when it is last NFT code took fee from all of the payment. these are the steps shows how issue would harm the borrower and borrower would lose funds: (of course user debt would be set to 0 in the end, but if price was higher than user debt user won't receive the extra amount)
1. user debt is 900 and price of auction is 1000 and user has no NFT left.
2. some one pays 1000 Papr and buys the auctioned token, now user would receive 0 amount because the penalty would be 1000 * 10% = 100 and the debt is 900.
3. but penalty should be (1000-900) * 10% = 10 and user should have received 90 token.

so users would receive less amount when their last NFT is liquidated and the price is higher than debt. users would lose 10% of the their intitled fund. most users can use one token as collateral so the bug can happen most of the time.

## Tools Used
VIM

## Recommended Mitigation Steps
the code should be like this:
```
uint256 maxDebtCached = isLastCollateral ? 0: _maxDebt(collateralValueCached, updateTarget());
```