## Tags

- bug
- G (Gas Optimization)
- sponsor confirmed

# [More gas efficient calculation of weights](https://github.com/code-423n4/2021-11-nested-findings/issues/28) 

# Handle

GiveMeTestEther


# Vulnerability details

## Impact
Calculation of the weights in the function "setRoyaltiesWeight" of FeeSplitter.sol (row 95) can be done more gas efficient.

    function setRoyaltiesWeight(uint256 _weight) public onlyOwner {
        totalWeights -= royaltiesWeight;
        royaltiesWeight = _weight;
        totalWeights += _weight;
    }

can be rewritten as

    function setRoyaltiesWeight(uint256 _weight) public onlyOwner {
        totalWeights = totalWeights - royaltiesWeight + _weight;
        royaltiesWeight = _weight;
    }

=> write only once to the storage of totalWeights.

## Proof of Concept

## Tools Used
Visual Studio Code

## Recommended Mitigation Steps
see above

