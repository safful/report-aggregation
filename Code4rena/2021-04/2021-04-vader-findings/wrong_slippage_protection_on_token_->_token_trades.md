## Tags

- bug
- disagree with severity
- 3 (High Risk)
- sponsor confirmed
- filed

# [Wrong slippage protection on Token -> Token trades](https://github.com/code-423n4/2021-04-vader-findings/issues/209) 

# Handle

@cmichelio


# Vulnerability details

## Vulnerability Details

The `Router.swapWithSynthsWithLimit` allows trading token to token and specifying slippage protection.
A token to token trade consists of two trades:

1. token to base
2. base to token

The slippage protection of the second trade (base to token) is computed wrong:

```solidity
require(iUTILS(UTILS()).calcSwapSlip(
        inputAmount, // should use outToken here from prev trade
        iPOOLS(POOLS).getBaseAmount(outputToken)
  ) <= slipLimit
);
```

It compares the **token** input amount (of the first trade) to the **base** reserve of the second pair.

## Impact

Slippage protection fails and either the trade is cancelled when it shouldn't be or it is accepted even though the user suffered more losses than expected.

## Recommended Mitigation Steps

It should use the base output from the first trade to check for slippage protection. Note that this still just computes the slippage protection of each trade individually. An even better way would be to come up with a formula to compute the slippage on the two trades at once.


