## Tags

- bug
- G (Gas Optimization)
- disagree with severity
- sponsor confirmed

# [Unused constant](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/156) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Unused constant BLOCK_DECREMENT may be an indication of missing logic or redundant code. In this case, this appears to be a redundant constant same as Factory.auctionDecrement.

## Proof of Concept

https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Auction.sol#L14

## Tools Used
Manual Analysis

## Recommended Mitigation Steps
Use the constant or remove it.

