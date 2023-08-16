## Tags

- bug
- 3 (High Risk)
- sponsor confirmed

# [Swaps are not split when trade crosses target price](https://github.com/code-423n4/2021-11-bootfinance-findings/issues/216) 

# Handle

cmichel


# Vulnerability details

The protocol uses two amplifier values A1 and A2 for the swap, depending on the target price, see `SwapUtils.determineA`.
The swap curve is therefore a join of two different curves at the target price.
When doing a trade that crosses the target price, it should first perform the trade partially with A1 up to the target price, and then the rest of the trade order with A2.

However, the `SwapUtils.swap / _calculateSwap` function does not do this, it only uses the "new A", see `getYC` step 5.

```solidity
// 5. Check if we switched A's during the swap
if (aNew == a){     // We have used the correct A
    return y;
} else {    // We have switched A's, do it again with the new A
    return getY(self, tokenIndexFrom, tokenIndexTo, x, xp, aNew, d);
}
```

## Impact
Trades that cross the target price and would lead to a new amplifier being used are not split up and use the new amplifier for the _entire trade_.
This can lead to a worse (better) average execution price than manually splitting the trade into two transactions, first up to but below the target price, and a second one with the rest of the trader order size, using both A1 and A2 values.

In the worst case, it could even be possible to make the entire trade with one amplifier and then sell the swap result again using the other amplifier making a profit.

## Recommended Mitigation Steps
Trades that lead to a change in amplifier value need to be split up into two trades using both amplifiers to correctly calculate the swap result.


