## Tags

- bug
- 3 (High Risk)
- primary issue
- satisfactory
- selected for report
- sponsor confirmed
- edited-by-warden
- H-02

# [Basket range formula is inefficient, leading the protocol to unnecessary haircut](https://github.com/code-423n4/2023-01-reserve-findings/issues/235) 

# Lines of code

https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/mixins/RecollateralizationLib.sol#L152-L202


# Vulnerability details



## Impact
The `BackingManager.manageTokens()` function checks if there's any deficit in collateral, in case there is, if there's a surplus from another collateral token it trades it to cover the deficit, otherwise it goes for a 'haircut' and cuts the amount of basket 'needed' (i.e. the number of baskets RToken claims to hold).
In order to determine how much deficit/surplus there is the protocol calculates the 'basket range', where the top range is the optimistic estimation of the number of baskets the token would hold after trading and the bottom range is a pessimistic estimation.

The estimation is done by dividing the total collateral value by the price of 1  basket unit (for optimistic estimation the max value is divided by min price of basket-unit and vice versa).
The problem is that this estimation is inefficient, for cases where just a little bit of collateral is missing the range 'band' (range.top - range.bottom) would be about 4% (when oracle error deviation is ±1%) instead of less than 1%.

This can cause the protocol an unnecessary haircut of a few percent where the deficit can be solved by simple trading.

This would also cause the price of `RTokenAsset` to deviate more than necessary before the haircut.


## Proof of Concept
In the following PoC, the basket changed so that it has 99% of the required collateral for 3 tokens and 95% for the 4th.
The basket range should be 98±0.03% (the basket has 95% collateral + 4% of 3/4 tokens. That 4% is worth 3±0.03% if we account for oracle error of their prices), but in reality the protocol calculates it as ~97.9±2%.
That range causes the protocol to avoid trading and go to an unnecessary haircut to ~95%


```diff
diff --git a/contracts/plugins/assets/RTokenAsset.sol b/contracts/plugins/assets/RTokenAsset.sol
index 62223442..03d3c3f4 100644
--- a/contracts/plugins/assets/RTokenAsset.sol
+++ b/contracts/plugins/assets/RTokenAsset.sol
@@ -123,7 +123,7 @@ contract RTokenAsset is IAsset {
     // ==== Private ====
 
     function basketRange()
-        private
+        public
         view
         returns (RecollateralizationLibP1.BasketRange memory range)
     {
diff --git a/test/Recollateralization.test.ts b/test/Recollateralization.test.ts
index 3c53fa30..386c0673 100644
--- a/test/Recollateralization.test.ts
+++ b/test/Recollateralization.test.ts
@@ -234,7 +234,42 @@ describe(`Recollateralization - P${IMPLEMENTATION}`, () => {
         // Issue rTokens
         await rToken.connect(addr1)['issue(uint256)'](issueAmount)
       })
+      it('PoC basket range', async () => {
+        let range = await rTokenAsset.basketRange();
+        let basketTokens = await basketHandler.basketTokens();
+        console.log({range}, {basketTokens});
+        // Change the basket so that current balance would be 99 or 95 percent of
+        // the new basket
+        let q99PercentLess = 0.25 / 0.99;
+        let q95ercentLess = 0.25 / 0.95;
+        await basketHandler.connect(owner).setPrimeBasket(basketTokens, [fp(q99PercentLess),fp(q99PercentLess), fp(q95ercentLess), fp(q99PercentLess)])
+        await expect(basketHandler.connect(owner).refreshBasket())
+        .to.emit(basketHandler, 'BasketSet')
+
+        expect(await basketHandler.status()).to.equal(CollateralStatus.SOUND)
+        expect(await basketHandler.fullyCollateralized()).to.equal(false)
+
+        range = await rTokenAsset.basketRange();
+
+        // show the basket range is 95.9 to 99.9
+        console.log({range});
 
+        let needed = await rToken.basketsNeeded();
+
+        // show that prices are more or less the same
+        let prices = await Promise.all( basket.map(x => x.price()));
+
+        // Protocol would do a haircut even though it can easily do a trade
+        await backingManager.manageTokens([]);
+
+        // show how many baskets are left after the haircut
+         needed = await rToken.basketsNeeded();
+         
+        console.log({prices, needed});
+        return;
+    
+      })
+      return;
       it('Should select backup config correctly - Single backup token', async () => {
         // Register Collateral
         await assetRegistry.connect(owner).register(backupCollateral1.address)
@@ -602,7 +637,7 @@ describe(`Recollateralization - P${IMPLEMENTATION}`, () => {
         expect(quotes).to.eql([initialQuotes[0], initialQuotes[1], initialQuotes[3], bn('0.25e18')])
       })
     })
-
+    return;
     context('With multiple targets', function () {
       let issueAmount: BigNumber
       let newEURCollateral: FiatCollateral
@@ -785,7 +820,7 @@ describe(`Recollateralization - P${IMPLEMENTATION}`, () => {
       })
     })
   })
-
+  return;
   describe('Recollateralization', function () {
     context('With very simple Basket - Single stablecoin', function () {
       let issueAmount: BigNumber

```

Output (comments are added by me):

```
{
  range: [
    top: BigNumber { value: "99947916501440267201" },  //  99.9 basket units
    bottom: BigNumber { value: "95969983506382791000" } // 95.9 basket units
  ]
}
{
  prices: [
    [
      BigNumber { value: "990000000000000000" },
      BigNumber { value: "1010000000000000000" }
    ],
    [
      BigNumber { value: "990000000000000000" },
      BigNumber { value: "1010000000000000000" }
    ],
    [
      BigNumber { value: "990000000000000000" },
      BigNumber { value: "1010000000000000000" }
    ],
    [
      BigNumber { value: "19800000000000000" },
      BigNumber { value: "20200000000000000" }
    ]
  ],
  needed: BigNumber { value: "94999999905000000094" } // basket units after haircut: 94.9
}
```

## Recommended Mitigation Steps
Change the formula so that we first calculate the 'base' (i.e. the min amount of baskets the RToken can satisfy without trading):
```
base = basketsHeldBy(backingManager) // in the PoC's case it'd be 95
(diffLowValue, diffHighValue) = (0,0) 
for each collateral token:
    diff = collateralBalance - basketHandler.quantity(base) 
    (diffLowValue, diffHighValue) = diff * (priceLow, priceHigh)
addBasketsLow = diffLowValue / basketPriceHigh
addBasketHigh = diffHighValue / basketPriceLow
range.top = base + addBasketHigh
range.bottom = base + addBasketLow
```