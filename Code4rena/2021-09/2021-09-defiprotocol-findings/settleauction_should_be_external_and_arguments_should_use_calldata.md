## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [settleAuction should be external and arguments should use calldata](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/108) 

# Handle

t11s


# Vulnerability details

## Impact
Gas is wasted making `settleAuction` public, and using `memory` Instead of calldata for its arguments.

## Proof of Concept

https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Auction.sol#L69

