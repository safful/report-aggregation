## Tags

- bug
- 1 (Low Risk)
- sponsor confirmed

# [reassignGlobalAdmin() Lack of Zero Address Check ](https://github.com/code-423n4/2021-11-malt-findings/issues/113) 

# Handle

Meta0xNull


# Vulnerability details

## Impact
A wrong user input or wallets defaulting to the zero addresses for a missing input can lead to the contract needing to redeploy or Users'FUND Locked inside the Contract.

## Proof of Concept
https://github.com/code-423n4/2021-11-malt/blob/main/src/contracts/Permissions.sol#L63-L77

## Tools Used
Manual Review

## Recommended Mitigation Steps
requires Addresses is not zero.

require(_admin != address(0), "Address Can't Be Zero")

