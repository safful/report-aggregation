## Tags

- bug
- G (Gas Optimization)
- resolved
- sponsor confirmed

# [claimFee loop does not check for zero transfer amount (Incentivizer.sol) ](https://github.com/code-423n4/2021-12-perennial-findings/issues/43) 

# Handle

ye0lde


# Vulnerability details

## Impact
Transfer amount can be checked for > 0 before calling `push' which makes a call to `safeTransfer`.
This can save gas by avoiding the external call.

## Proof of Concept

The transfer is here:
https://github.com/code-423n4/2021-12-perennial/blob/fd7c38823833a51ae0c6ae3856a3d93a7309c0e4/protocol/contracts/incentivizer/Incentivizer.sol#L237-L238


## Tools Used
Visual Studio Code, Remix

## Recommended Mitigation Steps
Check that transfer amount > 0 before L#237-238 are executed.

Consider checking for `amount` > 0 in these functions:
https://github.com/code-423n4/2021-12-perennial/blob/fd7c38823833a51ae0c6ae3856a3d93a7309c0e4/protocol/contracts/utils/types/Token18.sol#L51
https://github.com/code-423n4/2021-12-perennial/blob/fd7c38823833a51ae0c6ae3856a3d93a7309c0e4/protocol/contracts/utils/types/Token18.sol#L68
https://github.com/code-423n4/2021-12-perennial/blob/fd7c38823833a51ae0c6ae3856a3d93a7309c0e4/protocol/contracts/utils/types/Token18.sol#L85

