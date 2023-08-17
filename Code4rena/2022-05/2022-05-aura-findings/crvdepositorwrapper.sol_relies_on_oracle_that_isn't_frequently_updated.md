## Tags

- bug
- 2 (Med Risk)
- resolved
- sponsor confirmed

# [CrvDepositorWrapper.sol relies on oracle that isn't frequently updated](https://github.com/code-423n4/2022-05-aura-findings/issues/115) 

# Lines of code

https://github.com/code-423n4/2022-05-aura/blob/4989a2077546a5394e3650bf3c224669a0f7e690/contracts/CrvDepositorWrapper.sol#L56-L65


# Vulnerability details

## Impact
Unpredictable slippage, sandwich vulnerability or frequent failed transactions

## Proof of Concept
CrvDepostiorWrapper uses the TWAP provided by the 20/80 WETH/BAL. The issue is that this pool has only handled ~15 transactions per day in the last 30 days, which means that the oracle frequently goes more than an hour without updating. Each time a state changing operation is called, the following code in the balancer pool takes a snapshot of the pool state BEFORE any operation changes it:

https://github.com/balancer-labs/balancer-v2-monorepo/blob/80e1a5db7439069e2cb53e228bce0a8a51f5b23e/pkg/pool-weighted/contracts/oracle/OracleWeightedPool.sol#L156-L161

This could result in the price of the oracle frequently not reflecting the true value of the assets due to infrequency of update. Now also consider that the pool has a trading fee of 2%. Combine an inaccurate oracle with a high fee pool and trades can exhibit high levels of "slippage". To account for this outputBps in AuraStakingProxy needs to be set relatively low or risks frequent failed transactions when calling distribute due to slippage conditions not being met. The lower outputBps is set the more vulnerable distribute becomes to sandwich attacks. 

## Tools Used

## Recommended Mitigation Steps
Consider using chainlink oracles for both BAL and ETH to a realtime estimate of the LP value. A chainlink LP oracle implementation can be found in the link below:
https://blog.alphaventuredao.io/fair-lp-token-pricing/

