## Tags

- bug
- 0 (Non-critical)
- resolved
- sponsor confirmed

# [Implicit casts should be explicit as per the global code style](https://github.com/code-423n4/2022-01-xdefi-findings/issues/129) 

# Handle

Dravee


# Vulnerability details

## Impact
Code clarity / code style

## Proof of Concept
At the following places, the casts are implicit, whereas the project's style hints at explicit casts everywhere :
https://github.com/XDeFi-tech/xdefi-distribution/blob/v1.0.0-beta.0/contracts/XDEFIDistribution.sol#L255
https://github.com/XDeFi-tech/xdefi-distribution/blob/v1.0.0-beta.0/contracts/XDEFIDistribution.sol#L269
https://github.com/XDeFi-tech/xdefi-distribution/blob/v1.0.0-beta.0/contracts/XDEFIDistribution.sol#L314

## Tools Used
VS Code

## Recommended Mitigation Steps
Use explicit casts everywhere for unsigned integers, as it's the practice everywhere else

