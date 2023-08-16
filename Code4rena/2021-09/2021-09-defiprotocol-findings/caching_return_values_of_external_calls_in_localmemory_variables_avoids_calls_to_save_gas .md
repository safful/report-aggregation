## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Caching return values of external calls in local/memory variables avoids CALLs to save gas ](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/147) 

# Handle

0xRajeev


# Vulnerability details

## Impact

There are places across contracts where the same external calls are made multiple times within a function. Caching return values of such calls in local/memory variables avoids CALLs to save gas. CALLs cost 2600 gas after Berlin upgrade. MLOADs cost only 3 gas units.

## Proof of Concept

Cache factory.ownerSplit() return value to save 2600 gas in this function which gets called at every mint/burn.: https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Basket.sol#L120-L121

Hoist basketAsERC20.totalSupply() external call out of the loop because it remains the same and each call costs 2600 gas: https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Auction.sol#L96-L99

## Tools Used
Manual Analysis

## Recommended Mitigation Steps
Cache return values of external calls in local/memory variables

