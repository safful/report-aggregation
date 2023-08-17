## Tags

- bug
- 2 (Med Risk)
- satisfactory
- selected for report
- sponsor confirmed
- M-07

# [`Mint` function does not update `LiquidityPosition` state of caller before minting LP tokens. This ](https://github.com/code-423n4/2023-01-timeswap-findings/issues/158) 

# Lines of code

https://github.com/code-423n4/2023-01-timeswap/blob/ef4c84fb8535aad8abd6b67cc45d994337ec4514/packages/v2-pool/src/structs/Pool.sol#L302
https://github.com/code-423n4/2023-01-timeswap/blob/ef4c84fb8535aad8abd6b67cc45d994337ec4514/packages/v2-pool/src/structs/LiquidityPosition.sol#L60


# Vulnerability details

## Impact
When a LP mints V2 Pool tokens, `mint` function in [PoolLibrary](https://github.com/code-423n4/2023-01-timeswap/blob/ef4c84fb8535aad8abd6b67cc45d994337ec4514/packages/v2-pool/src/structs/Pool.sol#L302) gets called. Inside this function,  `updateDurationWeightBeforeMaturity` updates global `short`, `long0` and `long1` fee growth. 

Change in global fee growth necessitates an update to `LiquidityPosition` state of caller (specifically updating fees & fee growth rates) when there are state changes made to that position (in this case, increasing liquidity). This principle is followed in functions such as `burn`, `transferLiquidity`, `transferFees`. However when calling `mint`, this update is missing. As a result, `growth` & `fee` levels in liquidity position of caller are inconsistent with global fee growth rates. 

Inconsistent state leads to incorrect calculations of long0/long1 and short fees of LP holders which inturn can lead to loss of fees. Since this impacts actual rewards for users, I've marked it as MEDIUM risk

## Proof of Concept

Let's say, Bob has following sequence of events

- MINT at T0: Bob is a LP who mints N pool tokens at T0

- MINT at T1: Bob mints another M pool tokens at T1. At this point, had the protocol correctly updated fees before minting new pool tokens, Bob's fees & growth rate would be a function of current liquidity (N), global updated short fee growth rate at t1 (s_t1) and Bob's previous growth rate at t_0 (b_t0)

- BURN at T2: Bob burns N + M tokens at T2. At this point, Bob's fees should be a function of previous liquidity (N+M), global short fee growth rate (s_t2) and Bob's previous growth rate at t_1(b_t1) -> since this update never happened, Bob's previous growth rate is wrongly referenced b_t0 instead of b_t1. 

Bob could collect a lower fees because of this state inconsistency

## Tools Used

## Recommended Mitigation Steps
Update the liquidity position state right before minting.

After [line 302 of Pool.sol](https://github.com/code-423n4/2023-01-timeswap/blob/ef4c84fb8535aad8abd6b67cc45d994337ec4514/packages/v2-pool/src/structs/Pool.sol#L302), update the LiquidityPosition by adding

```
  liquidityPosition.update(pool.long0FeeGrowth, pool.long1FeeGrowth, pool.shortFeeGrowth);
```