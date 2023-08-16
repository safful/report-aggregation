## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`addLiquidity` Does Not Reset Approval If Not All Tokens Were Added To Liquidity Pool](https://github.com/code-423n4/2021-11-malt-findings/issues/228) 

# Handle

leastwood


# Vulnerability details

## Impact

`addLiquidity` is called when users reinvest their tokens through bonding events. The `RewardReinvestor` first transfers Malt and rewards tokens before adding liquidity to the token pool. `addLiquidity` provides protections against slippage by a margin of 5%, and any dust token amounts are transferred back to the caller. In this instance, the caller is the `RewardReinvestor` contract which further distributes the dust token amounts to the protocol's treasury. However, the token approval for this outcome is not handled properly. Dust approval amounts can accrue over time, leading to large Uniswap approval amounts by the `UniswapHandler` contract.

## Proof of Concept

https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/DexHandlers/UniswapHandler.sol#L212-L214
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/DexHandlers/UniswapHandler.sol#L216-L218

## Tools Used

Manual code review.

## Recommended Mitigation Steps

Consider resetting the approval amount if either `maltUsed < maltBalance` or `rewardUsed < rewardBalance` in `addLiquidity`.

