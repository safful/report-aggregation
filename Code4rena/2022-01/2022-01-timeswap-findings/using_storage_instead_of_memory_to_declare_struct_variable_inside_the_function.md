## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [using storage instead of memory to declare struct variable inside the function](https://github.com/code-423n4/2022-01-timeswap-findings/issues/141) 

# Handle

rfa


# Vulnerability details

## Impact
more expensive gas

## Proof of Concept
https://github.com/code-423n4/2022-01-timeswap/blob/main/Timeswap/Timeswap-V1-Core/contracts/TimeswapPair.sol#L58

instead of caching state on memory. just read it directly from the storage. 
State memory state = pools[maturity].state;

## Tools Used
self research on: 
https://remix.ethereum.org/

## Recommended Mitigation Steps
State storage state = pools[maturity].state;

