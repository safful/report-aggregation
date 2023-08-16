## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [AMM pool can be drained using a flashloan and calling `stabilize`](https://github.com/code-423n4/2021-11-malt-findings/issues/372) 

# Handle

stonesandtrees


# Vulnerability details

## Impact
All of the `rewardToken` in a given AMM pool can be removed from the AMM pool and distributed as LP rewards.

## Proof of Concept
In the `stabilize` method in the `StabilizerNode` the initial check to see if the Malt price needs to be stabilized it uses a short period TWAP:
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/StabilizerNode.sol#L156

However, if the price is above the threshold for stabilization then the trade size required to stabilize looks at the AMM pool directly which is vulnerable to flashloan manipulation.

https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/DexHandlers/UniswapHandler.sol#L250-L275

Attack:
1. Wait for TWAP to rise above the stabilization threshold
2. Flashloan remove all but a tiny amount of Malt from the pool.
3. Call `stabilize`. This will pass the TWAP check and execute `_distributeSupply` which in turn ultimately calls `_calculateTradeSize` in the `UniswapHandler`. This calculation will determine that almost all of the `rewardToken` needs to be removed from the pool to return the price to peg. 
4. Malt will mint enough Malt to remove a lot of the `rewardToken` from the pool.
5. The protocol will now distribute that received `rewardToken` as rewards. 0.3% of which goes directly to the attacker and the rest goes to LP rewards, swing trader and the treasury.

The amount of money that can be directly stolen by a malicious actor is small but it can cause a lot of pain for the protocol as the pool will be destroyed and confusion around rewards will be created.

## Tools Used
Manual review

## Recommended Mitigation Steps
Use a short TWAP to calculate the trade size instead of reading directly from the pool.

