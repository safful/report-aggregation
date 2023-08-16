## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [The contracts use unlocked pragma](https://github.com/code-423n4/2022-01-trader-joe-findings/issues/181) 

# Handle

hyh


# Vulnerability details

## Impact

As different compiler versions have critical behavior specifics if the contract gets accidentally deployed using another compiler version compared to one they tested with, various types of undesired behavior can be introduced.

## Proof of Concept

All the contracts in scope use unlocked pragma: ```pragma solidity ^0.8.0```, allowing wide enough range of versions.

Examples:

https://github.com/code-423n4/2022-01-trader-joe/blob/main/contracts/LaunchEvent.sol#L4

https://github.com/code-423n4/2022-01-trader-joe/blob/main/contracts/RocketJoeToken.sol#L3


## Recommended Mitigation Steps

Consider locking compiler version, for example `pragma solidity 0.8.6`.

This can have additional benefits, for example using custom errors to save gas and so forth.

