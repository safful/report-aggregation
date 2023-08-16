## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [Remove FixedPointMath ](https://github.com/code-423n4/2021-11-yaxis-findings/issues/18) 

# Handle

TimmyToes


# Vulnerability details

## Impact
Including unused libraries could potentially use up gas and certainly makes the code more difficult to understand, hindering developer integrations/poor confused security auditors.

## Proof of Concept
https://github.com/code-423n4/2021-11-yaxis/blob/0311dd421fb78f4f174aca034e8239d1e80075fe/contracts/v3/alchemix/adapters/YaxisVaultAdapter.sol#L19
https://github.com/code-423n4/2021-11-yaxis/blob/0311dd421fb78f4f174aca034e8239d1e80075fe/contracts/v3/alchemix/adapters/YearnVaultAdapter.sol#L19
The contract does not use FixedPointMath and compiles with these lines removed.

## Recommended Mitigation Steps
Remove line 10,19 from each contract (FixedPointMath lines).
I'd also prefer the removal of 
import "hardhat/console.sol";
but this is not having any impact and is just to tidy and shorten the files.

