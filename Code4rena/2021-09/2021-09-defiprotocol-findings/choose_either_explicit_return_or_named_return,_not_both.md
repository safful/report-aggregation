## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Choose either explicit return or named return, not both](https://github.com/code-423n4/2021-09-defiprotocol-findings/issues/154) 

# Handle

0xRajeev


# Vulnerability details

## Impact

Choosing either named return or explicit instead of specifying both may reduce gas due to unnecessary bytecode introduced. proposeBasketLicense() uses a named return variable which is never assigned and instead uses an explicit return statement.

## Proof of Concept

https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Factory.sol#L71

https://github.com/code-423n4/2021-09-defiProtocol/blob/52b74824c42acbcd64248f68c40128fe3a82caf6/contracts/contracts/Factory.sol#L90

## Tools Used
Manual Analysis

## Recommended Mitigation Steps
Choose either explicit return or named return, not both

