## Tags

- bug
- 0 (Non-critical)
- sponsor confirmed

# [alternative solidity coding](https://github.com/code-423n4/2021-06-tracer-findings/issues/15) 

# Handle

gpersoon


# Vulnerability details

## Impact
Solidity allows some tricks to make the code easier to read:    
LibMath.sol:
    uint256 public constant POSITIVE_INT256_MAX = 2**255 - 1;
    uint256 public constant POSITIVE_INT256_MAX = uint(type(int256).max);   // alternative coding

Insurance.sol:
    uint256 public multiplyFactor = 36523 * (10**11);
    uint256 public multiplyFactor = 0.0036523e18;   // alternative coding

 ## Proof of Concept
Provide direct links to all referenced code in GitHub. Add screenshots, logs, or any other relevant proof that illustrates the concept.

## Tools Used

## Recommended Mitigation Steps
Use the most readable coding


