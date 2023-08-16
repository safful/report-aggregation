## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [Struct TokenStream remove unused variable merkleAccess](https://github.com/code-423n4/2021-11-streaming-findings/issues/42) 

# Handle

GiveMeTestEther


# Vulnerability details

## Impact
Remove the unused merkleAccess variable in the TokenStream struct. According to the struct packing it uses a single storage slot.

## Proof of Concept
https://github.com/code-423n4/2021-11-streaming/blob/56d81204a00fc949d29ddd277169690318b36821/Streaming/src/Locke.sol#L106

## Tools Used

## Recommended Mitigation Steps
- remove the unused merkleAccess variable in the TokenStream struct

