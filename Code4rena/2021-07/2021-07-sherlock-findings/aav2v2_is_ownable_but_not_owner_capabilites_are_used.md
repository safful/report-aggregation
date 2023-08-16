## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Aav2V2 is Ownable but not owner capabilites are used](https://github.com/code-423n4/2021-07-sherlock-findings/issues/8) 

# Handle

patitonar


# Vulnerability details

## Impact
Reduce bytecode size of AaveV2 by removing Ownable given that there is no functionality for owners

## Proof of Concept
https://github.com/code-423n4/2021-07-sherlock/blob/d9c610d2c3e98a412164160a787566818debeae4/contracts/strategies/AaveV2.sol#L21

## Tools Used
Manual Review

## Recommended Mitigation Steps
Update AaveV2 to only extend from IStrategy and remove Ownable import

