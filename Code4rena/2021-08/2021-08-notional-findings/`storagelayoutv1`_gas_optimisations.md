## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [`StorageLayoutV1` Gas Optimisations](https://github.com/code-423n4/2021-08-notional-findings/issues/96) 

# Handle

leastwood


# Vulnerability details

## Impact

The `StorageLayoutV1.sol` contract is inherited by several contracts and represents a shared common state that ensures the slot layout of certain contracts are the same. It is possible to minimise the number of storage slots used by rearranging the state variables in the most efficient way.

## Proof of Concept

https://github.com/code-423n4/2021-08-notional/blob/main/contracts/global/StorageLayoutV1.sol

## Tools Used

Manual code review

## Recommended Mitigation Steps

Arrange the `uint16`  and `bytes1` variables such that they fit into the same slot.

