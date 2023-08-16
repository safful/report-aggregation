## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- disagree with severity

# [Dividend reward can be gamed](https://github.com/code-423n4/2021-07-spartan-findings/issues/182) 

# Handle

cmichel


# Vulnerability details

The `Router.addDividend` function tells the reserve to send dividends to the pool depending on the fees.

- The attacker provides LP to a curated pool. Ideally, they become a large LP holder to capture most of the profit, they should choose the smallest liquidity pool as the dividends are pool-independent.
- The `normalAverageFee` variable that determines the pool dividends can be set to zero by the attacker by trading a single wei in the pool `arrayFeeSize` (20) times (use `buyTo`). The fees of the single wei trades will be zero and thus the `normalAverageFee` will also be zero as, see `addTradeFee`.
- The attacker then does a trade that generates some non-zero fees, setting the `normalAverageFee` to this trade's fee. The `feeDividend` is then computed as `_fees * dailyAllocation / (_fees + normalAverageFee) = _fees * dailyAllocation / (2 * _fees) = dailyAllocation / 2`. Half of the `dailyAllocation` is sent to the pool.
- The attacker repeats the above steps until the reserve is almost empty. Each time the `dailyAllocation` gets smaller but it's still possible to withdraw almost all of it.
- They redeem their LP tokens and gain a share of the profits

## Impact
The reserve can be emptied by the attacker.

## Recommended Mitigation Steps
Counting only the last 20 trades as a baseline for the dividends does not work. It should probably average over a timespan but even that can be gamed if it is too short.
I think a better idea is to compute the dividends based on **volume** traded over a timespan instead of looking at individual trades.


