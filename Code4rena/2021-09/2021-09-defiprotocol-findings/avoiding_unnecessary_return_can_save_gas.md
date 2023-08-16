## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Avoiding unnecessary return can save gas](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/153) 

# Handle

0xRajeev


# Vulnerability details

## Impact
Unnecessary return of argument value via state variable which costs a SLOAD, returns the same value as argument back to caller where the return value is ignored.

## Proof of Concept

https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Basket.sol#L221

https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Basket.sol#L216-L222

https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Auction.sol#L104

## Tools Used
Manual Analysis

## Recommended Mitigation Steps

Remove return value for this function.

