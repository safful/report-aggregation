## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Minimize Storage Slots (LeveragedPool.sol)](https://github.com/code-423n4/2021-10-tracer-findings/issues/3) 

# Handle

ye0lde


# Vulnerability details

## Impact
 It is possible to minimize the number of storage slots used by rearranging the state variables in a more efficient way.

## Proof of Concept
In  LeveragedPool.sol:
https://github.com/tracer-protocol/perpetual-pools-contracts/blob/646360b0549962352fe0c3f5b214ff8b5f73ba51/contracts/implementation/LeveragedPool.sol#L22-L44

## Tools Used
Visual Studio Code

## Recommended Mitigation Steps
Arrange the uint32, bytes32, and bool variables such that they fit into the same slot.


