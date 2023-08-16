## Tags

- bug
- sponsor confirmed
- 1 (Low Risk)
- resolved

# [No `swap` slippage checks](https://github.com/code-423n4/2021-10-tempus-findings/issues/23) 

# Handle

cmichel


# Vulnerability details

The (second) `TempusController._exitTempusAmmAndRedeem` function swaps the difference of yield and principal shares using the AMM.

```solidity
swap(
    tempusAMM,
    tempusAMM.getSwapAmountToEndWithEqualShares(principals, yields, maxLeftoverShares),
    tokenIn,
    tokenOut,
    0 // @audit min return of zero
);
// yields and principals are updated to the received amounts and redeemed
// ...
```

It does not use a min return value for this swap and it is, therefore, susceptible to sandwich attacks.

> A common attack in DeFi is the sandwich attack. Upon observing a trade of asset X for asset Y, an attacker frontruns the victim trade by also buying asset Y, lets the victim execute the trade, and then backruns (executes after) the victim by trading back the amount gained in the first trade. Intuitively, one uses the knowledge that someone’s going to buy an asset, and that this trade will increase its price, to make a profit. The attacker’s plan is to buy this asset cheap, let the victim buy at an increased price, and then sell the received amount again at a higher price afterwards.

## Impact
Trades can happen at a bad price and lead to receiving fewer tokens than at a fair market price.
The attacker's profit is the user's loss.

## Recommended Mitigation Steps
Add minimum return amount checks.
Accept a function parameter that can be chosen by the transaction sender, then check that the actually received amount is above this parameter.
Similar to `minLpAmountsOut` but for the yields & principal shares (or the redeemed tokens).

