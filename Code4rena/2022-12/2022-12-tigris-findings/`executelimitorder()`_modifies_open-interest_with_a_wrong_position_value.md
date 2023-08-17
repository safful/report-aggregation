## Tags

- bug
- 2 (Med Risk)
- primary issue
- selected for report
- sponsor confirmed
- M-21

# [`executeLimitOrder()` modifies open-interest with a wrong position value](https://github.com/code-423n4/2022-12-tigris-findings/issues/576) 

# Lines of code

https://github.com/code-423n4/2022-12-tigris/blob/496e1974ee3838be8759e7b4096dbee1b8795593/contracts/Trading.sol#L513-L517


# Vulnerability details


The `PairsContract` registeres the total long/short position that's open for a pair of assets, whenever a new position is created the total grows accordingly.
However at `executeLimitOrder()` the position size that's added is wrongly calculated - it uses margin before fees, while the actual position is created after subtracting fees.

## Impact
The OpenInterest would register wrong values (11% diff in the case of PoC), which will distort the balance between long and short positions (the whole point of the OpenInterest is to balance them to be about equal).


## Proof of Concept
In the following test, an order is created with a x100 leverage, and the position size registered for OI is 11% greater than the actual position created.


```diff
diff --git a/test/07.Trading.js b/test/07.Trading.js
index ebe9948..dfb7f98 100644
--- a/test/07.Trading.js
+++ b/test/07.Trading.js
@@ -778,7 +778,7 @@ describe("Trading", function () {
      */
     it("Creating and executing limit buy order, should have correct price and bot fees", async function () {
       // Create limit order
-      let TradeInfo = [parseEther("1000"), MockDAI.address, StableVault.address, parseEther("10"), 0, true, parseEther("0"), parseEther("0"), ethers.constants.HashZero];
+      let TradeInfo = [parseEther("1000"), MockDAI.address, StableVault.address, parseEther("100"), 0, true, parseEther("0"), parseEther("0"), ethers.constants.HashZero];
       let PermitData = [permitSig.deadline, ethers.constants.MaxUint256, permitSig.v, permitSig.r, permitSig.s, true];
       await trading.connect(owner).initiateLimitOrder(TradeInfo, 1, parseEther("20000"), PermitData, owner.address);
       expect(await position.limitOrdersLength(0)).to.equal(1); // Limit order opened
@@ -787,6 +787,9 @@ describe("Trading", function () {
       await network.provider.send("evm_increaseTime", [10]);
       await network.provider.send("evm_mine");
 
+      let count = await position.getCount();
+      let id = count.toNumber() - 1;
+
       // Execute limit order
       let PriceData = [node.address, 0, parseEther("10000"), 10000000, 2000000000, false]; // 0.1% spread
       let message = ethers.utils.keccak256(
@@ -798,8 +801,22 @@ describe("Trading", function () {
       let sig = await node.signMessage(
         Buffer.from(message.substring(2), 'hex')
       );
+      // trading.connect(owner).setFees(true,3e8,1e8,1e8,1e8,1e8);
       
-      await trading.connect(user).executeLimitOrder(1, PriceData, sig);
+
+      let oi = await pairscontract.idToOi(0, stabletoken.address);
+      expect(oi.longOi.toNumber()).to.equal(0);
+      console.log({oi, stable:stabletoken.address});
+
+      await trading.connect(user).executeLimitOrder(id, PriceData, sig);
+      let trade = await position.trades(id);
+      console.log(trade);
+      oi = await pairscontract.idToOi(0, stabletoken.address);
+      console.log(oi);
+
+      expect(oi.longOi.div(10n**18n).toNumber()).to.equal(trade.margin.mul(trade.leverage).div(10n**18n * 10n**18n).toNumber());
+
+
       expect(await position.limitOrdersLength(0)).to.equal(0); // Limit order executed
       expect(await position.assetOpenPositionsLength(0)).to.equal(1); // Creates open position
       expect((await trading.openFees()).botFees).to.equal(2000000);
@@ -807,6 +824,7 @@ describe("Trading", function () {
       let [,,,,price,,,,,,,] = await position.trades(1);
       expect(price).to.equal(parseEther("20020")); // Should have guaranteed execution price with spread
     });
+    return;
     it("Creating and executing limit sell order, should have correct price and bot fees", async function () {
       // Create limit order
       let TradeInfo = [parseEther("1000"), MockDAI.address, StableVault.address, parseEther("10"), 0, false, parseEther("0"), parseEther("0"), ethers.constants.HashZero];
@@ -1606,6 +1624,7 @@ describe("Trading", function () {
       expect(await stabletoken.balanceOf(user.address)).to.equal(parseEther("1.5"));
     });
   });
+  return;
   describe("Modifying functions", function () {
     it("Updating TP/SL on a limit order should revert", async function () {
       let TradeInfo = [parseEther("1000"), MockDAI.address, StableVault.address, parseEther("10"), 0, true, parseEther("0"), parseEther("0"), ethers.constants.HashZero];

```

Output:
```
1) Trading
       Limit orders and liquidations
         Creating and executing limit buy order, should have correct price and bot fees:

      AssertionError: expected 100000 to equal 90000
      + expected - actual

      -100000
      +90000
```


## Recommended Mitigation Steps
Correct the calculation to use margin after fees.