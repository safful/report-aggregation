## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [A more efficient for loop index proceeding](https://github.com/code-423n4/2022-01-timeswap-findings/issues/49) 

# Handle

Jujic


# Vulnerability details

## Impact
Here you could use unchecked{++i} to save gas since it is more efficient then i++.

```
for (uint256 i; i < ids.length; i++) {

```

## Proof of Concept
https://github.com/code-423n4/2022-01-timeswap/blob/bf50d2a8bb93a5571f35f96bd74af54d9c92a210/Timeswap/Timeswap-V1-Core/contracts/TimeswapPair.sol#L359

## Tools Used
Remix

## Recommended Mitigation Steps

