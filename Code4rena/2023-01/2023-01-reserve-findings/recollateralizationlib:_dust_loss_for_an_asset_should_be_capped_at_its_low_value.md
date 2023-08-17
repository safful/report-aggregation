## Tags

- bug
- 2 (Med Risk)
- judge review requested
- satisfactory
- selected for report
- sponsor confirmed
- edited-by-warden
- M-22

# [RecollateralizationLib: Dust loss for an asset should be capped at its low value](https://github.com/code-423n4/2023-01-reserve-findings/issues/106) 

# Lines of code

https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/mixins/RecollateralizationLib.sol#L275


# Vulnerability details

## Impact
The `RecollateralizationLib.basketRange` function ([https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/mixins/RecollateralizationLib.sol#L152-L202](https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/mixins/RecollateralizationLib.sol#L152-L202)) internally calls the `RecollateralizationLib.totalAssetValue` function ([https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/mixins/RecollateralizationLib.sol#L226-L281](https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/mixins/RecollateralizationLib.sol#L226-L281)).  

I will show in this report that the `RecollateralizationLib.totalAssetValue` function returns a value for `assetsLow` that is too low.  

This in turn causes the `range.bottom` value ([https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/mixins/RecollateralizationLib.sol#L201](https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/mixins/RecollateralizationLib.sol#L201)) that the `RecollateralizationLib.basketRange` function returns to be too low.  

Before showing why the `assetsLow` value is underestimated however I will explain the impact of the `range.bottom` variable being too low.  

There are two places where this value is used:  

### 1. `RecollateralizationLib.prepareRecollateralizationTrade` function 
This function passes the `range` to the `RecollateralizationLib.nextTradePair` function ([https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/mixins/RecollateralizationLib.sol#L88-L91](https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/mixins/RecollateralizationLib.sol#L88-L91))

Since `range.bottom` is too low, the `needed` amount is too low ([https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/mixins/RecollateralizationLib.sol#L380](https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/mixins/RecollateralizationLib.sol#L380)).  

This causes the `if` statement to not be executed in some cases when it otherwise would be executed ([https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/mixins/RecollateralizationLib.sol#L381-L396](https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/mixins/RecollateralizationLib.sol#L381-L396)).  

And the `amtShort` is smaller than it should be ([https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/mixins/RecollateralizationLib.sol#L391](https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/mixins/RecollateralizationLib.sol#L391)).  

In the end this causes recollateralization trades to not buy as much assets as they could buy. This is because the amount of assets is underestimated so the protocol can actually hold more baskets than it thinks it can.  

Therefore underestimating `assetsLow` causes a direct loss to RToken holders because the protocol will not recollateralize the RToken to the level that it can and should.  

### 2. Price calculations of `RTokenAsset`
A `RTokenAsset` uses the `RecollateralizationLib.basketRange` function to calculate its value:  

[https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/plugins/assets/RTokenAsset.sol#L156](https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/plugins/assets/RTokenAsset.sol#L156)  

The `RTokenAsset` therefore underestimates its `low` and `lotLow` prices:  

[https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/plugins/assets/RTokenAsset.sol#L58](https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/plugins/assets/RTokenAsset.sol#L58)  

[https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/plugins/assets/RTokenAsset.sol#L99](https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/plugins/assets/RTokenAsset.sol#L99)   

This then can lead to issues in any places where the prices of `RTokenAsset`s are used.  

## Proof of Concept
Here is the affected line:  

[https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/mixins/RecollateralizationLib.sol#L275](https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/mixins/RecollateralizationLib.sol#L275)  

```solidity
potentialDustLoss = potentialDustLoss.plus(rules.minTradeVolume);
```

This line is executed for every asset in the `AssetRegistry` ([https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/mixins/RecollateralizationLib.sol#L242](https://github.com/reserve-protocol/protocol/blob/df7ecadc2bae74244ace5e8b39e94bc992903158/contracts/p1/mixins/RecollateralizationLib.sol#L242)).  

So for every asset in the `AssetRegistry` a potential dust loss of `minTradeVolume` is added.  

The following scenario shows why this is wrong:  

```
assume minTradeVolume = $50

assume further the following:
asset1 with low value $1
asset2 with low value $1
asset3 with low value $1
asset4 with low value $200

Currently potentialDustLoss will be 4*minTradeVolume = $200.
So assetsLow = $203 - $200 = $3.

Dust loss should not be calculated with $50 for the first 3 assets.
Dust loss for an asset should be capped at its low value.
So dust loss alltogether should be $1 + $1 + $1 + $50 = $53.

So assetsLow should be $1+$1+$1+$200 - $53 = $150.
```

## Tools Used
VSCode

## Recommended Mitigation Steps
I suggest that an asset can only incur as much dust loss as its balance is.  
If the protocol only holds `$5` of asset A then this should not cause a dust loss of say `$10`.  

The fix first saves the `assetLow` value which should be saved to memory because it is now needed two times then it caps the dust loss of an asset at its low value:  

```
diff --git a/contracts/p1/mixins/RecollateralizationLib.sol b/contracts/p1/mixins/RecollateralizationLib.sol
index 648d1813..b5b86cac 100644
--- a/contracts/p1/mixins/RecollateralizationLib.sol
+++ b/contracts/p1/mixins/RecollateralizationLib.sol
@@ -261,7 +261,8 @@ library RecollateralizationLibP1 {
 
             // Intentionally include value of IFFY/DISABLED collateral when low is nonzero
             // {UoA} = {UoA} + {UoA/tok} * {tok}
-            assetsLow += low.mul(bal, FLOOR);
+            uint192 assetLow = low.mul(bal,FLOOR);
+            assetsLow += assetLow;
             // += is same as Fix.plus
 
             // assetsHigh += high.mul(bal, CEIL), where assetsHigh is [0, FIX_MAX]
@@ -272,7 +273,7 @@ library RecollateralizationLibP1 {
             // += is same as Fix.plus
 
             // Accumulate potential losses to dust
-            potentialDustLoss = potentialDustLoss.plus(rules.minTradeVolume);
+            potentialDustLoss = potentialDustLoss.plus(fixMin(rules.minTradeVolume, assetLow));
         }
 
         // Account for all the places dust could get stuck
```


