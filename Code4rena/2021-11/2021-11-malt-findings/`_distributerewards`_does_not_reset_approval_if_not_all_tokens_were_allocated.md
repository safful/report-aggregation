## Tags

- bug
- 2 (Med Risk)
- sponsor confirmed

# [`_distributeRewards` Does Not Reset Approval If Not All Tokens Were Allocated](https://github.com/code-423n4/2021-11-malt-findings/issues/229) 

# Handle

leastwood


# Vulnerability details

## Impact

`_distributeRewards` attempts to reward LP token holders when the price of Malt exceeds its price target. Malt Finance is able to being Malt back to its peg by selling Malt and distributing rewards tokens to LP token holders. An external call to `Auction` is made via the `allocateArbRewards` function. Prior to this call, the `StabilizerNode` approves the contract for a fixed amount of tokens, however, the `allocateArbRewards` function does not necessarily utilise this entire amount. Hence, dust token approval amounts may accrue from within the `StabilizerNode` contract.

## Proof of Concept

https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/StabilizerNode.sol#L252-L253
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/Auction.sol#L809-L871

## Tools Used

Manual code review

## Recommended Mitigation Steps

Consider resetting the approval amount if the input `rewarded` amount to `allocateArbRewards` is less than the output amount.

