## Tags

- bug
- 2 (Med Risk)
- disagree with severity
- sponsor confirmed

# [Verification should be leafed based and not address based](https://github.com/code-423n4/2022-05-factorydao-findings/issues/148) 

# Lines of code

https://github.com/code-423n4/2022-05-factorydao/blob/db415804c06143d8af6880bc4cda7222e5463c0e/contracts/MerkleVesting.sol#L115
https://github.com/code-423n4/2022-05-factorydao/blob/db415804c06143d8af6880bc4cda7222e5463c0e/contracts/MerkleDropFactory.sol#L92


# Vulnerability details

## Impact
Contracts should clarify what is the intended behavior for Merkle trees with multiple leafs with the same address. 

## Recommended Mitigation Steps

There is 2 possible behaviors: 
 - either - what is currently done - you only authorize one claim per address, in which case the multiple leaf are here to give users a choice - for example you could use `MerkleVesting` to give users the choice between 2 sets of vesting parameters and have something close to `MerkleResistor`.
 - either you use a mapping based on the leaf to store if a leaf has been claimed or not. 


This behavior should be clarified in the comments at least, and made clear to merkle tree builders.

