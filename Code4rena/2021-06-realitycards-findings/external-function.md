## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed
- disagree with severity
- resolved

# [external-function](https://github.com/code-423n4/2021-06-realitycards-findings/issues/19) 

# Handle

heiho1


# Vulnerability details

## Impact

RCMarket#tokenURI(uint256) is declared external in the IRCMarket interface but is declared public in the RCMarket implementation.   This is inconsistent and affect the gas behavior of the function: https://gus-tavo-guim.medium.com/public-vs-external-functions-in-solidity-b46bcf0ba3ac

## Proof of Concept

https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/interfaces/IRCMarket.sol#L27

https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCMarket.sol#L344

https://github.com/code-423n4/2021-06-realitycards/blob/86a816abb058cc0ed9b6f5c4a8ad146f22b8034c/contracts/RCMarket.sol#L344

## Tools Used

Slither

## Recommended Mitigation Steps

Mark the implementation method as external.

