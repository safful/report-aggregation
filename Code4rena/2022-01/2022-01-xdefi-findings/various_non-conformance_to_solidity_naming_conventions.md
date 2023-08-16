## Tags

- bug
- 0 (Non-critical)
- resolved
- sponsor confirmed

# [Various Non-Conformance to Solidity naming conventions](https://github.com/code-423n4/2022-01-xdefi-findings/issues/60) 

# Handle

Dravee


# Vulnerability details

## Impact
Solidity defines a naming convention that should be followed.

## Proof of Concept
```
Variable XDEFIDistribution.MAX_TOTAL_XDEFI_SUPPLY (contracts/XDEFIDistribution.sol#14) is not in mixedCase
Constant XDEFIDistribution._pointsMultiplier (contracts/XDEFIDistribution.sol#17) is not in UPPER_CASE_WITH_UNDERSCORES
Variable XDEFIDistribution._pointsPerUnit (contracts/XDEFIDistribution.sol#18) is not in mixedCase
Variable XDEFIDistribution.XDEFI (contracts/XDEFIDistribution.sol#20) is not in mixedCase
Variable XDEFIDistribution._zeroDurationPointBase (contracts/XDEFIDistribution.sol#30) is not in mixedCase
Variable XDEFIDistribution._locked (contracts/XDEFIDistribution.sol#37) is not in mixedCase
Function IXDEFIDistribution.XDEFI() (contracts/interfaces/IXDEFIDistribution.sol#37) is not in mixedCase
```

## Tools Used
Slither

## Recommended Mitigation Steps
Follow the Solidity naming convention: https://docs.soliditylang.org/en/v0.4.25/style-guide.html#naming-conventions

