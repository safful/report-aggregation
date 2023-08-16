## Tags

- bug
- 1 (Low Risk)
- sponsor acknowledged
- sponsor confirmed
- disagree with severity

# [`getMostPremium()` can be wrong](https://github.com/code-423n4/2021-09-yaxis-findings/issues/139) 

# Handle

0xsanson


# Vulnerability details

## Impact
`NativeStrategyCurve3Crv._harvest` calls `getMostPremium` to get the best stablecoin to convert to. This function however is wrong in the case of `balancesUSDC = balancesUSDT < balancesDAI`, because it returns DAI, when it should be USDC or USDT.
This is naturally a rare occasion, but a bad actor can set the balances (by depositing/withdrawing the Curve pool) like this just before the harvest function is called. Since this would imbalance even more the pool, the bad actor could also gain a profit by making the right swaps after the harvest.

## Proof of Concept
https://github.com/code-423n4/2021-09-yaxis/blob/main/contracts/v3/strategies/NativeStrategyCurve3Crv.sol#L83

## Tools Used
editor

## Recommended Mitigation Steps
Convert all `<` into `<=` inside `getMostPremium()`.

