## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- fixed-in-upstream-repo

# [LongShort.sol: Cache marketUpdateIndex[marketIndex]](https://github.com/code-423n4/2021-08-floatcapital-findings/issues/58) 

# Handle

hickuphh3


# Vulnerability details

### Impact

By storing `marketUpdateIndex[marketIndex];` locally in `_updateSystemStateInternal()`, multiple sLOADs can be avoided.

### Recommended Mitigation Steps

Gas savings of about 500-600 is achieved.

```jsx
function _updateSystemStateInternal(uint32 marketIndex) internal virtual requireMarketExists(marketIndex) {
	...
	// cache marketUpdateIndex[marketIndex]
  uint256 currentMarketIndex = marketUpdateIndex[marketIndex];
	
	bool assetPriceHasChanged = oldAssetPrice != newAssetPrice;

	if (assetPriceHasChanged || msg.sender == staker) {
		uint256 syntheticTokenPrice_inPaymentTokens_long = syntheticToken_priceSnapshot[marketIndex][true][
			currentMarketIndex
    ];
    uint256 syntheticTokenPrice_inPaymentTokens_short = syntheticToken_priceSnapshot[marketIndex][false][
      currentMarketIndex
    ];

		if (
      userNextPrice_currentUpdateIndex[marketIndex][staker] == currentMarketIndex + 1 &&
      assetPriceHasChanged
    ) {
      IStaker(staker).pushUpdatedMarketPricesToUpdateFloatIssuanceCalculations(
        marketIndex,
        syntheticTokenPrice_inPaymentTokens_long,
        syntheticTokenPrice_inPaymentTokens_short,
        marketSideValueInPaymentToken[marketIndex][true],
        marketSideValueInPaymentToken[marketIndex][false],
        // This variable could allow users to do any next price actions in the future (not just synthetic side shifts)
        currentMarketIndex + 1
      );
    } else {
      IStaker(staker).pushUpdatedMarketPricesToUpdateFloatIssuanceCalculations(
        marketIndex,
        syntheticTokenPrice_inPaymentTokens_long,
        syntheticTokenPrice_inPaymentTokens_short,
        marketSideValueInPaymentToken[marketIndex][true],
        marketSideValueInPaymentToken[marketIndex][false],
        0
      );
    }
		...
		// increment currentMarketIndex
		currentMarketIndex ++;
	  marketUpdateIndex[marketIndex] = currentMarketIndex;
		syntheticToken_priceSnapshot[marketIndex][true][
			currentMarketIndex
	  ] = syntheticTokenPrice_inPaymentTokens_long;
	
	  syntheticToken_priceSnapshot[marketIndex][false][
	    currentMarketIndex
	  ] = syntheticTokenPrice_inPaymentTokens_short;
		...
		emit SystemStateUpdated(
	    marketIndex,
	    currentMarketIndex,
			...
		);
	}
}
```

