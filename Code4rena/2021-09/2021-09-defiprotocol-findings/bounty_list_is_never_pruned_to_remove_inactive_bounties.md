## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Bounty list is never pruned to remove inactive bounties](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/165) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Given that there is no removal of claimed/inactive bounties, the bounty list could grow very long over time requiring a lot of gas for traversal.

## Proof of Concept

https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Auction.sol#L126-L151

## Tools Used
Manual Analysis

## Recommended Mitigation Steps
Recommend pruning the claimed bounties by deleting them from the list.

