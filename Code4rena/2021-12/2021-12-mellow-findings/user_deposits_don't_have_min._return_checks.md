## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [User deposits don't have min. return checks](https://github.com/code-423n4/2021-12-mellow-findings/issues/46) 

# Handle

cmichel


# Vulnerability details

The `LPIssuer.deposit` first computes _balanced amounts_ on the user's defined `tokenAmounts`.
The idea is that LP tokens give the same percentage share of each vault tokens' tvl, therefore the provided amounts should be _balanced_, meaning, the `depositAmount / tvl` ratio should be equal for all vault tokens.

But the strategist can frontrun the user's deposit and rebalance the vault tokens, changing the tvl for each vault token which changes the rebalance.
This frontrun can happen accidentally whenever the strategist rebalances

## POC
There's a vault with two tokens A and B, tvls are `[500, 1500]`

- The user provides `[500, 1500]`, expecting to get 50% of the share supply (is minted 100% of old total supply).
- The strategist rebalances to `[1000, 1000]`
- The user's balanceFactor is `min(500/1000, 1500/1000) = 1/2`, their balancedAmounts are thus `tvl * balanceFactor = [500, 500]`, the `1000` excess token B are refunded. In the end, they only received `500/(1000+500) = 33.3%` of the total supply but used up all of their token A which they might have wanted to hold on to if they had known they'd only get 33.3% of the supply.

## Impact
Users can get rekt when depositing as the received LP amount is unpredictable and lead to a trade using a very different balanced token mix that they never intended.

## Recommended Mitigation Steps
Add minimum return amount checks.
Accept a function parameter that can be chosen by the user indicating their _expected LP amount_ for their deposit `tokenAmounts`, then check that the actually minted LP token amount is above this parameter.


