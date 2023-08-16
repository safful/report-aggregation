## Tags

- bug
- duplicate
- 3 (High Risk)
- sponsor confirmed

# [`ConcentratedLiquidityPoolManager`'s incentives can be stolen](https://github.com/code-423n4/2021-09-sushitrident-2-findings/issues/37) 

# Handle

cmichel


# Vulnerability details

The `ConcentratedLiquidityPoolManager` keeps all tokens for all incentives in the same contract. The `reclaimIncentive` function does not reduce the `incentive.rewardsUnclaimed` field and thus one can reclaim tokens several times.
This allows anyone to steal all tokens from all incentives by creating an incentive themself, and once it's expired, repeatedly claim the unclaimed rewards until the token balance is empty.

## POC
- Attacker creates an incentive for a non-existent pool using a random address for `pool` (This is done such that no other user can claim rewards as we need a non-zero `rewardsUnclaimed` balance for expiry). They choose the `incentive.token` to be the token they want to steal from other incentives. (for example, `WETH`, `USDC`, or `SUSHI`) They choose the `startTime, endTime, expiry` such that the checks pass, i.e., starting and ending in a few seconds from now, expiring in 5 weeks. Then they choose a non-zero `rewardsUnclaimed` and transfer the `incentive.token` to the PoolManager.
- Attacker waits for 5 weeks until the incentive is expired
- Attacker can now call `reclaimIncentive(pool, incentiveId, amount=incentive.rewardsUnclaimed, attacker, false)` to withdraw `incentive.rewardsUnclaimed` of `incentive.token` from the pool manager.
- As the `incentive.rewardsUnclaimed` variable has not been decreased, they can keep calling `reclaimIncentive` until the pool is drained.

## Impact
An attacker can steal all tokens in the PoolManager.

## Recommended Mitigation Steps
In `reclaimIncentive`, reduce `incentive.rewardsUnclaimed` by the withdrawn `amount`.


