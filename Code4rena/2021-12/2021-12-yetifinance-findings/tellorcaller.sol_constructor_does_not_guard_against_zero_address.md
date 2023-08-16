## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [TellorCaller.sol constructor does not guard against zero address](https://github.com/code-423n4/2021-12-yetifinance-findings/issues/78) 

# Handle

jayjonah8


# Vulnerability details

## Impact
the constructor in TellorCaller.sol should ensure that the _tellorMasterAddress arg passed in is not a zero address as a safegaurd.

## Proof of Concept
https://github.com/code-423n4/2021-12-yetifinance/blob/main/packages/contracts/contracts/Dependencies/TellorCaller.sol#L24

## Tools Used
Manual code review 

## Recommended Mitigation Steps
require(address(_tellorMasterAddress) != address(0), "Zero address")

