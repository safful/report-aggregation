## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- fixed-in-upstream-repo

# [Staker.sol: Cache shift amounts](https://github.com/code-423n4/2021-08-floatcapital-findings/issues/64) 

# Handle

hickuphh3


# Vulnerability details

### Impact

The user's `userNextPrice_amountStakedSyntheticToken_toShiftAwayFrom_long` and `userNextPrice_amountStakedSyntheticToken_toShiftAwayFrom_short` are retrieved a number of times in `_calculateAccumulatedFloat()`. Caching these values would help save gas.

Note that block scoping is needed to avoid the stack too deep problem.

### Recommended Mitigation Steps

```jsx
function _calculateAccumulatedFloat() {
	// block scope for shiftAmount variable to avoid stack too deep
	{
		// Update the users balances
	  uint256 shiftAmount = userNextPrice_amountStakedSyntheticToken_toShiftAwayFrom_long[marketIndex][user];
	  if (shiftAmount > 0) {
	    amountStakedShort += ILongShort(longShort).getAmountSyntheticTokenToMintOnTargetSide(
	      marketIndex,
	      shiftAmount,
	      true,
	      stakerTokenShiftIndex_to_longShortMarketPriceSnapshotIndex_mapping[usersShiftIndex]
	    );
	
	    amountStakedLong -= shiftAmount;
	    userNextPrice_amountStakedSyntheticToken_toShiftAwayFrom_long[marketIndex][user] = 0;
	  }
	
	  shiftAmount = userNextPrice_amountStakedSyntheticToken_toShiftAwayFrom_short[marketIndex][user]
	  if (shiftAmount > 0) {
	    amountStakedLong += ILongShort(longShort).getAmountSyntheticTokenToMintOnTargetSide(
	      marketIndex,
	      shiftAmount,
	      false,
	      stakerTokenShiftIndex_to_longShortMarketPriceSnapshotIndex_mapping[usersShiftIndex]
	    );
	
	    amountStakedShort -= shiftAmount;
	    userNextPrice_amountStakedSyntheticToken_toShiftAwayFrom_short[marketIndex][user] = 0;
	  }
	}
	// end of block scoping

	// Save the users updated staked amounts
	...
}
```

