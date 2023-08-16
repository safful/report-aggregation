## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed
- resolved

# [Unused return value from orderbook.findNewOwner() and treasury.payRent()](https://github.com/code-423n4/2021-06-realitycards-findings/issues/53) 

# Handle

JMukesh


# Vulnerability details

## Impact
checking the return value from function indicates ether function call was success or failure because of that, we should utilise the return value 

## Proof of Concept

In RCmarket.sol

https://github.com/code-423n4/2021-06-realitycards/blob/main/contracts/RCMarket.sol#L1025

https://github.com/code-423n4/2021-06-realitycards/blob/main/contracts/RCMarket.sol#L1060

## Tools Used
slither

## Recommended Mitigation Steps

Utilize return value

