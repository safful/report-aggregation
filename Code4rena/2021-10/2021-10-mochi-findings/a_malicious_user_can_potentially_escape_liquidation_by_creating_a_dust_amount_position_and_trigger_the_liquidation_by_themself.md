## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [A malicious user can potentially escape liquidation by creating a dust amount position and trigger the liquidation by themself](https://github.com/code-423n4/2021-10-mochi-findings/issues/127) 

# Handle

WatchPug


# Vulnerability details

In the current implementation, a liquidated position can be used for depositing and borrowing again.

However, if there is a liquidation auction ongoing, even if the position is now `liquidatable`, the call of `triggerLiquidation()` will still fail. 

The liquidator must `settleLiquidation` first.

If the current auction is not profitable for the liquidator, say the value of the collateral can not even cover the gas cost, the liquidator may be tricked and not liquidate the new loan at all.

Considering if the liquidator bot is not as small to handle this situation (take the profit of the new liquidation and the gas cost loss of the current auction into consideration), a malicious user can create a dust amount position trigger the liquidation by themself.

Since the collateral of this position is so small that it can not even cover the gas cost, liquidators will most certainly ignore this auction.

The malicious user will then deposit borrow the actual loan.

When this loan becomes `liquidatable`, liquidators may:

1. confuse the current dust auction with the `liquidatable` position;
2. unable to proceed with such a complex liquidation.

As a result, the malicious user can potentially escape liquidation.

### Recommendation

Consider making liquidated positions unable to be used (for depositing and borrowing) again.

