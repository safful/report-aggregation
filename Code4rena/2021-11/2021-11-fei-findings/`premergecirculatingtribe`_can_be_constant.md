## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [`preMergeCirculatingTribe` can be constant](https://github.com/code-423n4/2021-11-fei-findings/issues/147) 

# Handle

loop


# Vulnerability details

`preMergeCirculatingTribe` is a `uint256` set to a constant value which isn't changed by any function in the contract and can thus be declared as a constant state variable to save some gas during deployment and when using `preMergeCirculatingTribe`.

## Proof of Concept
https://github.com/code-423n4/2021-11-fei/blob/main/contracts/TRIBERagequit.sol#L28

