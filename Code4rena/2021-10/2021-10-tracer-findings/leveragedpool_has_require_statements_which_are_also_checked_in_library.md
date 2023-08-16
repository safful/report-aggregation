## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [LeveragedPool has require statements which are also checked in library](https://github.com/code-423n4/2021-10-tracer-findings/issues/10) 

# Handle

loop


# Vulnerability details

When making external calls to ERC20 functions LeveragedPool checks for zero addresses. These checks are already available in the OpenZeppelin ERC20 implementation which is used. This results in redundant checks which increase gas costs when calling these functions. 

## Proof of Concept
Require statements used in LeveragedPool:
- https://github.com/tracer-protocol/perpetual-pools-contracts/blob/646360b0549962352fe0c3f5b214ff8b5f73ba51/contracts/implementation/LeveragedPool.sol#L148
- https://github.com/tracer-protocol/perpetual-pools-contracts/blob/646360b0549962352fe0c3f5b214ff8b5f73ba51/contracts/implementation/LeveragedPool.sol#L163-L164
- https://github.com/tracer-protocol/perpetual-pools-contracts/blob/646360b0549962352fe0c3f5b214ff8b5f73ba51/contracts/implementation/LeveragedPool.sol#L234
- https://github.com/tracer-protocol/perpetual-pools-contracts/blob/646360b0549962352fe0c3f5b214ff8b5f73ba51/contracts/implementation/LeveragedPool.sol#L251

Checks in OpenZeppelin implementation:
- https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol#L225-L226
- https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol#L252
- https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/token/ERC20/ERC20.sol#L275

## Tools Used
Remix

