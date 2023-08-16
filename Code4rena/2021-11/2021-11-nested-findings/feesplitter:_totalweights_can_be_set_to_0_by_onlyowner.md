## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [FeeSplitter: totalWeights can be set to 0 by onlyOwner](https://github.com/code-423n4/2021-11-nested-findings/issues/43) 

# Handle

GiveMeTestEther


# Vulnerability details

## Impact
The storage variable totalWeights can be set 0 by onlyOwner and therefore we would have a division by zero in the function "_computeShareCount"
https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/FeeSplitter.sol#L268

## Proof of Concept
-Assumption we have one shareholder with weight S1 > 0 and royaltiesWeight > 0.
-With the function updateShareholder the onlyOwner sets the S1 of our shareholder to 0.
	- updateShareholder: https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/FeeSplitter.sol#L166
	- require(_totalWeights > 0, "FeeSplitter: TOTAL_WEIGHTS_ZERO") condition is met because _totalWeights is the sum of all shareholder weights + royaltiesWeight, and royaltiesWeight is > 0
- With the function setRoyaltiesWeight the onlyOwner sets the royaltiesWeight to 0.
	- setRoyaltiesWeight: https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/FeeSplitter.sol#L94
- => setRoyaltiesWeight is 0 and totalWeights is 0


## Tools Used
Manual Analysis

## Recommended Mitigation Steps
At the end of the function setRoyaltiesWeight check for 0 weight with a require: require(totalWeights > 0, "FeeSplitter: TOTAL_WEIGHTS_ZERO");
https://github.com/code-423n4/2021-11-nested/blob/5d113967cdf7c9ee29802e1ecb176c656386fe9b/contracts/FeeSplitter.sol#L94

