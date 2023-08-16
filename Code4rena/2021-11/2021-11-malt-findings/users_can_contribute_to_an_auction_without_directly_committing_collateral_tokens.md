## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [Users Can Contribute To An Auction Without Directly Committing Collateral Tokens](https://github.com/code-423n4/2021-11-malt-findings/issues/188) 

# Handle

leastwood


# Vulnerability details

## Impact

`purchaseArbitrageTokens` enables users to commit collateral tokens and in return receive arbitrage tokens which are redeemable in the future for Malt tokens. Each auction specifies a commitment cap which when reached, prevents users from participating in the auction. However, `realCommitment` can be ignored by directly sending the `LiquidityExtension` contract collateral tokens and subsequently calling `purchaseArbitrageTokens`.

## Proof of Concept

Consider the following scenario:
- An auction is currently active.
- A user sends collateral tokens to the `LiquidityExtension` contract.
- The same user calls `purchaseArbitrageTokens` with amount `0`.
- The `purchaseAndBurn` call returns a positive `purchased` amount which is subsequently used in auction calculations.

As a result, a user could effectively influence the average malt price used throughout the `Auction` contract.

https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/Auction.sol#L177-L214
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/LiquidityExtension.sol#L117-L128

## Tools Used

Manual code review.

## Recommended Mitigation Steps

Consider adding a check to ensure that `realCommitment != 0` in `purchaseArbitrageTokens`.

