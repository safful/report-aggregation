## Tags

- bug
- 3 (High Risk)
- sponsor confirmed
- disagree with severity

# [SynthVault rewards can be gamed](https://github.com/code-423n4/2021-07-spartan-findings/issues/166) 

# Handle

cmichel


# Vulnerability details

## Vulnerability Details

The `SynthVault._deposit` function adds `weight` for the user that depends on the spot value of the deposit synth amount in `BASE`.
This spot price can be manipulated and the cost of manipulation is relative to the pool's liquidity.
However, the reward (see `calcReward`) is measured in BASE tokens unrelated to the pool.
Therefore, if the pool's liquidity is low and the reward reserve is high, the attack can be profitable:

1. Manipulate the pool spot price of the `iSYNTH(_synth).LayerONE()` pool by dripping a lot of `BASE` into it repeatedly (sending lots of smaller trades is less costly due to the [path-independence of the continuous liquidity model](https://docs.thorchain.org/thorchain-finance/continuous-liquidity-pools)). This increases the `BASE` per `token` price.
2. Call `SynthVault.depositForMember` and deposit a _small_ amount of synth token. The `iUTILS(_DAO().UTILS()).calcSpotValueInBase(iSYNTH(_synth).LayerONE(), _amount)` will return an inflated weight due to the price.
3. Optionally drip more `BASE` into the pool and repeat the deposits
4. Drip back `token` to the pool to rebalance it

The user's `weight` is now inflated compared to the deposited / locked-up amount and they can claim a large share of the rewards.

## Impact
The cost of the attack depends on the pool's liquidity and the profit depends on the reserve.
It could therefore be profitable under certain circumstances.

## Recommended Mitigation Steps
Track a TWAP price of the synth instead, store the deposited synths instead, and compute the weight & total weight on the fly based on the TWAP * deposit amount instead of at the time of deposit.

