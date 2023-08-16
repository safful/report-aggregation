## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Caching state variables in local/memory variables avoids SLOADs to save gas](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/145) 

# Handle

0xRajeev


# Vulnerability details

## Impact

There are numerous places across contracts where the same state variables are read multiple times within a function. Caching state variables in local/memory variables avoids SLOADs to save gas. Warm SLOADs cost 100 gas after Berlin upgrade. MLOADs cost only 3 gas units. 
## Proof of Concept

bondAmount: https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Auction.sol#L62-L66

Cache basket, bondTimestamp, factory: https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Auction.sol#L78-L99

Cache basket and bondAmount: https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Auction.sol#L116-L121

Cache pendingWeights: https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Basket.sol#L50

Cache lastFee: https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Basket.sol#L111-L116

Cache factory: https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Basket.sol#L120-L121

Cache pendingPublisher: https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Basket.sol#L136-L139

Cache pendingLicenseFee: https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Basket.sol#L154-L157

Cache auction & pendingWeights: https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Basket.sol#L173-L182

Cache publisher and auction: https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Basket.sol#L208-L213

Cache proposals: https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Factory.sol#L62


## Tools Used
Manual Analysis

## Recommended Mitigation Steps
Cache state variables in local/memory variables to save gas.

