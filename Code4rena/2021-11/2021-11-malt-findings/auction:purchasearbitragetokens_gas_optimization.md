## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Auction:purchaseArbitrageTokens gas optimization](https://github.com/code-423n4/2021-11-malt-findings/issues/147) 

# Handle

GiveMeTestEther


# Vulnerability details

## Impact
L209 nextCommitmentId = nextCommitmentId + 1; can be removed and
L202 can be changed to nextCommitmentId++; to save a SLOAD

## Proof of Concept
https://github.com/code-423n4/2021-11-malt/blob/c3a204a2c0f7c653c6c2dda9f4563fd1dc1cecf3/src/contracts/Auction.sol#L177

## Tools Used
Manual Analysis

## Recommended Mitigation Steps

